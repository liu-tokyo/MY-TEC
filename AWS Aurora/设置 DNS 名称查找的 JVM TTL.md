# 设置 DNS 名称查找的 JVM TTL

- https://docs.amazonaws.cn/sdk-for-java/v1/developer-guide/java-dg-jvm-ttl.html

Java 虚拟机 (JVM) 缓存 DNS 名称查找。当 JVM 将主机名解析为 IP 地址时，它会在指定时间段内 (称为*time-to-live*(TTL)。

由于 Amazon 资源使用偶尔变更的 DNS 名称条目，因此建议您为 JVM 配置的 TTL 值不超过 60 秒。这可确保在资源的 IP 地址发生更改时，您的应用程序将能够通过重新查询 DNS 来接收和使用资源的新 IP 地址。

对于一些 Java 配置，将设置 JVM 默认 TTL，以便在重新启动 JVM 之前*绝不* 刷新 DNS 条目。因此，如果一个的 IP 地址Amazon在应用程序仍在运行时资源发生更改，则在您的应用程序之前将无法使用该资源。*手动启动*将刷新 JVM 和缓存的 IP 信息。在此情况下，设置 JVM 的 TTL，以便定期刷新其缓存的 IP 信息是极为重要的。

**注意**

默认 TTL 是变化的，具体取决于 JVM 的版本以及是否安装[安全管理器](http://docs.oracle.com/javase/tutorial/essential/environment/security.html)。许多 JVM 提供的默认 TTL 小于 60 秒。如果您使用此类 JVM 并且未使用安全管理器，则您可以忽略本主题的剩余内容。

## 如何设置 JVM TTL

要修改 JVM 的 TTL，请设置 [networkaddress.cache.ttl](http://docs.oracle.com/javase/7/docs/technotes/guides/net/properties.html) 属性值。根据您的需求，使用下列方法之一：

- **全局 (针对所有使用 JVM 的应用程序)**。在 `$JAVA_HOME/jre/lib/security/java.security` 文件中设置 `networkaddress.cache.ttl`：

  ```yaml
  networkaddress.cache.ttl=60
  ```

- **仅针对应用程序**，在应用程序的初始化代码中设置 `networkaddress.cache.ttl`：

  ```java
  java.security.Security.setProperty("networkaddress.cache.ttl" , "60");
  ```