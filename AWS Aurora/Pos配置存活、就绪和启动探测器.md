# Pos配置存活、就绪和启动探测器

这篇文章介绍如何给容器配置活跃（Liveness）、就绪（Readiness）和启动（Startup）探测器。

- https://kubernetes.io/zh/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/

[kubelet](https://kubernetes.io/zh/docs/reference/command-line-tools-reference/kubelet/) 使用存活探测器来确定什么时候要重启容器。 例如，存活探测器可以探测到应用死锁（应用程序在运行，但是无法继续执行后面的步骤）情况。 重启这种状态下的容器有助于提高应用的可用性，即使其中存在缺陷。

kubelet 使用就绪探测器可以知道容器何时准备好接受请求流量，当一个 Pod 内的所有容器都就绪时，才能认为该 Pod 就绪。 这种信号的一个用途就是控制哪个 Pod 作为 Service 的后端。 若 Pod 尚未就绪，会被从 Service 的负载均衡器中剔除。

kubelet 使用启动探测器来了解应用容器何时启动。 如果配置了这类探测器，你就可以控制容器在启动成功后再进行存活性和就绪态检查， 确保这些存活、就绪探测器不会影响应用的启动。 启动探测器可以用于对慢启动容器进行存活性检测，避免它们在启动运行之前就被杀掉。

## Before you begin

你必须拥有一个 Kubernetes 的集群，同时你的 Kubernetes 集群必须带有 kubectl 命令行工具。 建议在至少有两个节点的集群上运行本教程，且这些节点不作为控制平面主机。 如果你还没有集群，你可以通过 [Minikube](https://minikube.sigs.k8s.io/docs/tutorials/multi_node/) 构建一个你自己的集群，或者你可以使用下面任意一个 Kubernetes 工具构建：

- [Katacoda](https://www.katacoda.com/courses/kubernetes/playground)
- [玩转 Kubernetes](http://labs.play-with-k8s.com/)

## 定义存活命令

许多长时间运行的应用最终会进入损坏状态，除非重新启动，否则无法被恢复。 Kubernetes 提供了存活探测器来发现并处理这种情况。

在本练习中，你会创建一个 Pod，其中运行一个基于 `k8s.gcr.io/busybox` 镜像的容器。 下面是这个 Pod 的配置文件。

[`pods/probe/exec-liveness.yaml` ](https://raw.githubusercontent.com/kubernetes/website/main/content/zh/examples/pods/probe/exec-liveness.yaml)![Copy pods/probe/exec-liveness.yaml to clipboard](https://d33wubrfki0l68.cloudfront.net/0901162ab78eb4ff2e9e5dc8b17c3824befc91a6/44ccd/images/copycode.svg)

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    test: liveness
  name: liveness-exec
spec:
  containers:
  - name: liveness
    image: k8s.gcr.io/busybox
    args:
    - /bin/sh
    - -c
    - touch /tmp/healthy; sleep 30; rm -rf /tmp/healthy; sleep 600
    livenessProbe:
      exec:
        command:
        - cat
        - /tmp/healthy
      initialDelaySeconds: 5
      periodSeconds: 5
```

在这个配置文件中，可以看到 Pod 中只有一个 `Container`。 `periodSeconds` 字段指定了 kubelet 应该每 5 秒执行一次存活探测。 `initialDelaySeconds` 字段告诉 kubelet 在执行第一次探测前应该等待 5 秒。 kubelet 在容器内执行命令 `cat /tmp/healthy` 来进行探测。 如果命令执行成功并且返回值为 0，kubelet 就会认为这个容器是健康存活的。 如果这个命令返回非 0 值，kubelet 会杀死这个容器并重新启动它。

当容器启动时，执行如下的命令：

```shell
/bin/sh -c "touch /tmp/healthy; sleep 30; rm -rf /tmp/healthy; sleep 600"
```

这个容器生命的前 30 秒，`/tmp/healthy` 文件是存在的。 所以在这最开始的 30 秒内，执行命令 `cat /tmp/healthy` 会返回成功代码。 30 秒之后，执行命令 `cat /tmp/healthy` 就会返回失败代码。

创建 Pod：

```shell
kubectl apply -f https://k8s.io/examples/pods/probe/exec-liveness.yaml
```

在 30 秒内，查看 Pod 的事件：

```shell
kubectl describe pod liveness-exec
```

输出结果表明还没有存活探测器失败：

```
FirstSeen    LastSeen    Count   From            SubobjectPath           Type        Reason      Message
--------- --------    -----   ----            -------------           --------    ------      -------
24s       24s     1   {default-scheduler }                    Normal      Scheduled   Successfully assigned liveness-exec to worker0
23s       23s     1   {kubelet worker0}   spec.containers{liveness}   Normal      Pulling     pulling image "k8s.gcr.io/busybox"
23s       23s     1   {kubelet worker0}   spec.containers{liveness}   Normal      Pulled      Successfully pulled image "k8s.gcr.io/busybox"
23s       23s     1   {kubelet worker0}   spec.containers{liveness}   Normal      Created     Created container with docker id 86849c15382e; Security:[seccomp=unconfined]
23s       23s     1   {kubelet worker0}   spec.containers{liveness}   Normal      Started     Started container with docker id 86849c15382e
```

35 秒之后，再来看 Pod 的事件：

```shell
kubectl describe pod liveness-exec
```

在输出结果的最下面，有信息显示存活探测器失败了，这个容器被杀死并且被重建了。

```
FirstSeen LastSeen    Count   From            SubobjectPath           Type        Reason      Message
--------- --------    -----   ----            -------------           --------    ------      -------
37s       37s     1   {default-scheduler }                    Normal      Scheduled   Successfully assigned liveness-exec to worker0
36s       36s     1   {kubelet worker0}   spec.containers{liveness}   Normal      Pulling     pulling image "k8s.gcr.io/busybox"
36s       36s     1   {kubelet worker0}   spec.containers{liveness}   Normal      Pulled      Successfully pulled image "k8s.gcr.io/busybox"
36s       36s     1   {kubelet worker0}   spec.containers{liveness}   Normal      Created     Created container with docker id 86849c15382e; Security:[seccomp=unconfined]
36s       36s     1   {kubelet worker0}   spec.containers{liveness}   Normal      Started     Started container with docker id 86849c15382e
2s        2s      1   {kubelet worker0}   spec.containers{liveness}   Warning     Unhealthy   Liveness probe failed: cat: can't open '/tmp/healthy': No such file or directory
```

再等 30 秒，确认这个容器被重启了：

```shell
kubectl get pod liveness-exec
```

输出结果显示 `RESTARTS` 的值增加了 1。

```
NAME            READY     STATUS    RESTARTS   AGE
liveness-exec   1/1       Running   1          1m
```

## 定义一个存活态 HTTP 请求接口

另外一种类型的存活探测方式是使用 HTTP GET 请求。 下面是一个 Pod 的配置文件，其中运行一个基于 `k8s.gcr.io/liveness` 镜像的容器。

[`pods/probe/http-liveness.yaml` ](https://raw.githubusercontent.com/kubernetes/website/main/content/zh/examples/pods/probe/http-liveness.yaml)![Copy pods/probe/http-liveness.yaml to clipboard](https://d33wubrfki0l68.cloudfront.net/0901162ab78eb4ff2e9e5dc8b17c3824befc91a6/44ccd/images/copycode.svg)

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    test: liveness
  name: liveness-http
spec:
  containers:
  - name: liveness
    image: k8s.gcr.io/liveness
    args:
    - /server
    livenessProbe:
      httpGet:
        path: /healthz
        port: 8080
        httpHeaders:
        - name: Custom-Header
          value: Awesome
      initialDelaySeconds: 3
      periodSeconds: 3
```

在这个配置文件中，你可以看到 Pod 也只有一个容器。 `periodSeconds` 字段指定了 kubelet 每隔 3 秒执行一次存活探测。 `initialDelaySeconds` 字段告诉 kubelet 在执行第一次探测前应该等待 3 秒。 kubelet 会向容器内运行的服务（服务在监听 8080 端口）发送一个 HTTP GET 请求来执行探测。 如果服务器上 `/healthz` 路径下的处理程序返回成功代码，则 kubelet 认为容器是健康存活的。 如果处理程序返回失败代码，则 kubelet 会杀死这个容器并将其重启。

返回大于或等于 200 并且小于 400 的任何代码都标示成功，其它返回代码都标示失败。

你可以访问 [server.go](https://github.com/kubernetes/kubernetes/blob/master/test/images/agnhost/liveness/server.go)。 阅读服务的源码。 容器存活期间的最开始 10 秒中，`/healthz` 处理程序返回 200 的状态码。 之后处理程序返回 500 的状态码。

```go
http.HandleFunc("/healthz", func(w http.ResponseWriter, r *http.Request) {
    duration := time.Now().Sub(started)
    if duration.Seconds() > 10 {
        w.WriteHeader(500)
        w.Write([]byte(fmt.Sprintf("error: %v", duration.Seconds())))
    } else {
        w.WriteHeader(200)
        w.Write([]byte("ok"))
    }
})
```

kubelet 在容器启动之后 3 秒开始执行健康检测。所以前几次健康检查都是成功的。 但是 10 秒之后，健康检查会失败，并且 kubelet 会杀死容器再重新启动容器。

创建一个 Pod 来测试 HTTP 的存活检测：

```shell
kubectl apply -f https://k8s.io/examples/pods/probe/http-liveness.yaml
```

10 秒之后，通过查看 Pod 事件来确认活跃探测器已经失败，并且容器被重新启动了。

```shell
kubectl describe pod liveness-http
```

在 1.13 之前（包括 1.13）的版本中，如果在 Pod 运行的节点上设置了环境变量 `http_proxy`（或者 `HTTP_PROXY`），HTTP 的存活探测会使用这个代理。 在 1.13 之后的版本中，设置本地的 HTTP 代理环境变量不会影响 HTTP 的存活探测。

## 定义 TCP 的存活探测

第三种类型的存活探测是使用 TCP 套接字。 使用这种配置时，kubelet 会尝试在指定端口和容器建立套接字链接。 如果能建立连接，这个容器就被看作是健康的，如果不能则这个容器就被看作是有问题的。

[`pods/probe/tcp-liveness-readiness.yaml` ](https://raw.githubusercontent.com/kubernetes/website/main/content/zh/examples/pods/probe/tcp-liveness-readiness.yaml)![Copy pods/probe/tcp-liveness-readiness.yaml to clipboard](https://d33wubrfki0l68.cloudfront.net/0901162ab78eb4ff2e9e5dc8b17c3824befc91a6/44ccd/images/copycode.svg)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: goproxy
  labels:
    app: goproxy
spec:
  containers:
  - name: goproxy
    image: k8s.gcr.io/goproxy:0.1
    ports:
    - containerPort: 8080
    readinessProbe:
      tcpSocket:
        port: 8080
      initialDelaySeconds: 5
      periodSeconds: 10
    livenessProbe:
      tcpSocket:
        port: 8080
      initialDelaySeconds: 15
      periodSeconds: 20
```

如你所见，TCP 检测的配置和 HTTP 检测非常相似。 下面这个例子同时使用就绪和存活探测器。kubelet 会在容器启动 5 秒后发送第一个就绪探测。 探测器会尝试连接 `goproxy` 容器的 8080 端口。 如果探测成功，这个 Pod 会被标记为就绪状态，kubelet 将继续每隔 10 秒运行一次检测。

除了就绪探测，这个配置包括了一个存活探测。 kubelet 会在容器启动 15 秒后进行第一次存活探测。 与就绪探测类似，活跃探测器会尝试连接 `goproxy` 容器的 8080 端口。 如果存活探测失败，容器会被重新启动。

```shell
kubectl apply -f https://k8s.io/examples/pods/probe/tcp-liveness-readiness.yaml
```

15 秒之后，通过看 Pod 事件来检测存活探测器：

```shell
kubectl describe pod goproxy
```

## 定义 gRPC 活跃探测器

**FEATURE STATE:** `Kubernetes v1.23 [alpha]`

如果你的应用实现了 [gRPC 健康检查协议](https://github.com/grpc/grpc/blob/master/doc/health-checking.md)， kubelet 可以配置为使用该协议来执行应用活跃性检查。 你必须启用 `GRPCContainerProbe` [特性门控](https://kubernetes.io/zh/docs/reference/command-line-tools-reference/feature-gates/) 才能配置依赖于 gRPC 的检查机制。

下面是一个示例清单：

[`pods/probe/grpc-liveness.yaml` ](https://raw.githubusercontent.com/kubernetes/website/main/content/zh/examples/pods/probe/grpc-liveness.yaml)![Copy pods/probe/grpc-liveness.yaml to clipboard](https://d33wubrfki0l68.cloudfront.net/0901162ab78eb4ff2e9e5dc8b17c3824befc91a6/44ccd/images/copycode.svg)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: etcd-with-grpc
spec:
  containers:
  - name: etcd
    image: k8s.gcr.io/etcd:3.5.1-0
    command: [ "/usr/local/bin/etcd", "--data-dir",  "/var/lib/etcd", "--listen-client-urls", "http://0.0.0.0:2379", "--advertise-client-urls", "http://127.0.0.1:2379", "--log-level", "debug"]
    ports:
    - containerPort: 2379
    livenessProbe:
      grpc:
        port: 2379
      initialDelaySeconds: 10
```

要使用 gRPC 探测器，必须配置 `port` 属性。如果健康状态端点配置在非默认服务之上， 你还必须设置 `service` 属性。

**Note:**

与 HTTP 和 TCP 探测器不同，gRPC 探测不能使用命名端口或定制主机。

配置问题（例如：错误的 `port` 和 `service`、未实现健康检查协议） 都被认作是探测失败，这一点与 HTTP 和 TCP 探测器类似。

```shell
kubectl apply -f https://k8s.io/examples/pods/probe/grpc-liveness.yaml
```

15 秒钟之后，查看 Pod 事件确认活跃性检查并未失败：

```shell
kubectl describe pod etcd-with-grpc
```

在 Kubernetes 1.23 之前，gRPC 健康探测通常使用 [grpc-health-probe](https://github.com/grpc-ecosystem/grpc-health-probe/) 来实现，如博客 [Health checking gRPC servers on Kubernetes（对 Kubernetes 上的 gRPC 服务器执行健康检查）](https://kubernetes.io/blog/2018/10/01/health-checking-grpc-servers-on-kubernetes/)所描述。 内置的 gRPC 探测器行为与 `grpc-health-probe` 所实现的行为类似。 从 `grpc-health-probe` 迁移到内置探测器时，请注意以下差异：

- 内置探测器运行时针对的是 Pod 的 IP 地址，不像 `grpc-health-probe` 那样通常针对 `127.0.0.1` 执行探测； 请一定配置你的 gRPC 端点使之监听于 Pod 的 IP 地址之上。
- 内置探测器不支持任何身份认证参数（例如 `tls`）。
- 对于内置的探测器而言，不存在错误代码。所有错误都被视作探测失败。
- 如果 `ExecProbeTimeout` 特性门控被设置为 `false`，则 `grpc-health-probe` 不会考虑 `timeoutSeconds` 设置状态（默认值为 1s）， 而内置探测器则会在超时时返回失败。

## 使用命名端口

对于 HTTP 或者 TCP 存活检测可以使用命名的 [ContainerPort](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.23/#containerport-v1-core)。

```yaml
ports:
- name: liveness-port
  containerPort: 8080
  hostPort: 8080

livenessProbe:
  httpGet:
    path: /healthz
    port: liveness-port
```

## 使用启动探测器保护慢启动容器

有时候，会有一些现有的应用在启动时需要较长的初始化时间。 要这种情况下，若要不影响对死锁作出快速响应的探测，设置存活探测参数是要技巧的。 技巧就是使用相同的命令来设置启动探测，针对 HTTP 或 TCP 检测，可以通过将 `failureThreshold * periodSeconds` 参数设置为足够长的时间来应对糟糕情况下的启动时间。

这样，前面的例子就变成了：

```yaml
ports:
- name: liveness-port
  containerPort: 8080
  hostPort: 8080

livenessProbe:
  httpGet:
    path: /healthz
    port: liveness-port
  failureThreshold: 1
  periodSeconds: 10

startupProbe:
  httpGet:
    path: /healthz
    port: liveness-port
  failureThreshold: 30
  periodSeconds: 10
```

幸亏有启动探测，应用程序将会有最多 5 分钟（30 * 10 = 300s）的时间来完成其启动过程。 一旦启动探测成功一次，存活探测任务就会接管对容器的探测，对容器死锁作出快速响应。 如果启动探测一直没有成功，容器会在 300 秒后被杀死，并且根据 `restartPolicy` 来 执行进一步处置。

## 定义就绪探测器

有时候，应用会暂时性地无法为请求提供服务。 例如，应用在启动时可能需要加载大量的数据或配置文件，或是启动后要依赖等待外部服务。 在这种情况下，既不想杀死应用，也不想给它发送请求。 Kubernetes 提供了就绪探测器来发现并缓解这些情况。 容器所在 Pod 上报还未就绪的信息，并且不接受通过 Kubernetes Service 的流量。

**Note:** 就绪探测器在容器的整个生命周期中保持运行状态。

**Caution:** 活跃探测器 **不等待** 就绪性探测器成功。 如果要在执行活跃探测器之前等待，应该使用 `initialDelaySeconds` 或 `startupProbe`。

就绪探测器的配置和存活探测器的配置相似。 唯一区别就是要使用 `readinessProbe` 字段，而不是 `livenessProbe` 字段。

```yaml
readinessProbe:
  exec:
    command:
    - cat
    - /tmp/healthy
  initialDelaySeconds: 5
  periodSeconds: 5
```

HTTP 和 TCP 的就绪探测器配置也和存活探测器的配置完全相同。

就绪和存活探测可以在同一个容器上并行使用。 两者都可以确保流量不会发给还未就绪的容器，当这些探测失败时容器会被重新启动。

## 配置探测器

[Probe](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.23/#probe-v1-core) 有很多配置字段，可以使用这些字段精确地控制活跃和就绪检测的行为：

- `initialDelaySeconds`：容器启动后要等待多少秒后才启动存活和就绪探测器， 默认是 0 秒，最小值是 0。
- `periodSeconds`：执行探测的时间间隔（单位是秒）。默认是 10 秒。最小值是 1。
- `timeoutSeconds`：探测的超时后等待多少秒。默认值是 1 秒。最小值是 1。
- `successThreshold`：探测器在失败后，被视为成功的最小连续成功数。默认值是 1。 存活和启动探测的这个值必须是 1。最小值是 1。
- `failureThreshold`：当探测失败时，Kubernetes 的重试次数。 对存活探测而言，放弃就意味着重新启动容器。 对就绪探测而言，放弃意味着 Pod 会被打上未就绪的标签。默认值是 3。最小值是 1。

**Note:**

在 Kubernetes 1.20 版本之前，`exec` 探针会忽略 `timeoutSeconds`： 探针会无限期地持续运行，甚至可能超过所配置的限期，直到返回结果为止。

这一缺陷在 Kubernetes v1.20 版本中得到修复。你可能一直依赖于之前错误的探测行为， 甚至都没有觉察到这一问题的存在，因为默认的超时值是 1 秒钟。 作为集群管理员，你可以在所有的 kubelet 上禁用 `ExecProbeTimeout` [特性门控](https://kubernetes.io/zh/docs/reference/command-line-tools-reference/feature-gates/) （将其设置为 `false`），从而恢复之前版本中的运行行为。之后当集群中所有的 exec 探针都设置了 `timeoutSeconds` 参数后，移除此标志重载。 如果你有 Pod 受到此默认 1 秒钟超时值的影响，你应该更新这些 Pod 对应的探针的超时值， 这样才能为最终去除该特性门控做好准备。

当此缺陷被修复之后，在使用 `dockershim` 容器运行时的 Kubernetes `1.20+` 版本中，对于 exec 探针而言，容器中的进程可能会因为超时值的设置保持持续运行， 即使探针返回了失败状态。

**Caution:**

如果就绪态探针的实现不正确，可能会导致容器中进程的数量不断上升。 如果不对其采取措施，很可能导致资源枯竭的状况。

### HTTP 探测 

[HTTP Probes](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.23/#httpgetaction-v1-core) 允许针对 `httpGet` 配置额外的字段：

- `host`：连接使用的主机名，默认是 Pod 的 IP。也可以在 HTTP 头中设置 “Host” 来代替。
- `scheme` ：用于设置连接主机的方式（HTTP 还是 HTTPS）。默认是 "HTTP"。
- `path`：访问 HTTP 服务的路径。默认值为 "/"。
- `httpHeaders`：请求中自定义的 HTTP 头。HTTP 头字段允许重复。
- `port`：访问容器的端口号或者端口名。如果数字必须在 1～65535 之间。

对于 HTTP 探测，kubelet 发送一个 HTTP 请求到指定的路径和端口来执行检测。 除非 `httpGet` 中的 `host` 字段设置了，否则 kubelet 默认是给 Pod 的 IP 地址发送探测。 如果 `scheme` 字段设置为了 `HTTPS`，kubelet 会跳过证书验证发送 HTTPS 请求。 大多数情况下，不需要设置`host` 字段。 这里有个需要设置 `host` 字段的场景，假设容器监听 127.0.0.1，并且 Pod 的 `hostNetwork` 字段设置为了 `true`。那么 `httpGet` 中的 `host` 字段应该设置为 127.0.0.1。 可能更常见的情况是如果 Pod 依赖虚拟主机，你不应该设置 `host` 字段，而是应该在 `httpHeaders` 中设置 `Host`。

针对 HTTP 探针，kubelet 除了必需的 `Host` 头部之外还发送两个请求头部字段： `User-Agent` 和 `Accept`。这些头部的默认值分别是 `kube-probe/{{ skew latestVersion >}}` （其中 `1.23` 是 kubelet 的版本号）和 `*/*`。

你可以通过为探测设置 `.httpHeaders` 来重载默认的头部字段值；例如：

```yaml
livenessProbe:
  httpGet:
    httpHeaders:
      - name: Accept
        value: application/json

startupProbe:
  httpGet:
    httpHeaders:
      - name: User-Agent
        value: MyUserAgent
```

你也可以通过将这些头部字段定义为空值，从请求中去掉这些头部字段。

```yaml
livenessProbe:
  httpGet:
    httpHeaders:
      - name: Accept
        value: ""

startupProbe:
  httpGet:
    httpHeaders:
      - name: User-Agent
        value: ""
```

### TCP 探测 

对于 TCP 探测而言，kubelet 在节点上（不是在 Pod 里面）发起探测连接， 这意味着你不能在 `host` 参数上配置服务名称，因为 kubelet 不能解析服务名称。

### 探测器层面的 `terminationGracePeriodSeconds`

**FEATURE STATE:** `Kubernetes v1.22 [beta]`

在 1.21 发行版之前，Pod 层面的 `terminationGracePeriodSeconds` 被用来终止活跃探测或启动探测失败的容器。 这一行为上的关联不是我们想要的，可能导致 Pod 层面设置了 `terminationGracePeriodSeconds` 时容器要花非常长的时间才能重新启动。

在 1.21 及更高版本中，当特性门控 `ProbeTerminationGracePeriod` 被启用时， 用户可以指定一个探测器层面的 `terminationGracePeriodSeconds` 作为探测器规约的一部分。 当该特性门控被启用，并且 Pod 层面和探测器层面的 `terminationGracePeriodSeconds` 都已设置，kubelet 将使用探测器层面设置的值。

在 Kubernetes 1.22 中，`ProbeTerminationGracePeriod` 特性门控只能用在 API 服务器上。 kubelet 始终遵守探针级别 `terminationGracePeriodSeconds` 字段 （如果它存在于 Pod 上）。

如果你已经为现有 Pod 设置了 `terminationGracePeriodSeconds` 字段并且不再希望使用针对每个探针的终止宽限期，则必须删除现有的这类 Pod。

当你（或控制平面或某些其他组件）创建替换 Pod，并且特性门控 `ProbeTerminationGracePeriod` 被禁用时，API 服务器会忽略 Pod 级别的 `terminationGracePeriodSeconds` 字段设置， 即使 Pod 或 Pod 模板指定了它。

例如:

```yaml
spec:
  terminationGracePeriodSeconds: 3600  # pod-level
  containers:
  - name: test
    image: ...

    ports:
    - name: liveness-port
      containerPort: 8080
      hostPort: 8080

    livenessProbe:
      httpGet:
        path: /healthz
        port: liveness-port
      failureThreshold: 1
      periodSeconds: 60
      # Override pod-level terminationGracePeriodSeconds #
      terminationGracePeriodSeconds: 60
```

探测器层面的 `terminationGracePeriodSeconds` 不能用于就绪态探针。 这一设置将被 API 服务器拒绝。