grpc中的远程调用并不是通过反射+动态代理实现的，而是通过使用protobuf对请求响应进行编码解码，直接和语言绑定实现的

protobuf针对不同的语言会将其编译为该语言的代码，方便不同语言｜cs之间远程调用，效率高，不需要cs之间通过反射+动态代理解决

![grpcimg1](https://github.com/einQimiaozi/-golang-/blob/main/img/截屏2021-12-08%20下午6.50.20.png)

![grpcimg1](https://github.com/einQimiaozi/-golang-/blob/main/img/截屏2021-12-08%20下午6.51.27.png)

![grpcimg1](https://github.com/einQimiaozi/-golang-/blob/main/img/截屏2021-12-08%20下午6.54.59.png)

![grpcimg1](https://github.com/einQimiaozi/-golang-/blob/main/img/截屏2021-12-08%20下午6.58.24.png)

## gprc配置问题：

grpc配置命令：

```
protoc --proto_path proto/ --go_out=pb/ proto/*.proto
```

可以将配置命令写入makefile中（因为太长了记不住），另外该命令是最新版protobuf文件生成命令，网上查到的命令都是老版了，已经失效了

## protobuf应用

创建一个message存储个人信息，使用枚举类型自定义性别，在另外一个proto中定义地址类型

注意，两个proto文件都放在用级目录下

info.proto

```
syntax = "proto3";

package my.info;

option go_package = "/proto";

import "address.proto";

message Persona {
  string name = 1;
  Gender gender  = 2;
  uint32 age = 3;
  uint32 id = 4;
  Address address = 5;
}

enum Gender {
  NULL = 0;
  MAN = 1;
  FALEMAM = 2;
}
```

address.proto

```
syntax = "proto3";

package my.info;

option go_package = "/proto";

message Address {
  string country = 1;
  string city = 2;
  string detail = 3;
}
```

编译两个proto后生成go文件，在main中使用

```
package main

import (
	"fmt"
	"helloGrpc/pb/proto"
)

func main() {
	info := NewInfo()
	fmt.Println(info)
}

func NewInfo() *proto.Persona {
	info := &proto.Persona{
		Name: "Nick",
		Gender: proto.Gender_FALEMAM,
		Age: 18,
		Id: 1234,
		Address: &proto.Address{
			Country: "China",
			City: "Peking",
			Detail: "1st Street",
		},
	}
	return info
}
```

## 消息变更

grpc中如果变更message，尽量使用reversed关键字保留消息编号，不要直接删除消息，否则会导致使用grpc通信的两端对消息数据结构同步不及时带来的数据传错问题，这样做还可以很好的对消息版本进行向下兼容

如果message1添加了一个新的字段变为message2，那么节点1使用message1的结构通信是，新添加的字段会直接被赋一个默认值（根据数据类型不同默认值也不同），或者你可以根据业务场景设置一个绝对不会被用到的值做默认值

## grpc生命周期

![shengmingzhouqi](https://github.com/einQimiaozi/-golang-/blob/main/img/shengmingzhouqi.png)

其中注册传输通道和建立连接只需要一次，客户端和服务器可复用，元数据是否返回你可以自己定，不是必须的

## grpc身份认证（识别节点是客户端还是服务器，并安全传输数据）

1.无身份认证，不安全，适合快速建立连接

2.tls/ssl

3.使用google token认证

4.自定义协议进行认证，这部分一般不支持多语言，只能和业务开发语言紧耦合

## grpc四种消息传输方式

![1](https://github.com/einQimiaozi/-golang-/blob/main/img/1.png)

![2](https://github.com/einQimiaozi/-golang-/blob/main/img/2.png)

![3](https://github.com/einQimiaozi/-golang-/blob/main/img/3.png)

![4](https://github.com/einQimiaozi/-golang-/blob/main/img/4.png)

在grpc中不管是request还是reponse都不能为空，如果不需要传输数据也需要带一个默认值或者空指针

## c/s交互

cs在使用grpc交互的时候有6个行为

  - Magic： 定义cs交互使用的协议，一般默认为http2
  - SETTINGS：定义链接参数和cs交互前的确认ack，参数并不是单次rpc请求的参数，而是整个链接过程中的参数
  - HEADERS：grpc的请求头信息
  - DATA：cs交互具体要传输的数据帧
  - WINDOW_UPDATE：控制流窗口大小
  - PING：每次交互检测当前链接是否还可用








