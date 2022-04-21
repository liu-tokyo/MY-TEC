# 使用 Amazon Aurora PostgreSQL 进行故障转移

- https://aws.amazon.com/cn/blogs/database/failover-with-amazon-aurora-postgresql/

复制、故障转移、弹性、灾难恢复和备份*——*在传统或非基于云的架构中实现任何或所有这些可能具有挑战性。此外，它们有时需要大量的重新设计工作。由于所涉及的高实施和基础设施成本，一些企业被迫对其应用程序进行分层，以便只有最关键的应用程序才能得到很好的保护。

[您可以通过迁移到Amazon Aurora for PostgreSQL](https://aws.amazon.com/rds/aurora/details/postgresql-details/)来帮助缓解这些问题。AWS 提供了广泛的关系数据库引擎选择，包括（但不限于）Oracle、MySQL、PostgreSQL 和 Aurora。对于 PostgreSQL，AWS 支持多种风格，包括[Amazon EC2](https://aws.amazon.com/ec2/)实例上的 PostgreSQL、[Amazon RDS for PostgreSQL](https://aws.amazon.com/rds/postgresql/)和兼容 PostgreSQL 的 Amazon Aurora。在选择正确 PostgreSQL 版本的众多晴雨表中，以下是几个重要的指标：

- 高可用性 (HA)
- 表现
- 易于管理

让我们深入了解 Amazon Aurora PostgreSQL 如何开箱即用地满足这些标准。

**高可用性：** HA 内置于 Aurora PostgreSQL 的架构中，在三个可用区中维护了六个数据副本。这是每个可用区中的两个副本，即使整个可用区出现故障，也可以通过轻微中断来提高可用性。此外，数据库会持续备份到 Amazon S3，因此您可以利用 S3 (99.999999999) 的高持久性进行备份。Aurora PostgreSQL 还支持时间点恢复。

**性能：**与 Amazon EC2 上的 PostgreSQL 相比，Amazon Aurora PostgreSQL 的性能最高可提高三倍。有关基准测试的更多详细信息，请下载与[Amazon Aurora PostgreSQL 兼容的版本基准测试指南](https://d1.awsstatic.com/product-marketing/Aurora/RDS_Aurora_PostgreSQL_Performance_Assessment_Benchmarking_V1-0.pdf)。此外，使用[Performance Insights](https://aws.amazon.com/rds/performance-insights/)可以更轻松地进行数据库分析和故障排除。

**可管理性：** Amazon Aurora PostgreSQL 通过处理常规数据库任务（例如预置、修补、备份、恢复、故障检测和修复）来简化管理。Aurora 存储可自动扩展、增长和重新平衡整个队列的 I/O，以提供一致的性能。

在这篇文章中，我概述了 Amazon Aurora 集群和终端节点，以及它们如何帮助您在对配置进行最小更改的情况下实现读取的故障转移和负载平衡。我将通过一个示例向您展示故障转移如何在 Aurora 集群中工作，并描述如何使故障转移透明化。这篇文章主要讨论 Amazon Aurora PostgreSQL，但您也可以将大部分概念应用到 Amazon Aurora MySQL。

## Aurora 集群简介

与 Amazon EC2 上的 PostgreSQL 或 Amazon RDS for PostgreSQL 不同，当您创建 Aurora PostgreSQL 数据库实例时，您实际上是在创建数据库*集群*。在 Aurora PostgreSQL 中，数据库集群是一个读/写实例和最多 15 个读实例的集合，以及跨多个可用区的数据存储（集群卷）。每个可用区维护两个数据库集群数据副本。

Amazon Aurora 副本与源实例共享相同的底层存储，从而降低成本并避免将数据复制到副本节点的需要。Amazon RDS for PostgreSQL 只读副本是一个物理副本。每当源数据库实例发生更改时，它使用异步复制方法更新只读副本。此外，使用 RDS for PostgreSQL，每个数据库源实例最多只能有 5 个只读副本，而 Aurora 集群支持一个主（主或读/写）实例以及最多 15 个只读副本。

下图说明了具有一个主副本和三个只读副本的 Aurora PostgreSQL 架构：![包含一个主副本和三个只读副本的 Aurora PostgreSQL 架构的屏幕截图](https://d2908q01vomqb2.cloudfront.net/887309d048beef83ad3eabf2a79a64a389ab1c9f/2018/06/08/Aurora-Arch.jpg)

有关 Aurora 架构的更多信息，请参阅 AWS 数据库博客文章[介绍 Aurora 存储引擎](https://aws.amazon.com/blogs/database/introducing-the-aurora-storage-engine/)。

## 端点

与 Aurora PostgreSQL 实例的连接是使用终端节点建立的。端点是包含主机地址和端口的 URL，以冒号分隔。当您创建 Aurora PostgreSQL 实例时，AWS 在集群级别和实例级别创建终端节点。在集群级别，创建了两个端点，一个用于读/写操作（称为*集群端点*），另一个用于只读操作（称为*读取器端点*）。故障转移是通过集群端点实现的，而跨多个只读副本的只读连接的负载平衡是通过读取器端点实现的。此外，如果集群没有只读副本，则读取器端点提供到主实例的连接。

在实例级别，每个实例创建一个端点。使用实例端点，您可以像传统连接一样直接连接到实例。如果没有充分的理由，不鼓励仅使用实例端点（没有集群端点）。您可以将集群端点与实例端点一起用于读取查询的手动负载平衡。

下图说明了端点的工作方式。![img](https://d2908q01vomqb2.cloudfront.net/887309d048beef83ad3eabf2a79a64a389ab1c9f/2018/06/08/End-points.jpg)

**注意：**如果重命名数据库集群，集群和读取器端点会发生变化。

在 Amazon RDS 控制台中，您可以通过在导航窗格中选择**集群来找到集群和读取器终端节点：**

![pgcluster 的集群和读取器端点详细信息的屏幕截图](https://d2908q01vomqb2.cloudfront.net/887309d048beef83ad3eabf2a79a64a389ab1c9f/2018/06/08/pgcluster.jpg)

## 故障转移如何工作？

如果数据库集群中的主实例发生故障，Aurora 会按以下顺序自动进行故障转移：

1. 如果 Aurora 只读副本可用，请将现有只读副本提升到新的主实例。
2. 如果没有可用的只读副本，则创建一个新的主实例。

如果有多个 Aurora 只读副本，则升级标准基于为只读副本定义的优先级。优先级编号可以从 0 到 15 不等，并且可以随时修改。Amazon Aurora PostgreSQL 将具有最高优先级的 Aurora 副本提升到新的主实例。对于具有相同优先级的只读副本，Aurora PostgreSQL 会提升大小最大的副本或以任意方式提升副本。

如果应用程序使用集群端点连接并实施连接重试逻辑，则它们的服务中断最少。在故障转移期间，AWS 修改集群终端节点以指向新创建/升级的数据库实例。架构良好的应用程序会自动重新连接。故障转移期间的停机时间取决于是否存在健康的只读副本。如果没有配置只读副本，或者现有只读副本不健康，那么您可能会注意到创建新实例的停机时间增加了。

### Aurora 集群故障转移示例

让我们来看一个 Amazon Aurora 集群的故障转移示例。下表包含具有一个主节点和两个只读副本的集群的详细信息。

| 集群名称   | `pgcluster`                                                  |
| ---------- | ------------------------------------------------------------ |
| 主实例     | `myinstance-us-east-2a`                                      |
| 只读副本   | `myinstance-us-east-2b (Priority 0)`                         |
| 只读副本   | `myinstance-us-east-2c (Priority 1)`                         |
| 集群端点   | `pgcluster.cluster-xxxxxxxxxx.us-east-2.rds.amazonaws.com`   |
| 阅读器端点 | `pgcluster.cluster-ro-xxxxxxxxxx.us-east-2.rds.amazonaws.com` |

此图表示名为 的 Aurora PostgreSQL 集群`pgcluster`。请注意，集群端点`pgcluster.cluster-xxxxxxxxxx.us-east-2.rds.amazonaws.com`当前指向实例`myinstance-us-east-2a`。![Aurora PostgreSQL 集群的屏幕截图](https://d2908q01vomqb2.cloudfront.net/887309d048beef83ad3eabf2a79a64a389ab1c9f/2018/06/08/Instance2.jpg)

以下代码说明了`pgdb`使用集群端点连接到端口 5432 上的数据库。作为该插图的一部分，该示例创建了一个名为的表`failover`并将一行插入到该表中。

```sql
psql  -h pgcluster.cluster-xxxxxxxxxx.us-east-2.rds.amazonaws.com -d pgdb -p 5432 -U pgadmin
--Listing the name of master instance 
pgdb=> select server_id from aurora_replica_status() where session_id='MASTER_SESSION_ID';
       server_id
-----------------------
 myinstance-us-east-2a
(1 row)
pgdb=> CREATE TABLE FAILOVER(FAILOVERNAME  VARCHAR(20), FAILOVERTYPE INTEGER);
CREATE TABLE                          
pgdb=>
pgdb=> INSERT INTO FAILOVER VALUES ('TEST-1',1 );
INSERT 0 1
pgdb=>
pgdb=> SELECT * FROM FAILOVER;
  failovername   | failovertype
-----------------+--------------
 TEST-1          |            1
(1 row)
```

以下代码显示了使用读取器端点连接到只读副本实例以及失败的事务。通过读取器端点从只读副本访问该`failover`表以验证同步。

```sql
psql  -h pgcluster.cluster-ro-xxxxxxxxxx.us-east-2.rds.amazonaws.com -d pgdb -p 5432 -U pgadmin
--Listing the name of master instance 
pgdb=> select server_id from aurora_replica_status() where session_id='MASTER_SESSION_ID';
       server_id
-----------------------
 myinstance-us-east-2a
(1 row)

pgdb=> SELECT * FROM FAILOVER;
  failovername   | failovertype
-----------------+--------------
 TEST-1          |            1
(1 row)
pgdb=> INSERT INTO FAILOVER VALUES (‘TEST-2’, 1 );
ERROR:  cannot execute INSERT in a read-only transaction
```

正如预期的那样，由于只读副本不支持可写事务，因此发生错误。

### 手动故障转移

以下屏幕截图显示了`myinstance-us-east-2a`Amazon RDS 控制台中的当前主节点：![屏幕截图显示了 Amazon RDS 控制台中的当前主“myinstance-us-east-2a”](https://d2908q01vomqb2.cloudfront.net/887309d048beef83ad3eabf2a79a64a389ab1c9f/2018/06/08/DB-Instance-riter.jpg)

要进行手动故障转移，请在控制台上选择主实例名称，然后在**实例操作**菜单上选择**故障转移。**![img](https://d2908q01vomqb2.cloudfront.net/887309d048beef83ad3eabf2a79a64a389ab1c9f/2018/06/08/Failover.jpg)

然后在**故障转移**窗口中确认：![img](https://d2908q01vomqb2.cloudfront.net/887309d048beef83ad3eabf2a79a64a389ab1c9f/2018/06/08/Failover-DB-Cluster.jpg)

手动故障转移完成后，将`myinstance-us-east-2b`实例提升为新的主实例，并将`myinstance-us-east-2a`实例角色更改为只读副本。此外，集群端点现在指向`myinstance-us-east-2b`，而`myinstance-us-east-2a`可通过读取器端点获得。

以下屏幕截图显示只读副本成为 RDS 控制台中的新主副本：![img](https://d2908q01vomqb2.cloudfront.net/887309d048beef83ad3eabf2a79a64a389ab1c9f/2018/06/08/NEw-Writer.jpg)

### 重新连接实例

现在您可以使用相同的集群端点重新连接：

```sql
psql  -h pgcluster.cluster-xxxxxxxxxx.us-east-2.rds.amazonaws.com -d pgdb -p 5432 -U pgadmin
--Listing the name of master instance after failover
pgdb=> select server_id from aurora_replica_status() where session_id='MASTER_SESSION_ID';
       server_id
-----------------------
 myinstance-us-east-2b
(1 row)

pgdb=> INSERT INTO FAILOVER VALUES (‘TEST-2’, 1 );
INSERT 0 1
pgdb=> SELECT * FROM FAILOVER;
    failovername    | failovertype
--------------------+--------------
 TEST-1             |            1
 TEST-2             |            1
(2 rows)
```

如前面的代码所示，连接成功，无需对端点进行任何更改。您可以通过添加应用程序连接重试逻辑使故障转移透明。

`PGConn`除了集群端点，您还可以通过在应用程序级别配置积极的 TCP keepalive 设置和 JDBC 连接设置或类来更快地响应故障转移。有关详细信息，请参阅[Amazon Aurora PostgreSQL 的最佳实践](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/AuroraPostgreSQL.BestPractices.html)。

## 结论

处理无人值守的恢复对于任何组织实现正常运行时间服务水平协议 (SLA) 都非常重要。Aurora PostgreSQL 凭借其创新的故障转移架构，不仅支持更高的可用性和可靠性，还通过读取可扩展性提供增强的性能。

您的应用程序可以通过多种方式响应故障转移，但使用集群和读取器端点，您可以通过最少的配置更改来实现读取的故障转移和负载平衡。此外，通过使用集群端点以及遵循 TCP keepalive 和 JDBC 最佳实践，您可以实现快速故障转移和高可用性。不鼓励使用实例端点作为主要端点。但是，它可用于支持读取的手动负载平衡。此外，您可以通过向应用程序添加自动重试功能来使故障转移透明化。