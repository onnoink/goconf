# goconf
Configuration for Go applications

这是一个从`kratos`框架中迁移出来的Go应用配置模块，推荐使用`protobuf`进行应用配置的预设在，这是`krtaos`中的最佳实践,
可以更好的约束配置文件的内容。

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

使用方法，目前支持文件来源，可以通过实现`Source`接口的方式定义自己的配置来源

```go
    c := config.New(
        config.WithSource(
            env.NewSource(""),
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



