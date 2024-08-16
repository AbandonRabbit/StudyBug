分布式key-value存储系统，用于配置共享和服务的注册和发现。

## 安装

`go get go.etcd.io/etcd/client/v3 `

## 使用

### 创建链接

~~~go
cli, err := clientv3.New(clientv3.Config{
        Endpoints:   []string{"127.0.0.1:2379"},
        DialTimeout: 5 * time.Second,
})
~~~

### put和get操作

~~~go
//put
ctx, cancel := context.WithTimeout(context.Background(), time.Second)
_, err = cli.Put(ctx, "lmh", "lmh")
cancel()

//get
ctx, cancel = context.WithTimeout(context.Background(), time.Second)
resp, err := cli.Get(ctx, "lmh")
cancel()
~~~

### watch操作

~~~go
//获取未来更改通知
// watch key:lmh change
rch := cli.Watch(context.Background(), "lmh") // <-chan WatchResponse
for wresp := range rch {
	for _, ev := range wresp.Events {
		fmt.Printf("Type: %s Key:%s Value:%s\n", ev.Type, ev.Kv.Key, ev.Kv.Value)
	}
}
~~~

### lease租约

~~~go
// 创建一个5秒的租约
resp, err := cli.Grant(context.TODO(), 5)

// 5秒钟之后, /lmh/ 这个key就会被移除
 _, err = cli.Put(context.TODO(), "/lmh/", "lmh", clientv3.WithLease(resp.ID))
~~~

### keepAlive

```go
_, err = cli.Put(context.TODO(), "/lmh/", "lmh", clientv3.WithLease(resp.ID))

// the key 'foo' will be kept forever
ch, kaerr := cli.KeepAlive(context.TODO(), resp.ID)
```