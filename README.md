# goconf


goconf 是针对Go应用程序的配置解决方案。它旨在在应用程序中工作，并且可以扩展不同类型的配置需求和格式。

## Install

```shell
go get github.com/onnoink/goconf
```


## 使用方式


> tips Go应用程序开发人员应对程序内配置的结构、类型有全面的了解，所以推荐在Go语言中通过 Unmarshal 配置内容到 `struct` 结构体的方式使用和管理配置信息。


### 定义配置结构

使用结构体进行配置结构的定义，如下面的关于数据库的定义，配置结构体需要使用一个外部结构体包裹，

```go

type MyGoConf struct {
	Database Database `json:"database,omitempty"`
}


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

### 加载配置信息 
goconf通过实现`Source`来支持不同的配置来源，可以支持
* `nacos`
* `file`
* `appllo`等
库中包含`file` 来源的实现，可以通过实现`Source`接口的方式实现自己的配置来源

### Example

```go
    // 声明一个新的配置实例，并使用Option方法加载配置文件
    c := config.New(
        config.WithSource(
            file.NewSource("your config file path : example ./configs.yaml"),
        ),
    )
	
	defer c.Close()
	
	// 声明配置信息
    var bc conf.Bootstrap
	
	//  Unmarshal配置信息到结构体
    if err := c.Scan(&bc); err != nil {
        panic(err)
    }
```



