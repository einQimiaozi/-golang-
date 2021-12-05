![grpcimg1](https://github.com/einQimiaozi/-golang-/blob/main/img/截屏2021-12-05%20下午8.46.38.png)

gprc配置问题：

grpc配置命令：

```
protoc --go_out=pb --go_opt=paths=source_relative \
      --go-grpc_out=pb --go-grpc_opt=paths=source_relative \
      proto/*.proto
```

可以将配置命令写入makefile中（因为太长了记不住），另外该命令是最新版protobuf文件生成命令，网上查到的命令都是老版了，已经失效了

## protobuf语法细节

protobuf支持各种基本数据类型（自己查），如果希望定义枚举或者自定义结构，需要将结构写在另一个proto文件中，将两个文件的package设置为同一个package，然后就可以import了

通过import其他自定义结构，可以在当前proto的message使用这些结构或枚举类型
