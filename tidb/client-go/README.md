#  overview

client-go 是 TiKV 的 go 语言版的客户端。提供了 raw kv 接口以及 txn ( 事务 ) kv 的接口。

这篇文章会简单介绍 Raw  KV  以及  Txn  KV  的接口使用以及相关源码。下面是  client-go  项目下各个 package 的简单介绍 。

| Package                | Introduction                               |
| ---------------------- | ------------------------------------------ |
| codec                  | 各种数据类型的编码                         |
| config                 | 配置参数等逻辑                             |
| **locate**             | 定位 key  属于哪个 region，region  cache等 |
| mockstore/**mocktikv** | 在单机存储引擎上模拟 TiKV 的一些行为       |
| **rawkv**              | raw kv 接口                                |
| **txnkv**              | 事务相关的操作接口和实现                   |
| rpc                    | rpc 请求（ PD & TiKV ），连接池等逻辑      |
| retry                  | 重试  backoff 等逻辑                       |

#  Raw  KV 

## Put

Put 接口使用示例如下（忽略了 一些 error check 的逻辑 ）

```go
	// NewClient creates a client with PD cluster addrs.
	cli, err := rawkv.NewClient(context.TODO(), []string{"127.0.0.1:2379"}, config.Default())
	defer cli.Close()

	key := []byte("Company")
	val := []byte("PingCAP")

	// put key into tikv
	err = cli.Put(context.TODO(), key, val)
```

`rawkv.NewClient`  的入参 `127.0.0.1:2379`  是 PD 的地址 ，通过访问 PD 可以获取访问 key 所在的 region，然后根据  region 获取其 owner region 所在 node ( TiKV )  的地址，然后发送 rpc 请求到对应 node (  TiKV ) 完成相应操作。

```go
type Client struct {
	clusterID   uint64
	conf        *config.Config
	regionCache *locate.RegionCache // 缓存 region 的信息 
	pdClient    pd.Client					  
	rpcClient   rpc.Client          // rpc client, 管理 tikv 节点的 rpc 连接以及发送 rpc 请求  
}
```

`cli.Put` 是 raw  put 接口，其源码逻辑大致如下：

```go
// Put stores a key-value pair to TiKV.
func (c *Client) Put(ctx context.Context, key, value []byte) error {
	req := &rpc.Request{
		Type: rpc.CmdRawPut,
		RawPut: &kvrpcpb.RawPutRequest{
			Key:   key,
			Value: value,
		},
	}
	resp, _, err := c.sendReq(ctx, key, req)
	... // error check logic.
}
```

`Put` 函数首先会构造一个 `rpc.Request`，其类型为 `CmdRawPut` ，并将要 put 的 key, value 存在 `kvrpcpb.RawPutRequest` 中。然后调用 `c.sendReq` 函数发送 req 请求，这个函数非常重要，后面几乎所有的 rpc 请求都是调用这个函数来发送的，其逻辑大致如下：

* locate ( 定位 ) rpc 请求中 key 所在的 region
* 发送 rpc 请求到相应的 owner region
* 处理 region error 并决定是否需要重试。

具体代码如下： 

```go
func (c *Client) sendReq(ctx context.Context, key []byte, req *rpc.Request) (*rpc.Response, *locate.KeyLocation, error) {
  // backoff 用于发送请求出错时的等待重试，错误可能是 region miss 或者 tikv server too busy 等等
	bo := retry.NewBackoffer(ctx, retry.RawkvMaxBackoff)
  // sender 的结构体是 RegionRequestSender，用于将 rpc 请求发送给相应
	sender := rpc.NewRegionRequestSender(c.regionCache, c.rpcClient)
	for {
    // 定位 key 所在的 region。会先通过 regionCache 缓存的 region 信息来 locate, 如果没找到或者缓存的 region 已经过期，就重新访问 pd 来 locate key 所在region。
		loc, err := c.regionCache.LocateKey(bo, key)
		if err != nil {
			return nil, nil, err
		}
    // 发送 rpc 请求到相应的 region 
		resp, err := sender.SendReq(bo, req, loc.Region, c.conf.RPC.ReadTimeoutShort)
		if err != nil {
			return nil, nil, err
		}
		regionErr, err := resp.GetRegionError()
		if err != nil {
			return nil, nil, err
		}
		if regionErr != nil {
      // region error 可能是 region miss，tikv server too busy 等等。
      // backoff 用于等待一段时间后重试。
			err := bo.Backoff(retry.BoRegionMiss, errors.New(regionErr.String()))
			if err != nil {
				return nil, nil, err
			}
			continue
		}
		return resp, loc, nil
	}
}
```

`RegionRequestSender.SendReq` 先获取 region 的 meta 信息以及 owner region 所在的 node ( TiKV ) 地址，然后设置相关的 region 元信息到 rpc 请求的 context 后，然后通过 `rpc.Client`的 `SendRequest` 方法发送 rpc 请求，`SendRequest` 首先尝试从 rpc 连接池里面获取一个相应 node 地址的 rpc connection，如果没有就新建一个，然后发送 rpc 请求。

`RegionRequestSender.SendReq` 里面有调用一个 `onRegionError`  函数，用于处理一些 region 错误, 比如：

* region  not  leader. 可能是 pd 调度了热点  region，然后 region cache 缓存没有更新导致。
* store not  match. 每个 rpc request 都会 带上一个  store  ID， 如果 TiKV node  收到的请求 里面发现 Store  ID  和自己的 ID  不匹配，就会 返回这个错误。  
* Region epoch does not match。每次 region 元信息 发生 变更 时，  region 的  epoch  就会变，说明缓存的 region   久了。
* TiKV  server is  busy. 
* stale commannd.  

###  Get/Delete

`Get/Delete` 情况基本类似，只是把  `rpc.Request` 的类型换成了 `CmdRawGet/CmdRawDelete` 



## Txn KV

[TiKV 事务模型概览](https://pingcap.com/blog-cn/tidb-transaction-model/)

