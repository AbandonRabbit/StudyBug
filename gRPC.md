# 简介

RPC的全称是Remote Procedure Call，远程过程调用。这是一种协议，是用来屏蔽分布式计算中的各种调用细节，使得你可以像是本地调用一样直接调用一个远程的函数。

**客户端与服务端沟通的过程**

1. 客户端发送数据(以字节流的方式)编码
2. 服务端接受并解析。根据约定知道要执行什么。然后把结果返回给客户解码

**RPC**

1. RPC就是将上述过程封装下，使其操作更加优化
2. 使用一些大家都认可的协议使其规范化
3. 做成一些框架。直接或间接产生利益

**gRPC**

gRPC是一个高性能的、开源的通用的RPC框架。

在gRPC中，我们称调用方为client，被调用方为server。

gRPC会屏蔽底层的细节，client只需要直接调用定义好的方法，就能拿到预期的返回结果。对于server端来说，还需要实现我们定义的方法。同样的，gRPC也会帮我们屏蔽底层的细节，我们只需要实现所定义的方法的具体逻辑即可。

此外，gRPC还是语言无关的。可以用C++作为服务端，使用Golang、 Java等作为客户端。为了实现这一点，我们在"定义服务“和在编码和解码的过程中，应该是做到语言无关的。



gRPC使用了**Protocol Buffss**。这是谷歌开源的一套成熟的数据结构序列化机制。



序列化:将数据结构或对象转换成二进制串的过程

反序列化:将在序列化过程中所产生的二进制串转换成数据结构或者对象的过程



protobuf是谷歌开源的一种数据格式，适合高性能，对响应速度有要求的数据传输场景。因为profobuf是二进制数据格式，需要编码和解码，数据本身不具有可读性。因此只能反序列化之后得到真正可读的数据。

优势

1. 序列化后体积相比Json和XML很小，适舍网络传输
2. 支持跨平台多语言
3. 消息格式升级和兼容性还不错
4. 序列化反序列化速度很快

# 安装

1. 安装Protocol Buffers
   下载地址：https://github.com/protocolbuffers/protobuf/releases   *下载之后记得配置环境变量，再命令行中输入protoc*

2. 安装gRPc的核心库
   `go get google.go1ang.org/grpc`

3. 安装代码生成工具
   ~~~go
   go install google.golang.org/protobuf/cmd/protoc-gen-go@latest
   
   go install google.golang.org/grpc/cmd/protoc-gen-go-grpc@latest
   ~~~

   







生成RPC文件

~~~go
//生成go语言文件
protoc --go_out=. hello.proto

//生成rpc相关文件
protoc --go-grpc_out=. hello.proto
~~~



## proto文件

~~~protobuf
//指定语法版本
syntax="proto3";

//指定生成目录  . 表示当前目录   service表示生成的go文件的包名为service
option go_package =".;service";

// 结构体 定义参数
message HelloRequest{
  //message 消息格式，需要传输的数据格式的定义

  //后面的数字为消息号  指定生成位置
  string requestName = 1;
  int64 agr = 2;
  /*
  字段规则：
    required：必填字段，不设置会导致编码异常，在 protobuf2 中使用，在 protobuf3 中删除
    optional：可选字段，protobuf3中没有 required 和 optional 等说明关键字，都默认为 optional
    repeated：可重复字段，在go语言中会定义成切片

    每个字段都必须有自己唯一的标识号

    消息结构体可以嵌套，如果调用内部结构体直接调用就好
  */
}

// 结构体 定义参数
message HelloResponse{
  string responseMsg=1;
}

//定义一个服务 包含一些方法
service SayHello{
  //定义一个SayHello方法，需要传入HelloRequest参数，返回一个HelloResponse参数
  rpc SayHello(HelloRequest) returns (HelloResponse){}

}

~~~

**protobuf文件只是服务端和客户端调用的约束，只是一个接口文档，server端只需要把方法实现，客户端调用这个方法**



## 服务端编写

