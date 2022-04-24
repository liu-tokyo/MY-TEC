# Amazon Aurora PostgreSQL 的最佳实践

- https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/AuroraPostgreSQL.BestPractices.html

一个基本的最佳实践是学习如何管理您的 Amazon Aurora PostgreSQL 数据库集群的性能和扩展，以及了解基本的维护任务。有关更多信息，请参阅 [管理 Amazon Aurora PostgreSQL](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/AuroraPostgreSQL.Managing.html)。

另一个重要的最佳实践是了解如何使用 Aurora PostgreSQL 的关键功能，例如快速故障转移。在下文中，您可以了解如何确保故障转移能够尽快发生。要在故障转移后快速恢复，您可以对 Aurora PostgreSQL 数据库集群使用集群缓存管理。有关更多信息，请参阅[使用 Aurora PostgreSQL 的集群缓存管理进行故障转移后的快速恢复](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/AuroraPostgreSQL.cluster-cache-mgmt.html)。

## 使用 Amazon Aurora PostgreSQL 进行快速故障转移

您可以通过多种方式使用 Aurora PostgreSQL 加快故障转移的执行速度。本节讨论以下每种方法：

- 积极设置 TCP keepalives 以确保在发生故障时等待服务器响应的较长运行查询将在读取超时到期之前停止。
- 积极设置 Java DNS 缓存超时，以确保 Aurora 只读终端节点可以在后续连接尝试中正确循环通过只读节点。
- 将 JDBC 连接字符串中使用的超时变量设置得尽可能低。对短期和长期运行的查询使用单独的连接对象。
- 使用提供的读写 Aurora 端点建立与集群的连接。
- 使用 RDS API 测试应用程序对服务器端故障的响应，并使用丢包工具测试应用程序对客户端故障的响应。
- 使用 AWS JDBC Driver for PostgreSQL（预览版）充分利用 Aurora PostgreSQL 的故障转移功能。有关适用于 PostgreSQL 的 AWS JDBC 驱动程序的更多信息以及使用它的完整说明，请参阅 [适用于 PostgreSQL 的 AWS JDBC 驱动程序 GitHub 存储库](https://awslabs.github.io/aws-postgresql-jdbc/).

**话题**

- [设置 TCP keepalives 参数](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/AuroraPostgreSQL.BestPractices.html#AuroraPostgreSQL.BestPractices.FastFailover.TCPKeepalives)
- [配置应用程序以实现快速故障转移](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/AuroraPostgreSQL.BestPractices.html#AuroraPostgreSQL.BestPractices.FastFailover.Configuring)
- [测试故障转移](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/AuroraPostgreSQL.BestPractices.html#AuroraPostgreSQL.BestPracticesFastFailover.Testing)
- [快速故障转移 Java 示例](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/AuroraPostgreSQL.BestPractices.html#AuroraPostgreSQL.BestPractices.FastFailover.Example)

### 设置 TCP keepalives 参数

TCP keepalive 过程很简单：当你建立一个 TCP 连接时，你关联了一组定时器。当保活计时器达到零时，您发送保活探测数据包。如果您收到对 keepalive 探测的回复，则可以假设连接仍在运行。

启用 TCP keepalive 参数并积极设置它们可确保如果您的客户端不再能够连接到数据库，那么任何活动连接都会快速关闭。此操作允许应用程序做出适当的反应，例如选择要连接的新主机。

您需要设置以下 TCP keepalive 参数：

- `tcp_keepalive_time`控制时间，以秒为单位，当套接字没有发送数据（ACK 不被视为数据）时，发送 keepalive 数据包。我们推荐以下设置：

  `tcp_keepalive_time = 1`

- `tcp_keepalive_intvl`控制发送初始数据包后发送后续保活数据包之间的时间（以秒为单位）（使用`tcp_keepalive_time`参数设置）。我们推荐以下设置：

  `tcp_keepalive_intvl = 1`

- `tcp_keepalive_probes`是在通知应用程序之前发生的未确认的保活探测的数量。我们推荐以下设置：

  `tcp_keepalive_probes = 5`

当数据库停止响应时，这些设置应在五秒内通知应用程序。如果 keepalive 数据包经常在应用程序的网络中被丢弃，则可以设置更高的*tcp_keepalive_probes 值。*这随后会增加检测实际故障所需的时间，但允许在不太可靠的网络中提供更多缓冲。

**在 Linux 上设置 TCP keepalive 参数**

1. 在测试如何配置 TCP keepalive 参数时，我们建议通过命令行使用以下命令进行配置： 此建议配置是系统范围的，这意味着它会影响所有其他使用 SO_KEEPALIVE 选项创建套接字的应用程序。

   ```
   sudo sysctl net.ipv4.tcp_keepalive_time=1
   sudo sysctl net.ipv4.tcp_keepalive_intvl=1
   sudo sysctl net.ipv4.tcp_keepalive_probes=5
   ```

2. 找到适用于您的应用程序的配置后，通过将以下行添加到*/etc/sysctl.conf*来保留这些设置，包括您所做的任何更改：

   ```
   tcp_keepalive_time = 1
   tcp_keepalive_intvl = 1
   tcp_keepalive_probes = 5
   ```

有关在 Windows 上设置 TCP keepalive 参数的信息，请参阅[有关 TCP keepalive 您可能想知道的事情](https://blogs.technet.microsoft.com/nettracer/2010/06/03/things-that-you-may-want-to-know-about-tcp-keepalives/).

### 配置应用程序以实现快速故障转移

本部分讨论您可以进行的几项 Aurora PostgreSQL 特定配置更改。要了解有关 PostgreSQL JDBC 驱动程序设置和配置的更多信息，请参阅[PostgreSQL JDBC 驱动程序](https://jdbc.postgresql.org/documentation/head/index.html)文档。

**话题**

- [减少 DNS 缓存超时](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/AuroraPostgreSQL.BestPractices.html#AuroraPostgreSQL.BestPractices.FastFailover.Configuring.Timeouts)
- [为快速故障转移设置 Aurora PostgreSQL 连接字符串](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/AuroraPostgreSQL.BestPractices.html#AuroraPostgreSQL.BestPractices.FastFailover.Configuring.ConnectionString)
- [获取主机字符串的其他选项](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/AuroraPostgreSQL.BestPractices.html#AuroraPostgreSQL.BestPractices.FastFailover.Configuring.HostString)

#### 减少 DNS 缓存超时

当您的应用程序在故障转移后尝试建立连接时，新的 Aurora PostgreSQL 写入器将是以前的读取器，可以在 DNS 更新完全传播之前使用 Aurora **只读终端节点找到它**。将 java DNS TTL 设置为较低的值有助于在后续连接尝试时在读取器节点之间循环。

```
// Sets internal TTL to match the Aurora RO Endpoint TTL
java.security.Security.setProperty("networkaddress.cache.ttl" , "1");
// If the lookup fails, default to something like small to retry
java.security.Security.setProperty("networkaddress.cache.negative.ttl" , "3");
```

#### 为快速故障转移设置 Aurora PostgreSQL 连接字符串

要使用 Aurora PostgreSQL 快速故障转移，您的应用程序的连接字符串应该有一个主机列表（在以下示例中以粗体突出显示），而不仅仅是一个主机。以下是可用于连接到 Aurora PostgreSQL 集群的示例连接字符串：

```
jdbc:postgresql://myauroracluster.cluster-c9bfei4hjlrd.us-east-1-beta.rds.amazonaws.com:5432,
myauroracluster.cluster-ro-c9bfei4hjlrd.us-east-1-beta.rds.amazonaws.com:5432
/postgres?user=<primaryuser>&password=<primarypw>&loginTimeout=2
&connectTimeout=2&cancelSignalTimeout=2&socketTimeout=60
&tcpKeepAlive=true&targetServerType=primary
```

有关 PostgreSQL JDBC 驱动程序参数的更多信息，请参阅[连接到数据库](https://jdbc.postgresql.org/documentation/head/connect.html).

为获得最佳可用性并避免对 RDS API 的依赖，连接的最佳选择是维护一个文件，其中包含您的应用程序在建立与数据库的连接时从中读取的主机字符串。此主机字符串将具有可用于集群的所有 Aurora 端点。有关 Aurora 终端节点的更多信息，请参阅[Amazon Aurora 连接管理](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/Aurora.Overview.Endpoints.html)。例如，您可以将端点存储在本地文件中，如下所示：

```
myauroracluster.cluster-c9bfei4hjlrd.us-east-1-beta.rds.amazonaws.com:5432,
myauroracluster.cluster-ro-c9bfei4hjlrd.us-east-1-beta.rds.amazonaws.com:5432
```

您的应用程序将从该文件中读取以填充 JDBC 连接字符串的主机部分。重命名数据库集群会导致这些端点发生变化；确保您的应用程序在该事件发生时处理它。

另一种选择是使用数据库实例节点列表：

```
my-node1.cksc6xlmwcyw.us-east-1-beta.rds.amazonaws.com:5432,
my-node2.cksc6xlmwcyw.us-east-1-beta.rds.amazonaws.com:5432,
my-node3.cksc6xlmwcyw.us-east-1-beta.rds.amazonaws.com:5432,
my-node4.cksc6xlmwcyw.us-east-1-beta.rds.amazonaws.com:5432
```

这种方法的好处是 PostgreSQL JDBC 连接驱动程序将遍历此列表中的所有节点以查找有效连接，而在使用 Aurora 端点时，每次连接尝试只会尝试两个节点。使用数据库实例节点的缺点是，如果您在集群中添加或删除节点并且实例端点列表变得陈旧，则连接驱动程序可能永远找不到要连接的正确主机。

积极设置以下参数，以帮助确保您的应用程序不会等待太长时间才能连接到任何一台主机。

- `targetServerType`– 使用此参数来控制驱动程序是连接到写入节点还是读取节点。为确保您的应用程序仅重新连接到写入节点，请将 `targetServerType`值设置为`primary`.

  参数的值`targetServerType`包括 `primary`、`secondary`、`any`和 `preferSecondary`。该`preferSecondary`值首先尝试与读取器建立连接，但如果无法建立读取器连接，则连接到写入器。

- `loginTimeout`– 控制您的应用程序在 建立套接字连接*后等待登录数据库的时间。*

- `connectTimeout`– 控制套接字等待建立与数据库的连接的时间。

您可以修改其他应用程序参数以加快连接过程，具体取决于您希望应用程序的激进程度。

- `cancelSignalTimeout`– 在某些应用程序中，您可能希望在已超时的查询上发送“尽力而为”取消信号。如果此取消信号在您的故障转移路径中，您应该考虑积极设置它以避免将此信号发送到死主机。
- `socketTimeout`– 此参数控制套接字等待读取操作的时间。此参数可用作全局“查询超时”，以确保没有查询等待超过此值。一个好的做法是让一个连接处理程序运行短期查询并将此值设置得较低，并为长时间运行的查询设置另一个连接处理程序，并将此值设置得更高。然后，如果服务器出现故障，您可以依靠 TCP keepalive 参数来停止长时间运行的查询。
- `tcpKeepAlive`– 启用此参数以确保遵守您设置的 TCP keepalive 参数。
- `loadBalanceHosts`– 当设置为 时`true`，此参数使应用程序连接到从候选主机列表中选择的随机主机。

#### 获取主机字符串的其他选项

您可以从多个来源获取主机字符串，包括 `aurora_replica_status`函数和使用 Amazon RDS API。

您的应用程序可以连接到数据库集群中的任何数据库实例并查询该`aurora_replica_status`函数以确定集群的写入者是谁，或者查找集群中的任何其他读取器节点。您可以使用此功能来减少查找要连接的主机所需的时间，但在某些情况下，该`aurora_replica_status`功能可能会在某些网络故障情况下显示过时或不完整的信息。

确保您的应用程序可以找到要连接的节点的一种好方法是尝试连接到**集群写入器****端点**，然后再 连接到**集群读取器****端点**，直到您可以建立可读连接。除非您重命名数据库集群，否则这些端点不会更改，因此通常可以保留为应用程序的静态成员或存储在应用程序从中读取的资源文件中。

使用这些端点之一建立连接后，您可以调用该`aurora_replica_status`函数以获取有关集群其余部分的信息。例如，以下命令使用`aurora_replica_status`函数检索信息。

```
postgres=> SELECT server_id, session_id, highest_lsn_rcvd, cur_replay_latency_in_usec, now(), last_update_timestamp
FROM aurora_replica_status();

server_id | session_id | highest_lsn_rcvd | cur_replay_latency_in_usec | now | last_update_timestamp
-----------+--------------------------------------+------------------+----------------------------+-------------------------------+------------------------
mynode-1 | 3e3c5044-02e2-11e7-b70d-95172646d6ca | 594221001 | 201421 | 2017-03-07 19:50:24.695322+00 | 2017-03-07 19:50:23+00
mynode-2 | 1efdd188-02e4-11e7-becd-f12d7c88a28a | 594221001 | 201350 | 2017-03-07 19:50:24.695322+00 | 2017-03-07 19:50:23+00
mynode-3 | MASTER_SESSION_ID | | | 2017-03-07 19:50:24.695322+00 | 2017-03-07 19:50:23+00
(3 rows)
```

因此，例如，连接字符串的 hosts 部分可以同时以 writer 和 reader 集群端点开始：

```
myauroracluster.cluster-c9bfei4hjlrd.us-east-1-beta.rds.amazonaws.com:5432,
myauroracluster.cluster-ro-c9bfei4hjlrd.us-east-1-beta.rds.amazonaws.com:5432
```

在这种情况下，您的应用程序将尝试建立与任何节点类型（主要或辅助节点）的连接。当您的应用程序连接时，一个好的做法是首先通过查询命令的结果来检查节点的读/写状态`SHOW transaction_read_only`。

如果查询的返回值为`OFF`，那么您已经成功连接到主节点。如果返回值为`ON`，并且您的应用程序需要读/写连接，则可以调用该 `aurora_replica_status`函数来确定 `server_id`具有`session_id='MASTER_SESSION_ID'`. 此函数为您提供主节点的名称。您可以将它与下面描述的“endpointPostfix”结合使用。

需要注意的一件事是当您连接到具有陈旧数据的副本时。发生这种情况时，该`aurora_replica_status`函数可能会显示过时的信息。可以在应用程序级别设置过时阈值，并通过查看服务器时间和 `last_update_timestamp`. `aurora_replica_status`通常，您的应用程序应避免由于函数返回的信息冲突而在两个主机之间切换 。您的应用程序应该首先尝试所有已知的主机，而不是盲目地遵循 `aurora_replica_status`函数返回的数据。

##### 使用 DescribeDBClusters API 列出实例的 Java 示例

[您可以使用适用于 Java 的 AWS 开发工具包以](https://aws.amazon.com/sdk-for-java/)编程方式查找实例列表 ，特别是[DescribeDBClusters](https://docs.aws.amazon.com/AmazonRDS/latest/APIReference/API_DescribeDBClusters.html) API。下面是一个小例子，说明如何在 java 8 中执行此操作：

```
AmazonRDS client = AmazonRDSClientBuilder.defaultClient();
DescribeDBClustersRequest request = new DescribeDBClustersRequest()
   .withDBClusterIdentifier(clusterName);
DescribeDBClustersResult result = 
rdsClient.describeDBClusters(request);

DBCluster singleClusterResult = result.getDBClusters().get(0);

String pgJDBCEndpointStr = 
singleClusterResult.getDBClusterMembers().stream()
   .sorted(Comparator.comparing(DBClusterMember::getIsClusterWriter)
   .reversed()) // This puts the writer at the front of the list
   .map(m -> m.getDBInstanceIdentifier() + endpointPostfix + ":" + singleClusterResult.getPort()))
   .collect(Collectors.joining(","));
```

pgJDBCEndpointStr 将包含一个格式化的端点列表。例如：

```
my-node1.cksc6xlmwcyw.us-east-1-beta.rds.amazonaws.com:5432,
my-node2.cksc6xlmwcyw.us-east-1-beta.rds.amazonaws.com:5432
```

该变量`endpointPostfix`可以是您的应用程序设置的常量，也可以通过查询 `DescribeDBInstances`API 以获取集群中的单个实例。此值在区域内和单个客户中保持不变，因此它会保存 API 调用以简单地将这个常量保存在应用程序从中读取的资源文件中。在上面的示例中，它将被设置为：

```
.cksc6xlmwcyw.us-east-1-beta.rds.amazonaws.com
```

出于可用性目的，如果 API 没有响应或响应时间过长，一个好的做法是默认使用数据库集群的 Aurora 终端节点。端点保证在更新 DNS 记录所需的时间内是最新的。这通常少于 30 秒。您可以将其存储在应用程序使用的资源文件中。

### 测试故障转移

在所有情况下，您都必须拥有一个包含两个或更多数据库实例的数据库集群。

在服务器端，某些 API 可能会导致中断，可用于测试应用程序的响应方式：

- [FailoverDBCluster](https://docs.aws.amazon.com/AmazonRDS/latest/APIReference/API_FailoverDBCluster.html) - 将尝试将数据库集群中的新数据库实例提升为写入器。

  以下代码示例显示了如何使用它 `failoverDBCluster`来导致中断。有关设置 Amazon RDS 客户端的更多详细信息，请参阅[使用适用于 Java 的 AWS 开发工具包](https://docs.aws.amazon.com/sdk-for-java/v1/developer-guide/basics.html)。

  ```
  public void causeFailover() {
      
      final AmazonRDS rdsClient = AmazonRDSClientBuilder.defaultClient();
     
      FailoverDBClusterRequest request = new FailoverDBClusterRequest();
      request.setDBClusterIdentifier("cluster-identifier");
  
      rdsClient.failoverDBCluster(request);
  }
  ```

- [RebootDBInstance](https://docs.aws.amazon.com/AmazonRDS/latest/APIReference/API_RebootDBInstance.html) – 此 API 不保证故障转移。不过，它将关闭写入器上的数据库，并可用于测试您的应用程序如何响应连接断开（请注意， **ForceFailover**参数不适用于 Aurora 引擎，而应使用 `FailoverDBCluster`API）。

- [ModifyDBCluster](https://docs.aws.amazon.com/AmazonRDS/latest/APIReference/API_ModifyDBCluster.html) – 当集群中的节点开始侦听新端口时，修改**端口** 将导致中断。通常，您的应用程序可以通过确保只有您的应用程序控制端口更改并可以适当地更新它所依赖的端点，通过让某人在 API 级别进行修改时手动更新端口，或通过查询 RDS API 来响应此故障在您的应用程序中确定端口是否已更改。

- [ModifyDBInstance](https://docs.aws.amazon.com/AmazonRDS/latest/APIReference/API_ModifyDBInstance.html) – 修改**DBInstanceClass**将导致中断。

- [DeleteDBInstance](https://docs.aws.amazon.com/AmazonRDS/latest/APIReference/API_DeleteDBInstance.html) – 删除主数据库/写入程序将导致一个新的数据库实例被提升为数据库集群中的写入程序。

从应用程序/客户端，如果使用 Linux，您可以测试应用程序如何响应基于端口、主机的突然丢包，或者使用 iptables 是否没有发送或接收 tcp keepalive 数据包。

### 快速故障转移 Java 示例

以下代码示例显示了应用程序如何设置 Aurora PostgreSQL 驱动程序管理器。`getConnection()`应用程序会在需要连接时调用。调用此函数可能无法找到有效主机，例如未找到写入器但`targetServerType`参数设置为 时 `primary`。调用应用程序应该简单地重试调用该函数。这可以很容易地包装到连接池中，以避免将重试行为推送到应用程序上。大多数连接池允许您指定 JDBC 连接字符串，因此您的应用程序可以调用 `getJdbcConnectionString()`并将其传递给连接池以在 Aurora PostgreSQL 上使用更快的故障转移。

```
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;
import java.sql.Statement;
import java.util.ArrayList;
import java.util.List;
import java.util.stream.Collectors;
import java.util.stream.IntStream;

import org.joda.time.Duration;

public class FastFailoverDriverManager {
   private static Duration LOGIN_TIMEOUT = Duration.standardSeconds(2);
   private static Duration CONNECT_TIMEOUT = Duration.standardSeconds(2);
   private static Duration CANCEL_SIGNAL_TIMEOUT = Duration.standardSeconds(1);
   private static Duration DEFAULT_SOCKET_TIMEOUT = Duration.standardSeconds(5);

   public FastFailoverDriverManager() {
       try {
            Class.forName("org.postgresql.Driver");
       } catch (ClassNotFoundException e) {
            e.printStackTrace();
       }

       /*
        * RO endpoint has a TTL of 1s, we should honor that here. Setting this aggressively makes sure that when
        * the PG JDBC driver creates a new connection, it will resolve a new different RO endpoint on subsequent attempts
        * (assuming there is > 1 read node in your cluster)
        */
       java.security.Security.setProperty("networkaddress.cache.ttl" , "1");
       // If the lookup fails, default to something like small to retry
       java.security.Security.setProperty("networkaddress.cache.negative.ttl" , "3");
   }

   public Connection getConnection(String targetServerType) throws SQLException {
       return getConnection(targetServerType, DEFAULT_SOCKET_TIMEOUT);
   }

   public Connection getConnection(String targetServerType, Duration queryTimeout) throws SQLException {
        Connection conn = DriverManager.getConnection(getJdbcConnectionString(targetServerType, queryTimeout));

        /*
         * A good practice is to set socket and statement timeout to be the same thing since both 
         * the client AND server will stop the query at the same time, leaving no running queries 
         * on the backend
         */
        Statement st = conn.createStatement();
        st.execute("set statement_timeout to " + queryTimeout.getMillis());
        st.close();

       return conn;
   }

   private static String urlFormat = "jdbc:postgresql://%s"
           + "/postgres"
           + "?user=%s"
           + "&password=%s"
           + "&loginTimeout=%d"
           + "&connectTimeout=%d"
           + "&cancelSignalTimeout=%d"
           + "&socketTimeout=%d"
           + "&targetServerType=%s"
           + "&tcpKeepAlive=true"
           + "&ssl=true"
           + "&loadBalanceHosts=true";
   public String getJdbcConnectionString(String targetServerType, Duration queryTimeout) {
       return String.format(urlFormat, 
                getFormattedEndpointList(getLocalEndpointList()),
                CredentialManager.getUsername(),
                CredentialManager.getPassword(),
                LOGIN_TIMEOUT.getStandardSeconds(),
                CONNECT_TIMEOUT.getStandardSeconds(),
                CANCEL_SIGNAL_TIMEOUT.getStandardSeconds(),
                queryTimeout.getStandardSeconds(),
                targetServerType
       );
   }

   private List<String> getLocalEndpointList() {
       /*
         * As mentioned in the best practices doc, a good idea is to read a local resource file and parse the cluster endpoints. 
         * For illustration purposes, the endpoint list is hardcoded here
         */
        List<String> newEndpointList = new ArrayList<>();
        newEndpointList.add("myauroracluster.cluster-c9bfei4hjlrd.us-east-1-beta.rds.amazonaws.com:5432");
        newEndpointList.add("myauroracluster.cluster-ro-c9bfei4hjlrd.us-east-1-beta.rds.amazonaws.com:5432");

       return newEndpointList;
   }

   private static String getFormattedEndpointList(List<String> endpoints) {
       return IntStream.range(0, endpoints.size())
               .mapToObj(i -> endpoints.get(i).toString())
               .collect(Collectors.joining(","));
   }
}        
```

## 解决存储问题

如果排序或索引创建操作所需的内存量超过可用内存量，Aurora PostgreSQL 会将多余的数据写入存储。当它写入数据时，它使用用于存储错误和消息日志的相同存储空间。如果您的排序或索引创建功能超过了可用内存，您可能会出现本地存储短缺。如果您遇到 Aurora PostgreSQL 存储空间不足的问题，您可以重新配置数据排序以使用更多内存，或缩短 PostgreSQL 日志文件的数据保留期。有关更改日志保留期的更多信息，请参阅[PostgreSQL 数据库日志文件](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/USER_LogAccess.Concepts.PostgreSQL.html)。

如果您的 Aurora 集群大于 40 TB，请不要使用 db.t2、db.t3 或 db.t4g 实例类。