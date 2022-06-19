# 通过 maven 插件自动根据 proto 文件生成 java 代码

## 目录
- [通过 maven 插件自动根据 proto 文件生成 java 代码](#通过-maven-插件自动根据-proto-文件生成-java-代码)
  - [目录](#目录)
- [1 问题：gRPC 官方文档不够详细](#1-问题grpc-官方文档不够详细)
- [2 通过 maven 构建 java 工程](#2-通过-maven-构建-java-工程)
  - [2.1 pom核心依赖](#21-pom核心依赖)
  - [2.2 pom配置 proto 插件](#22-pom配置-proto-插件)
- [3 定义 proto 文件](#3-定义-proto-文件)
- [4 通过 maven 插件根据 proto 生成 java 代码](#4-通过-maven-插件根据-proto-生成-java-代码)
- [5 gRPC-java，server 端代码示例](#5-grpc-javaserver-端代码示例)
- [6 gRPC-java，client 端代码示例](#6-grpc-javaclient-端代码示例)
- [7 gRPC-java示例代码运行结果](#7-grpc-java示例代码运行结果)
- [8 参考](#8-参考)

---

# 1 问题：gRPC 官方文档不够详细

在调研 gRPC java 时遇到一个问题，根据官方文档，没有办法一次性就把示例跑成功。

而是花了整整两天时间，翻了各种文档才搞清楚，proto compiler、maven、gRPC-java 这几个之间的关系。

现在提供一个端到端的，能够保证一次性就跑起来的 gRPC-java 示例程序。

# 2 通过 maven 构建 java 工程

java version: 1.8 gRPC version: 1.29.0 pom.xml 核心配置部分

## 2.1 pom核心依赖

```javascript
<dependency>
<groupId>io.grpc</groupId>
<artifactId>grpc-netty-shaded</artifactId>
<version>1.29.0</version>
</dependency>
<dependency>
<groupId>io.grpc</groupId>
<artifactId>grpc-protobuf</artifactId>
<version>1.29.0</version>
</dependency>
<dependency>
<groupId>io.grpc</groupId>
<artifactId>grpc-stub</artifactId>
<version>1.29.0</version>
</dependency>
<!-- necessary for Java 9+ -->
<!--java 低于 1.8 的版本不需要此依赖-->
<dependency>
<groupId>org.apache.tomcat</groupId>
<artifactId>annotations-api</artifactId>
<version>6.0.53</version>
<scope>provided</scope>
</dependency>
```

## 2.2 pom配置 proto 插件

os-maven-plugin：此插件可以检测当前系统信息 ${os.detected.classifier}：这个变量获取操作系统的版本，例如osx-x86_64

```javascript
<build>
<extensions>
<extension>
<groupId>kr.motd.maven</groupId>
<artifactId>os-maven-plugin</artifactId>
<version>1.6.2</version>
</extension>
</extensions>
<plugins>
<plugin>
<groupId>org.xolstice.maven.plugins</groupId>
<artifactId>protobuf-maven-plugin</artifactId>
<version>0.6.1</version>
<configuration>
<protocArtifact>com.google.protobuf:protoc:3.11.0:exe:${os.detected.classifier}</protocArtifact>
<pluginId>grpc-java</pluginId>
<pluginArtifact>io.grpc:protoc-gen-grpc-java:1.29.0:exe:${os.detected.classifier}</pluginArtifact>
</configuration>
<executions>
<execution>
<goals>
<goal>compile</goal>
<goal>compile-custom</goal>
</goals>
</execution>
</executions>
</plugin>
</plugins>
</build>
```

# 3 定义 proto 文件

在 src/main/proto 目录下放 helloworld.proto 文件

```javascript
syntax = "proto3";
option java_generic_services = true;
option java_multiple_files = true;
option java_package = "io.grpc.examples.helloworld";
// The greeting service definition.
service Greeter {
// Sends a greeting
rpc SayHello (HelloRequest) returns (HelloReply) {}
}
// The request message containing the user's name.
message HelloRequest {
string name = 1;
}
// The response message containing the greetings
message HelloReply {
string message = 1;
}
```

# 4 通过 maven 插件根据 proto 生成 java 代码

执行 `mvn compile`命令，自动生成代码。 默认生成的代码在，target/generated-sources/protobuf 目录下。 其中 grpc-java 目录下放的是生成的 Service 对应的类，java 目录下放的是生成的message 对应的 java对象。

# 5 gRPC-java，server 端代码示例

直接运行 main 函数，服务端就开始工作。

```javascript
package io.grpc.examples.helloworld;
import java.io.IOException;
import java.util.concurrent.TimeUnit;
import io.grpc.Server;
import io.grpc.ServerBuilder;
import io.grpc.stub.StreamObserver;
/**
* @ClassName HelloWorldServer
* @Description
* @Author hugo_lei
* @Date 13/5/20 上午11:09
*/
public class HelloWorldServer {
private Server server;
private void start() throws IOException {
/* The port on which the server should run */
int port = 50051;
server = ServerBuilder.forPort(port)
.addService(new GreeterImpl())
.build()
.start();
Runtime.getRuntime().addShutdownHook(new Thread() {
@Override
public void run() {
// Use stderr here since the logger may have been reset by its JVM shutdown hook.
System.err.println("*** shutting down gRPC server since JVM is shutting down");
try {
HelloWorldServer.this.stop();
} catch (InterruptedException e) {
e.printStackTrace(System.err);
}
System.err.println("*** server shut down");
}
});
}
private void stop() throws InterruptedException {
if (server != null) {
server.shutdown().awaitTermination(30, TimeUnit.SECONDS);
}
}
/**
* Await termination on the main thread since the grpc library uses daemon threads.
*/
private void blockUntilShutdown() throws InterruptedException {
if (server != null) {
server.awaitTermination();
}
}
/**
* Main launches the server from the command line.
*/
public static void main(String[] args) throws IOException, InterruptedException {
final HelloWorldServer server = new HelloWorldServer();
server.start();
server.blockUntilShutdown();
}
static class GreeterImpl extends GreeterGrpc.GreeterImplBase {
@Override
public void sayHello(HelloRequest req, StreamObserver<HelloReply> responseObserver) {
HelloReply reply = HelloReply.newBuilder().setMessage("Hello " + req.getName()).build();
System.out.println("=====server=====");
System.out.println("server: Hello " + req.getName());
responseObserver.onNext(reply);
responseObserver.onCompleted();
}
}
}
```

# 6 gRPC-java，client 端代码示例

每执行一次 main 函数，client 就相当于向 server 发送了一次请求。

```javascript
package io.grpc.examples.helloworld;
import java.util.concurrent.TimeUnit;
import io.grpc.Channel;
import io.grpc.ManagedChannel;
import io.grpc.ManagedChannelBuilder;
import io.grpc.StatusRuntimeException;
/**
* @ClassName HelloWorldClient
* @Description
* @Author hugo_lei
* @Date 13/5/20 上午11:13
*/
public class HelloWorldClient {
private final GreeterGrpc.GreeterBlockingStub blockingStub;
/** Construct client for accessing HelloWorld server using the existing channel. */
public HelloWorldClient(Channel channel) {
// 'channel' here is a Channel, not a ManagedChannel, so it is not this code's responsibility to
// shut it down.
// Passing Channels to code makes code easier to test and makes it easier to reuse Channels.
blockingStub = GreeterGrpc.newBlockingStub(channel);
}
/** Say hello to server. */
public void greet(String name) {
HelloRequest request = HelloRequest.newBuilder().setName(name).build();
HelloReply response;
try {
response = blockingStub.sayHello(request);
} catch (StatusRuntimeException e) {
return;
}
System.out.println("Greeting: " + response.getMessage());
}
/**
* Greet server. If provided, the first element of {@code args} is the name to use in the
* greeting. The second argument is the target server.
*/
public static void main(String[] args) throws Exception {
String user = "hahahahah";
// Access a service running on the local machine on port 50051
String target = "localhost:50051";
// Allow passing in the user and target strings as command line arguments
if (args.length > 0) {
if ("--help".equals(args[0])) {
System.err.println("Usage: [name [target]]");
System.err.println("");
System.err.println(" name The name you wish to be greeted by. Defaults to " + user);
System.err.println(" target The server to connect to. Defaults to " + target);
System.exit(1);
}
user = args[0];
}
if (args.length > 1) {
target = args[1];
}
// Create a communication channel to the server, known as a Channel. Channels are thread-safe
// and reusable. It is common to create channels at the beginning of your application and reuse
// them until the application shuts down.
ManagedChannel channel = ManagedChannelBuilder.forTarget(target)
// Channels are secure by default (via SSL/TLS). For the example we disable TLS to avoid
// needing certificates.
.usePlaintext()
.build();
try {
HelloWorldClient client = new HelloWorldClient(channel);
client.greet(user);
} finally {
// ManagedChannels use resources like threads and TCP connections. To prevent leaking these
// resources the channel should be shut down when it will no longer be used. If it may be used
// again leave it running.
channel.shutdownNow().awaitTermination(5, TimeUnit.SECONDS);
}
}
}
```

# 7 gRPC-java示例代码运行结果

# 8 参考

1. [grpc-java](https://github.com/grpc/grpc-java)
2. [os-maven-plugin](https://github.com/trustin/os-maven-plugin#issues-with-eclipse-m2e-or-other-ides)
3. [protobuf-maven-plugin](https://www.xolstice.org/protobuf-maven-plugin/)

本文参与[腾讯云自媒体分享计划](https://copyfuture.com/developer/support-plan)，欢迎正在阅读的你也加入，一起分享。