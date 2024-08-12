# 安装

1. 安装goctl

   `go install github.com/zeromicro/go-zero/tools/goctl@latest` 使用 `goctl`测试安装是否成功

2. 安装组件

   `goctl env check --install --verbose --force`

   > go-zero微服务框架下的代码生成工具。使用goctl可显著提升开发效率，让开发人员将时间重点放在业务开发上，其功能有
   >
   > api服务生成
   >
   > rpc服务生成
   >
   > model代码生成
   >
   > 模板管理

3. 安装protoc & protoc-gen-go

   `goctl env check -i -f --verbose`

快速创建 api 服务

`goctl api new api`

# ETCD

Etcd是一个高可用的分布式键值存储系统，主要用于共享配置信息和服务发现。它采用Raft一致性算法来保证数据的强一致性，并且支持对数据进行监视和更新。 *可以理解为加强版的 redis* ，比 redis 的数据可靠性更强

主要是用于微服务的配置中心，服务发现

## 基本命令

~~~etcd
// 设置或更新值
etcdctl put name 张三
// 获取值
etcdctl get name
// 只要value
etcdctl get name --print-value-only
// 获取name前缀的键值对
etcdctl get --prefix name
// 删除键值对
etcdctl del name
// 监听键的变化
etcdctl watch name
~~~

