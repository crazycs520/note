# learn resource

*  [illustrated guide](http://thesecretlivesofdata.com/raft/) to Raft
*  https://raft.github.io/ 
*  [raft paper](https://pdos.csail.mit.edu/6.824/papers/raft-extended.pdf)   [中文](https://github.com/maemual/raft-zh_cn/blob/master/raft-zh_cn.md)
*  [advice](https://thesquareplanet.com/blog/students-guide-to-raft/) written for 6.824 students in 2016





#paper read note

**Raft is a consensus algorithmfor managing a replicated log.**

##Figure 2: important

* State

  ```go
  type Raft struct{
    
  	//persistent state on all server
  	CurrentTerm int        //latest term server has seen ,initialized to 0
  	VoteFor     int        //candidateID that receive vote in currnt term
  	Logs        []LogEntry //log entries , first index is 1

  	// volatile state on all server
  	CommitIndex int //index of highest log entry known to be committed , initialized ti 0
  	LastApplied int //index of highest log entry applied to state machine,initialized to 0

  	// volatile state on leaders
  	NextIndex  []int //each server's index of next log entry to send to that server,initalized to 							leader last log +1
  	MatchIndex []int //each server's  index of highest log entry known wo be replicated on 									server,initialized to 0
    
    	// 已获得的投票数
  	grantedVotesCount int

  	//state：follower , candidate, leader
  	State   string

  	//election timeout
  	ElectionTimer *time.Timer
  }
  ```

* AppendEntries RPC

  Invoked by leader to replicate log entries;

  Also used as heartbeat

  * Arguments:

    ```Go
    type AppendEntriesArgs struct {
    	Term         int        //leader’s term
    	LeaderID     int        //so follower can redirect clients
    	PreLogIndex  int        //index of log entry immediately preceding new ones
    	PreLogTerm   int        //term of prevLogIndex entry
    	Entries      []LogEntry //log entries to store (empty for heartbeat;may send more than one for efficiency)
    	LeaderCommit int        //leader’s commitIndex
    }
    ```

  * Results

    ```Go
    type AppendEntryReply struct {
    	Term        int  //currentTerm, for leader to update itself
    	Success     bool //true if follower contained entry matching prevLogIndex and prevLogTerm
    	CommitIndex int
    }
    ```

  * Receiver implementation

    1. Reply false if term < currentTerm (§5.1)
    2. Reply false if log doesn’t contain an entry at prevLogIndex whose term matches prevLogTerm (§5.3)
    3. If an existing entry conflicts with a new one (same index but different terms), delete the existing entry and all that follow it (§5.3)
    4. Append any new entries not already in the log
    5. If leaderCommit > commitIndex, set commitIndex = min(leaderCommit, index of last new entry)

* RequestVote RPC

  Invoked by candidates to gather votes (§5.2).

  * Arguments

    ```Go
    type RequestVoteArgs struct {
    	Term         int //candidate’s term
    	CandidateID  int //candidate requesting vote
    	LastLogIndex int //index of candidate’s last log entry (§5.4)
    	LastLogTerm  int //term of candidate’s last log entry (§5.4)
    }
    ```

  * Results

    ```Go
    type RequestVoteReply struct {
    	Term        int  // currentTerm, for candidate to update itself
    	VoteGranted bool // true means candidate received vote
    }
    ```

  * Receiver implementation

    1. Reply false if term < currentTerm (§5.1)
    2. If votedFor is null or candidateId, and candidate’s log is at least as up-to-date as receiver’s log, grant vote (§5.2, §5.4)
    3. 

* Rules for Servers

  * All Servers
    * If commitIndex > lastApplied: increment lastApplied, apply log[lastApplied] to state machine (§5.3)
    * If RPC request or response contains term T > currentTerm:  set currentTerm = T, convert to follower (§5.1)
  * Followers
    * Respond to RPCs from candidates and leaders
    * If election timeout elapses without receiving AppendEntries RPC from current leader or granting vote to candidate: convert to candidate
  * Candidates
    * On conversion to candidate, start election:
      1. Increment currentTerm
      2. Vote for self
      3. Reset election timer
      4. Send RequestVote RPCs to all other servers
    * If votes received from majority of servers: become leader
    * If AppendEntries RPC received from new leader: convert to follower
    * If election timeout elapses: start new election
  * Leaders
    * Upon election: send initial empty AppendEntries RPCs (heartbeat) to each server; repeat during idle periods to prevent election timeouts (§5.2)
    * If command received from client: append entry to local log, respond after entry applied to state machine (§5.3)
    * If last log index ≥ nextIndex for a follower: send AppendEntries RPC with log entries starting at nextIndex
      * If successful: update nextIndex and matchIndex for follower (§5.3)
      * If AppendEntries fails because of log inconsistency: decrement nextIndex and retry (§5.3)
    * If there exists an N such that N > commitIndex, a majority of matchIndex[i] ≥ N, and log[N].term == currentTerm: set commitIndex = N (§5.3, §5.4).

### Raft Basic

* Three states of Server

  ​	<img src="https://raw.githubusercontent.com/crazycs520/images/master/raft2.png" style="zoom:40%" />

*  Raft divides time into **terms**  of arbitrary length。

  ​	<img src="https://raw.githubusercontent.com/crazycs520/images/master/raft1.png" style="zoom:50%" />

*  Terms are numbered with consecutive integers. 

* Each term begins with an election , in which one or more candidates attempt to become leader

*  If a candidate wins the election, then it serves as leader for the rest of the term

* Some elections fail, in which case the term ends without choosing a leader.

* The transitions between terms may be observed at different times on different servers.

*  Terms act as a logical clock in Raft

*  Raft servers communicate using RPCs( remote procedure calls)

  *  RequestVote RPCs are initiated by candidates during elections
  *  AppendEntries RPCs are initiated by leaders to replicate log entries and to provide a form of **heartbeat**( (AppendEntriesRPCs that carry no log entries)
  *  a third RPC for transferring snapshots between servers.

### Leader election

*  Raft uses a heartbeat mechanism to trigger leader election.  If a follower receives no communication over a period of time called the election timeout , then it assumes there is no viable leader and begins an election to choose a new leader.
*  To begin an election, a follower increments its current term and transitions to candidate state. It then votes for itself and issues RequestVote RPCs in parallel to each of the other servers in the cluster.  A candidate continues in this state until one of three things happens:
  *  it wins theelection
  *  another server establishes itself as leader . a candidate may receive an AppendEntries RPC from another server claiming to be leader.
    *  If the leader’s term (included in its RPC) is at least as large as the candidate’s current term, then the candidate recognizes the leader as legitimate and returns to follower state.
    *  If the leader’s term (included in its RPC) is at least as large as the candidate’s current term, then the candidate recognizes the leader as legitimate and returns to follower state.
  *  a period of time goes by with no winner.
    *  Raft uses randomized election timeouts to ensure that split votes

### Log replication



​	<img src="https://raw.githubusercontent.com/crazycs520/images/master/raft3.png" style="zoom:30%" />

> Figure 6: Logs are composed of entries, which are numbered sequentially. Each entry contains the term in which it was created (the number in each box) and a command for the state machine. An entry is considered committed if it is safe for that entry to be applied to state machines.

* each log entry has

  *  the term number
  *  Command for the state machine
  *  log index

*  apply a log entry to the state machines; such an entry is called committed. 

*  A log entry is **committed** once the leader that created the entry has replicated it on a majority of the servers

*  Log Matching Property

  *  If two entries in different logs have the same index and term, then they store the same command.
  *  If two entries in different logs have the same index and term, then the logs are identical in all preceding entries.

*  leader crashes can leave the logs inconsistent

  ​	<img src="https://raw.githubusercontent.com/crazycs520/images/master/raft4.png" style="zoom:30%" />

### safety

####Election restriction

* the voter denies its vote if its own log is more up-to-date than that of the candidate.
* which of two logs is **more up-to-date** by comparing the index and term of the last entries in the logs.

#### Committing entries from previous terms

<img src="https://raw.githubusercontent.com/crazycs520/images/master/raft5.png" style="zoom:40%" />

* (a)  S1 is leader and partially replicates the log entry at index 2.
* (b)  S1 crashes; S5 is elected leader for term 3 with votes from S3, S4, and itself, and accepts a different entry at log index 2.
* (c)  S5 crashes; S1 restarts, is elected leader, and continues replication. At this point, the log entry from term 2 has been replicated on a majority of the servers, but it is not committed.
* (d)   If S1 crashes as in, S5 could be elected leader (with votes from S2, S3, and S4) and overwrite the entry with its own entry from term 3. 
* **(e)  if S1 replicates an entry from its current term on a majority of the servers before crashing**, then this entry is committed (S5 cannot win an election). At this point all preceding entries in the log are committed as well.

1. QA: how to figure out this situation ?

   reference : https://groups.google.com/forum/#!topic/raft-dev/d-3XQbyAg2Y

   1. after winning the election, the leader MUST commit a no-op entry to the log. Only after that it can be considered the active leader.

 



# lab2 Raft

## code note

### config.go

```Go
type config struct {
	mu        sync.Mutex
	t         *testing.T
	net       *labrpc.Network
	n         int
	done      int32 // tell internal threads to die
	rafts     []*Raft
	applyErr  []string // from apply channel readers
	connected []bool   // whether each server is on the net
	saved     []*Persister
	endnames  [][]string    // the port file names each sends to
	logs      []map[int]int // copy of each server's committed entries
}
```







![image-20200516173318235](/Users/cs/Library/Application Support/typora-user-images/image-20200516173318235.png)

![image-20200516173520911](/Users/cs/Library/Application Support/typora-user-images/image-20200516173520911.png)