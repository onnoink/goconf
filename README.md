# goconf
Configuration for Go applications

## 简介

这是一个从`kratos`框架中迁移出来的Go应用配置模块，关于应用内配置，程序编写者应对程序内配置的结构、类型有全面的了解，所以推荐在Go语言中通过`struct`结构体的方式对配置内容进行规范化管理。


## 关于配置文件结构体的定义

可以直接使用结构体进行配置结构的定义，例如下面的关于数据库的定义，配置结构体需要使用一个统一的外部结构体包裹，并且加上json tag

```go

type Database struct {
	Driver string `json:"driver,omitempty"`
	Source string `json:"source,omitempty"`
}

```

也可以通过protobuf的形式定义配置文件结构，通过protoc输出go的相关结构体

```protobuf

syntax = "proto3";
package your_name.api;

option go_package = "your_name/conf;conf";

import "google/protobuf/duration.proto";

message Bootstrap {
  Server server = 1;
  Data data = 2;
}

message Server {
  message HTTP {
    string network = 1;
    string addr = 2;
    google.protobuf.Duration timeout = 3;
  }
  
  message GRPC {
    string network = 1;
    string addr = 2;
    google.protobuf.Duration timeout = 3;
  }
  
  HTTP http = 1;
  GRPC grpc = 2;
}

message Data {
  message Database {
    string driver = 1;
    string source = 2;
  }
  
  message Redis {
    string network = 1;
    string addr = 2;
    string password = 5;
    google.protobuf.Duration read_timeout = 3;
    google.protobuf.Duration write_timeout = 4;
  }
  
  Database database = 1;
  Redis redis = 2;
}

```

使用`protc`命令进行生成对应go语言文件

```go 
// 省略部分代码

type Bootstrap struct {
    state         protoimpl.MessageState
    sizeCache     protoimpl.SizeCache
    unknownFields protoimpl.UnknownFields
    
    Server *Server `protobuf:"bytes,1,opt,name=server,proto3" json:"server,omitempty"`
    Data   *Data   `protobuf:"bytes,2,opt,name=data,proto3" json:"data,omitempty"`
}

type Server struct {
    state         protoimpl.MessageState
    sizeCache     protoimpl.SizeCache
    unknownFields protoimpl.UnknownFields
    
    Http *Server_HTTP `protobuf:"bytes,1,opt,name=http,proto3" json:"http,omitempty"`
    Grpc *Server_GRPC `protobuf:"bytes,2,opt,name=grpc,proto3" json:"grpc,omitempty"`
}

```

## 使用方法目

### 实现Source支持不能来源 
goconf通过实现`Source`来支持不同的配置来源，可以支持`nacos`,`file`,`appllo`等，当前支持`file`来源，可以通过实现`Source`接口的方式定义自己的配置来源


### Example

```go
    c := config.New(
        config.WithSource(
            file.NewSource("your config file path : examole ./configs.yaml"),
        ),
    )
	defer c.Close()
    var bc conf.Bootstrap
    if err := c.Scan(&bc); err != nil {
        panic(err)
    }
	
	// 下面可以使用bc.Server bc.Data 获取Bootstrap下面的配置
```



