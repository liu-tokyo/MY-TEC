# GRPC简介

## **gRPC是什么**

更多人可能接触的更多的是基于**REST**的通信。我们已经看到，REST 是一种灵活的体系结构样式，它定义了对实体资源的基于CRUD的操作。 客户端使用请求/响应通信模型跨 HTTP 与资源进行交互。尽管 REST 是广泛实现的，但一种较新的通信技术**gRPC**已在各个生态中获得巨大的动力。

gRPC，其实就是RPC框架的一种，前面带了一个g，代表是RPC中的大哥，龙头老大的意思，另外g也有global的意思，意思是全球化比较fashion，是一个**高性能、开源和通用**的 RPC 框架，基于**ProtoBuf(Protocol Buffers)** 序列化协议开发，且支持众多开发语言。面向服务端和移动端，基于 HTTP/2 设计，带来诸如双向流、流控、头部压缩、单 TCP 连接上的多复用请求等特。这些特性使得其在移动设备上表现更好，更省电和节省空间占用。

在 gRPC 里**客户端**应用可以像调用本地对象一样直接调用另一台不同的机器上**服务端**应用的方法，使得您能够更容易地创建分布式应用和服务。与许多 RPC 系统类似，gRPC 也是基于以下理念：定义一个**服务**，指定其能够被远程调用的方法（包含参数和返回类型）。在服务端实现这个接口，并运行一个 gRPC 服务器来处理客户端调用。在客户端拥有一个**存根**能够像服务端一样的方法。



