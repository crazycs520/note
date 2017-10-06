# learn resource

*  [illustrated guide](http://thesecretlivesofdata.com/raft/) to Raft
*  https://raft.github.io/ 
*  [raft paper](https://pdos.csail.mit.edu/6.824/papers/raft-extended.pdf)   [中文](https://github.com/maemual/raft-zh_cn/blob/master/raft-zh_cn.md)
*  [advice](https://thesquareplanet.com/blog/students-guide-to-raft/) written for 6.824 students in 2016



#paper read note

**Raft is a consensus algorithmfor managing a replicated log.**

## Raft consensus algorithm

Figure 2: important

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