1. 创建gRPC Server对象，你可以理解为它是Server端的抽象对象
2. 将server(其包含需要被调用的服务端接口)注册到gRPC Server的内部注册中心。这样可以在接受到请求时，通过内部的服务发现，发现该服务端接口并转接进行逻辑处理
3. 创建Listen，监听TCP端口
4. gRPC Server开始lis.Accept，直到Stop

## 客户端编写

1. 创建与给定目标(服务端)的连接交互
2. 创建server的客户端对象
3. 发送RPC请求，等待同步响应，得到回调后返回响应结果
4. 输出响应结果

# 加密

*此处说到的认证，不是用户的身份认证，而是指多个server和多个client之间，如何识别对方是谁，并且可以安全的进行数据传输.* 

**加密方式：**

1. SSL/TLS认证方式(采用http2协议)
2. 基于Token的认证方式(基于安全连接)
3. 不采用任何措施的连接，这是不安全的连接(默认采用http1)
4. 自定义的身份认证

客户端和服务端之间调用，我们可以通过加入证书的方式，实现调用的安全性

> TLS (Transport Layer Security，安全传输层)，TLS是建立在传输层TCP协议之上的协议，服务于应用层，它的前身是SSL (Secure Socket Layer,安全套接字层)，它实现了将应用层的报文进行加密后再交由TCP进行传输的功能。

TLS协议主要解决如下三个网络安全问题。

1. 保密(message privacy)，保密通过加密encryption实现，所有信息都加密传输，第三方无法嗅探
2. 完整性(message integrity)，通过MAC校验机制，一旦被篡改，通信双方会立刻发现;
3. 认证(mutual authentication)，双方认证,双方都可以配备证书，防止身份被冒充;

### TSL认证方式实现

便捷包下载地址：http://slproweb.com/products/Win320penSSL.html  *安装完之后配置环境变量*

使用命令行输入 openssl 测试安装是否成功

**生成证书**

~~~cmd
#1、生成私钥文件 key  输出为 server.key 文件
openssl genrsa -out server.key 1024

#2、生成证书文件 crt 全部回车即可，接下来的提示输入信息可以不填  
openssl req -new -x509 -key server.key -out server.crt -days 36500

#3、生成csr
openssl req -new -key server.key -out server.csr
~~~

~~~cmd
# 更改openss7.cnf (Linux是openssl.cfg)
# 1) 复制一份你安装的openssl的bin目录里面的openssl.cnf 文件到你项目所在的目录
# 2）找到[ CA_default ]，打开 copy_extensions = copy (就是把前面的#去掉)
# 3）找到[ req ]，打开req_extensions = v3_req # The extensions to add to a certificate request
# 4) 找到[ v3_req ]，添加subjectAltName = @alt_names
# 5) 添加新的标签[ alt_names ]，和标签字段
DNS.1 = * .kuangstudy . com
~~~

~~~cmd
#生成证书私钥test.key
openssl genpkey -algorithm RSA -out test.key

#通过私钥test.key生成证书请求文件test.csr（注意cfg和cnf)
openssl req -new -nodes -key test.key -out test.csr -days 3650 -subj "/C=cn/OU=myorg/O=mycomp/CN=myname " -config ./openssl.cnf -extensions v3_req
#test.csr是上面生成的证书请求文件。ca.crt/server.key是cA证书文件和key，用来对test.csr进行签名认证。这两个文件在第一部分生成。

#生成SAN证书pem
openssl x509 -req -days 365 -in test.csr -out test.pem -CA server.crt -CAkey server.key -CAcreateserial -extfile ./openssl.cnf -extensions v3_req
~~~

在代码中使用

~~~go
//服务端使用步骤
//创建认证
creds,err := credentials.NewServerTLSFromFile("SAN证书绝对路径","私钥绝对路径")
//开启认证  在创建gRPC服务时将creds带入进去
grpcServer := grpc.NewServer(grpc.Creds(creds))

//客户端使用步骤
creds,err := credentialsNewClientTLSFromFile("SAN证书绝对路径","请求域")
conn, err := grpc.NewClient("127.0.0.1:9090", grpc.WithTransportCredentials(creds))
~~~

### token认证实现