![img](https://pic3.zhimg.com/80/v2-8f7c89dbc3881888d4257490ac2666be_720w.jpg)

大致请求流程:

1. 客户端（gRPC Stub）调用 A 方法，发起 RPC 调用。
2. 对请求信息使用 Protobuf 进行对象序列化压缩（IDL）。
3. 服务端（gRPC Server）接收到请求后，解码请求体，进行业务逻辑处理并返回。
4. 对响应结果使用 Protobuf 进行对象序列化压缩（IDL）。
5. 客户端接受到服务端响应，解码请求体。回调被调用的 A 方法，唤醒正在等待响应（阻塞）的客户端调用并返回响应结果。

> ❝ 一个RPC框架大致需要动态代理、序列化、网络请求、网络请求接受(netty实现)、动态加载、反射这些知识点。现在开源及各公司自己造的RPC框架层出不穷，唯有掌握原理是一劳永逸的。

## **RPC框架是什么**

RPC 框架说白了就是让你可以像调用本地方法一样调用远程服务提供的方法，而不需要关心底层的通信细节。简单地说就让远程服务调用更加简单、透明。 RPC包含了客户端（Client）和服务端（Server）

业界主流的 RPC 框架整体上分为三类：

- 支持多语言的 RPC 框架，比较成熟的有 Google 的 gRPC、Apache（Facebook）的 Thrift；
- 只支持特定语言的 RPC 框架，例如新浪微博的 Motan；
- 支持服务治理等服务化特性的分布式服务框架，其底层内核仍然是 RPC 框架, 例如阿里的 Dubbo。

## **gRPC的特性**

看官方文档的介绍，有以下几点特性：

- grpc可以跨语言使用。支持多种语言 支持C++、Java、Go、Python、Ruby、C#、Node.js、Android Java、Objective-C、PHP等编程语言
- 基于 IDL ( 接口定义语言（Interface Define Language）)文件定义服务，通过 proto3 工具生成指定语言的数据结构、服务端接口以及客户端 Stub；
- 通信协议基于标准的 HTTP/2 设计，支持·双向流、消息头压缩、单 TCP 的多路复用、服务端推送等特性，这些特性使得 gRPC 在移动端设备上更加省电和节省网络流量；
- 序列化支持 PB（Protocol Buffer）和 JSON，PB 是一种语言无关的高性能序列化框架，基于 HTTP/2 + PB, 保障了 RPC 调用的高性能。
- 安装简单，扩展方便（用该框架每秒可达到百万个RPC）

## **gRPC使用流程**

gprc的使用流程一般是这样的：

1. 定义标准的proto文件(**后面部分会详细讲解protobuf的使用**)
2. 生成标准代码
3. 服务端使用生成的代码提供服务(**参考各个语言的使用**)
4. 客户端使用生成的代码调用服务(**参考各个语言的使用**)

## **gRPC优势和劣势**

先看一下gRPC与带有json的HTTP APIs对比表格

![img](https://pic1.zhimg.com/80/v2-45d30c94566bd1aa4a94e8f2cda591e0_720w.jpg)



### **优势**

### **性能**

- gRPC消息使用一种有效的二进制消息格式protobuf进行序列化。Protobuf在服务器和客户机上的序列化非常快。Protobuf序列化后的消息体积很小，能够有效负载，在移动应用程序等有限带宽场景中显得很重要。与采用文本格式的JSON相比，采用二进制格式的protobuf在速度上可以达到前者的5倍！Auth0网站所做的性能测试结果显示，protobuf和JSON的优势差异在Java、Python等环境中尤为明显。下图是Auth0在两个Spring Boot应用程序间所做的对比测试结果。

![img](https://pic1.zhimg.com/80/v2-d412005ce0b3599f8d90176fdc7a0810_720w.jpg)

**结果显示，protobuf所需的请求时间最多只有JSON的20%左右，即速度是其5倍!**

- gRPC是为HTTP/2而设计的，它是HTTP的一个主要版本，与HTTP 1.x相比具有显著的性能优势：：
- 二进制框架和压缩。HTTP/2协议在发送和接收方面都很紧凑和高效。通过单个TCP连接复用多个HTTP/2调用。多路复用消除了线头阻塞。

### **代码生成**

- 所有gRPC框架都为代码生成提供了一流的支持。gRPC开发的核心文件是*.proto文件 ，它定义了gRPC服务和消息的约定。根据这个文件，gRPC框架将生成服务基类，消息和完整的客户端代码。
- 通过在服务器和客户端之间共享*.proto文件，可以从端到端生成消息和客户端代码。客户端的代码生成消除了客户端和服务器上的重复消息，并为您创建了一个强类型的客户端。无需编写客户端代码，可在具有许多服务的应用程序中节省大量开发时间。

### **严格的规范**

- 不存在具有JSON的HTTP API的正式规范。开发人员不需要讨论URL，HTTP动词和响应代码的最佳格式。（想想，是用Post还是Get好？使用Get还是用Put好？一想到有选择恐惧症的你是不是又开了纠结，然后浪费了大量的时间）
- 该gRPC规范是规定有关gRPC服务必须遵循的格式。gRPC消除了争论并节省了开发人员的时间，因为gPRC在各个平台和实现之间是一致的。

### **流**

HTTP/2为长期的实时通信流提供了基础。gRPC通过HTTP/2为流媒体提供一流的支持。

gRPC服务支持所有流组合：

- 一元（没有流媒体）： 简单rpc 这就是一般的rpc调用，一个请求对象对应一个返回对象。客户端发起一次请求，服务端响应一个数据，即标准RPC通信。
- 服务器到客户端流：客户端流式rpc 客户端传入多个请求对象，服务端返回一个响应结果。**应用场景：物联网终端向服务器报送数据。**
- 客户端到服务器流：服务端流式rpc 一个请求对象，服务端可以传回多个结果对象。服务端流 RPC 下，客户端发出一个请求，但不会立即得到一个响应，而是在服务端与客户端之间建立一个单向的流，服务端可以随时向流中写入多个响应消息，最后主动关闭流，而客户端需要监听这个流，不断获取响应直到流关闭。**应用场景举例：典型的例子是客户端向服务端发送一个股票代码，服务端就把该股票的实时数据源源不断的返回给客户端。**
- 双向流媒体：双向流式rpc 结合客户端流式rpc和服务端流式rpc，可以传入多个对象，返回多个响应对象。**应用场景：聊天应用。**

### **截至时间/超时和取消**

- gRPC允许客户端指定他们愿意等待RPC完成的时间。该期限被发送到服务端，服务端可以决定在超出了限期时采取什么行动。例如，服务器可能会在超时时取消正在进行的gRPC / HTTP /数据库请求。
- 通过子gRPC调用截至时间和取消操作有助于实施资源使用限制。

### **劣势**

### **浏览器支持有限**

当下，不可能直接从浏览器调用gRPC服务。**gRPC大量使用HTTP/2功能，没有浏览器提供支持gRPC客户机的Web请求所需的控制级别**。例如，浏览器不允许调用者要求使用的HTTP/2，或者提供对底层HTTP/2框架的访问。

gRPC Web是gRPC团队的一项附加技术，它在浏览器中提供有限的gRPC支持。gRPC Web由两部分组成：支持所有现代浏览器的JavaScript客户端和服务器上的gRPC Web代理。gRPC Web客户端调用代理，代理将在gRPC请求上转发到gRPC服务器。

gRPC Web并非支持所有gRPC功能。不支持客户端和双向流，并且对服务器流的支持有限。

### **不是人类可读的**

HTTP API请求以文本形式发送，可以由人读取和创建。

默认情况下，gRPC消息使用protobuf编码。**虽然protobuf的发送和接收效率很高，但它的二进制格式是不可读的*8。protobuf需要在*.proto文件中指定的消息接口描述才能正确反序列化。需要额外的工具来分析线路上的Protobuf有效负载，并手工编写请求。

存在诸如服务器反射和gRPC命令行工具等功能，以帮助处理二进制protobuf消息。另外，Protobuf消息支持与JSON之间的转换。内置的JSON转换提供了一种有效的方法，可以在调试时将Protobuf消息转换为可读的形式。

## **gRPC场景**

### **适合场景**

gRPC非常适合以下场景：

- 微服务 - gRPC设计为低延迟和高吞吐量通信。gRPC非常适用于效率至关重要的轻型微服务。
- 点对点实时通信 - gRPC对双向流媒体提供出色的支持。gRPC服务可以实时推送消息而无需轮询。
- 多语言混合开发环境 - gRPC工具支持所有流行的开发语言，使gRPC成为多语言开发环境的理想选择。
- 网络受限环境 - 使用Protobuf（一种轻量级消息格式）序列化gRPC消息。gRPC消息始终小于等效的JSON消息。

### **不建议使用场景**

在以下场景中，建议使用其他框架而不是gRPC：

- 浏览器可访问的API - 浏览器不完全支持gRPC。gRPC-Web可以提供浏览器支持，但它有局限性并引入了服务器代理。
- 广播实时通信 - gRPC支持通过流媒体进行实时通信，但不存在向已注册连接广播消息的概念。例如，在应该将新聊天消息发送到聊天室中的所有客户端的聊天室场景中，需要每个gRPC呼叫以单独地将新的聊天消息流传输到客户端。对于这种场景，SignalR是这种情况的有用框架。SignalR具有持久连接的概念和对广播消息的内置支持。
- 进程间通信 - 进程必须承载HTTP/2服务才能接受传入的gRPC调用。对于Windows，进程间通信管道是一种快速，轻量级的通信方法。

## **protobuf**

protobuf 即 Protocol Buffers，**是一种轻便高效的结构化数据存储格式，与语言、平台无关，可扩展可序列化**。

**Protocol Buffers 是一种灵活，高效，自动化机制的结构数据序列化方法－可类比 XML，但是比 XML 更小（3 ~ 10倍）、更快（20 ~ 100倍）、更为简单。json、xml都是基于文本格式，protobuf 是以二进制方式存储的，占用空间小，但也带来了可读性差的缺点**。

protobuf 在通信协议和数据存储等领域应用广泛。例如著名的分布式缓存工具 Memcached 的 Go 语言版本groupcache 就使用了 protobuf 作为其 RPC 数据格式。

Protobuf 在 .proto 定义需要处理的结构化数据，可以通过 **protoc 工具**，将 .proto 文件转换为 C、C++、Golang、Java、Python 等多种语言的代码，兼容性好，易于使用。

![img](https://pic3.zhimg.com/80/v2-9171799f63043db93e5d1a135415d49e_720w.jpg)

### **protobuf示例**

接下来，我们创建一个非常简单的示例，student.proto

```text
syntax = "proto3";
package main;

// this is a comment
message Student {
  string name = 1;
  bool male = 2;
  repeated int32 scores = 3;
}
```

逐行解读student.proto

- protobuf 有2个版本，默认版本是 proto2，如果需要 proto3，则需要在非空非注释第一行使用 syntax = "proto3" 标明版本。
- package，即包名声明符是可选的，用来防止不同的消息类型有命名冲突。
- 消息类型 使用 message 关键字定义，Student 是类型名，name, male, scores 是该类型的 3 个字段，类型分别为 string, bool 和 []int32。字段可以是标量类型，也可以是合成类型。
- 每个字段的修饰符默认是 singular，一般省略不写，repeated 表示字段可重复，即用来表示数组类型。
- 每个字符 =后面的数字称为标识符，每个字段都需要提供一个唯一的标识符。标识符用来在消息的二进制格式中识别各个字段，一旦使用就不能够再改变，标识符的取值范围为 [1, 2^29 - 1] 。
- .proto 文件可以写注释，单行注释 //，多行注释 /* ... */
- 一个 .proto 文件中可以写多个消息类型，即对应多个结构体(struct)。

一个完整的示例。

```text
syntax = "proto3";
package greeting.v3;

import "google/api/annotations.proto";

option go_package = "github.com/garystafford/pb-greeting/gen/go/greeting/v3";

message Greeting {
  string id = 1;
  string service = 2;
  string message = 3;
  string created = 4;
  string hostname = 5;
}

message GreetingRequest {
  Greeting greeting = 1;
}

message GreetingResponse {
  repeated Greeting greeting = 1;
}

service GreetingService {
  rpc Greeting (GreetingRequest) returns (GreetingResponse) {
    option (google.api.http) = {
      get: "/api/greeting"
    };
  }
}
```

### **字段类型**

### **标量类型**

![img](https://pic4.zhimg.com/80/v2-0dadbc0328b8330682f846818de5c06f_720w.jpg)

### **枚举(Enumerations)**

枚举类型适用于提供一组预定义的值，选择其中一个。例如我们将性别定义为枚举类型。

```text
message Student {
  string name = 1;
  enum Gender {
    FEMALE = 0;
    MALE = 1;
  }
  Gender gender = 2;
  repeated int32 scores = 3;
}
```

- 枚举类型的第一个选项的标识符必须是0，这也是枚举类型的默认值。
- 别名（Alias），允许为不同的枚举值赋予相同的标识符，称之为别名，需要打开allow_alias选项。

```text
message EnumAllowAlias {
  enum Status {
    option allow_alias = true;
    UNKOWN = 0;
    STARTED = 1;
    RUNNING = 1;
  }
}
```

### **使用其他消息类型**

Result是另一个消息类型，在 SearchReponse 作为一个消息字段类型使用。

```text
message SearchResponse {
  repeated Result results = 1; 
}

message Result {
  string url = 1;
  string title = 2;
  repeated string snippets = 3;
}
```

嵌套写也是支持的：

```text
message SearchResponse {
  message Result {
    string url = 1;
    string title = 2;
    repeated string snippets = 3;
  }
  repeated Result results = 1;
}
```

如果定义在其他文件中，可以导入其他消息类型来使用：

```text
import "myproject/other_protos.proto";
```

### **任意类型(Any)**

Any 可以表示不在 .proto 中定义任意的内置类型。

```text
import "google/protobuf/any.proto";

message ErrorStatus {
  string message = 1;
  repeated google.protobuf.Any details = 2;
}
```

### **oneof**

```text
message SampleMessage {
  oneof test_oneof {
    string name = 4;
    SubMessage sub_message = 9;
  }
}
```

### **map**

```text
map<key_type, value_type> map_field = N;
```

key_type可以是任何整数或字符串类型（除浮点类型和字节之外的任何标量类型）。请注意，枚举不是有效的key_type。

value_type 可以是除另一个映射之外的任何类型。

```text
syntax = "proto3";
message Product
{
    string name = 1; // 商品名
    // 定义一个k/v类型，key是string类型，value也是string类型
    map<string, string> attrs = 2; // 商品属性，键值对
}
```

> ❝ Map 字段不能使用repeated关键字修饰。

### **定义服务(Services)**

如果消息类型是用来远程通信的(Remote Procedure Call, RPC)，可以在 .proto 文件中定义 RPC 服务接口。例如我们定义了一个名为 SearchService 的 RPC 服务，提供了 Search 接口，入参是 SearchRequest 类型，返回类型是 SearchResponse

```text
service SearchService {
  rpc Search (SearchRequest) returns (SearchResponse);
}
```

### **proto3 与 proto2 的区别**

- 在第一行非空白非注释行，必须写：syntax = "proto3";
- 移除了 “required”，并把 “optional” 改名为"singular"；
- 语言增加 Go、Ruby、JavaNano 支持；
- 移除了 default 选项；
- 枚举类型的第一个字段必须为 0 ；
- 移除了对分组的支持；
- 分组的功能完全可以用消息嵌套的方式来实现，
- 旧代码在解析新增字段时，会把不认识的字段丢弃，再序列化后新增的字段就没了；
- 移除了对扩展的支持，新增了 Any 类型；
- Any 类型是用来替代 proto2 中的扩展。
- 增加了 JSON 映射特性；

### **推荐风格**

- 文件(Files)

- - 文件名使用小写下划线的命名风格，例如 lower_snake_case.proto
  - 每行不超过 80 字符
  - 使用 2 个空格缩进

- 包(Packages)

- - 包名应该和目录结构对应，例如文件在my/package/目录下，包名应为 my.package

- 消息和字段(Messages & Fields)

- - 消息名使用首字母大写驼峰风格(CamelCase)，例如message StudentRequest { ... }
  - 字段名使用小写下划线的风格，例如 string status_code = 1
  - 枚举类型，枚举名使用首字母大写驼峰风格，例如 enum FooBar，枚举值使用全大写下划线隔开的风格(CAPITALS_WITH_UNDERSCORES )，例如 FOO_DEFAULT=1

- 服务(Services)

- - RPC 服务名和方法名，均使用首字母大写驼峰风格，例如service FooService{ rpc GetSomething() }



参照：

- https://zhuanlan.zhihu.com/p/411315625