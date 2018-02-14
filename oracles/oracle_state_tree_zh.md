[back](./oracles.md)
## 预言机状态树

预言机状态树包含了预言机对象。每一个预言同样有一个分离的树，包括了对这个特定树的所有查询。
一个预言的存在和查询/响应的存在，在双方指定合约时，必须被证明。从效率的角度来看，所有的
查询不会被直接存储在预言对象中，而是预言对象包含一个分离的默克尔(Merkle)树的根 hash 值，
这个默克尔树包含了这个预言的所有查询。


### 预言状态树对象

- 预言状态树包含了预言对象
- 查询状态树(每个预言对象对应一个)包含查询对象

#### 预言对象

- 被预言注册事务创建
- 当TTL 超时被删除

#### 预言查询对象

- 被预言查询事务创建
- 被预言响应事务关闭
- 一旦关闭将不可改变
- 超时被删数(对于一个存活的查询是查询超时，对于一个关闭的查询是响应超时)

超时是由创建时查询 TTL，和在预言查询事务响应的TTL 决定的。如果/当 一个预言的响应被链接受时，
超时会根据响应的TTL 被更新。

```
{ query_id       :: id()
, oracle_address :: pubkey()
, query          :: query()
, response       :: oracle_response()
, expires        :: block_height()
, response_ttl   :: relative_ttl()
, fee            :: integer()
}
```

### Oracle state tree update
### 预言状态树的更新

在链的顶端，当一个对象的TTL 超时时，预言状态树就会被修剪，我们定义这个操作的顺序：

1. 删除超时的对象，对象应该按照他们的ID 生序排序被删除
2. 按照交易的顺序，在块中插入一个新的对象

ID 的排列顺序是按照树的中序遍历。

### 处理对象的TTL

我们将会按照TTL 和ID 的排序来保留对象的cache，cache 会有以下优点：

- 下一个将要删除的对象是cache 中的第一个对象
  - TTL 低于块的高度 -> 我们完了
  - TTL 等于块的高度 -> 删除这个对象
- 缓存将会按照预言状态树的中序遍历被重建

### 预言查询对象的修剪

If the oracle query has not been given a response, the poster of
the query should be refunded the oracle query fee. If the oracle has
responded, the oracle was already given the funds at response time.

**NOTE:** 退款还未实现
