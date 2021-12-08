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


