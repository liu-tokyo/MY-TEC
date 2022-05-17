# Amazon Aurora 数据库集群

Amazon Aurora *数据库集群*包含一个或多个数据库实例以及一个管理这些数据库实例的数据的集群卷。Aurora *集群卷* 是一个跨多个可用区的虚拟数据库存储卷，每个可用区具有一个数据库集群数据副本。Aurora 数据库集群由两类数据库实例组成：



- **主数据库实例** – 支持读取和写入操作，**并执行针对集群卷的所有数据修改**。每个 Aurora 数据库集群均有一个主数据库实例。
- **Aurora 副本** – 连接到同一存储卷作为主数据库实例并仅支持读取操作。除主数据库实例之外，每个 Aurora 数据库集群最多可拥有 15 个 Aurora 副本。请将 Aurora 副本放在单独的可用区以保持高可用性。如果主数据库实例变得不可用，Aurora 自动故障转移到 Aurora 副本。您可以为 Aurora 副本指定故障转移优先级。Aurora 副本还可以从主数据库实例分载读取工作负载。
- 对于 Aurora 多主集群，所有数据库实例都具有读写功能。在此情况下，主实例和 Aurora 副本之间的区别不适用。为了讨论集群可在其中使用单主或多主复制的复制拓扑，我们将调用这些*读取器* 和*写入器* 数据库实例。



下图说明了集群卷与 Aurora 数据库集群中的主数据库实例和 Aurora 副本之间的关系。



![img](https://pic4.zhimg.com/80/v2-df0be2c4a6667782fae2849ffdea8893_720w.jpg)



**注意**
上述信息适用于所有使用单主复制的 Aurora 集群。它们包括预配置集群、并行查询集群、全局数据库集群、无服务器集群，以及所有 MySQL 5.7 兼容集群和 PostgreSQL 兼容集群。
使用多主复制的 Aurora 集群具有不同的读写和只读数据库实例组合。多主集群中的所有数据库实例都可以执行写操作。没有单个执行所有写操作的数据库实例，也没有任何只读数据库实例。因此，术语*主实例* 和 *Aurora 副本* 不适用于多主集群。在讨论可能使用多主复制的集群时，我们应用*写入器* 数据库实例和*读取器* 数据库实例。

Aurora 集群说明了计算容量和存储的分离。例如，仅具有单个数据库实例的 Aurora 配置仍是集群，因为基础存储卷涉及跨多个可用区 (AZ) 分布的多个存储节点。

## **Amazon Aurora 连接管理**

Amazon Aurora 通常涉及数据库实例集群而不是单个实例。每个连接均由特定的数据库实例处理。在连接到 Aurora 集群时，您指定的主机名和端口将指向名为*终端节点* 的中间处理程序。Aurora 使用终端节点机制来提取这些连接。因此，当某些数据库实例不可用时，您不必对所有主机名进行硬编码或编写自己的逻辑来进行负载均衡和重新路由连接。

对于某些 Aurora 任务，不同的实例或实例组执行不同的角色。例如，主实例处理所有数据定义语言 (DDL) 和数据操作语言 (DML) 语句。最多 15 个 Aurora 副本处理只读查询流量。

通过使用终端节点，您可以根据使用案例将每个连接映射到相应的实例或实例组。例如，要执行 DDL 语句，您可以连接到作为主实例的任一实例。要执行查询，您可以连接到读取器终端节点，并通过 Aurora 自动在所有 Aurora 副本之间执行负载均衡。对于具有不同容量或配置的数据库实例的集群，您可以连接到与不同的数据库实例子集关联的自定义终端节点。对于诊断或优化，您可以连接到特定实例终端节点以检查有关特定数据库实例的详细信息。

## **Aurora 终端节点的类型**

终端节点表示为包含主机地址和端口的 Aurora 特定的 URL。可从 Aurora 数据库集群使用以下类型的终端节点。

**集群终端节点**Aurora 数据库集群的*集群终端节点*，它连接到该数据库集群的当前主数据库实例。此终端节点是唯一可以执行写操作（如 DDL 语句）的终端节点。因此，集群终端节点是您在首次设置集群时或集群仅包含单个数据库实例时连接到的终端节点。
每个 Aurora 数据库集群均有一个集群终端节点和一个主数据库实例。
对数据库集群上的所有写入操作使用集群终端节点，这些操作包括插入、更新、删除和 DDL 更改。您还可以对读取操作（如查询）使用集群终端节点。
集群终端节点为数据库集群的读取/写入连接提供故障转移支持。如果数据库集群的当前主数据库实例失败，Aurora 将自动故障转移到新的主数据库实例。在故障转移期间，数据库集群将继续为从新的主数据库实例到集群终端节点的请求提供服务，对服务造成的中断最少。
以下示例介绍 Aurora MySQL 数据库集群中的集群终端节点。
`mydbcluster.cluster-123456789012.us-east-1.rds.amazonaws.com:3306`**读取器终端节点**Aurora 数据库集群的*读取器终端节点* 连接到该数据库集群的可用 Aurora 副本之一。每个 Aurora 数据库集群均有一个读取器终端节点。如果有多个 Aurora 副本，则读取器终端节点会将每个连接请求定向到 Aurora 副本之一。
读取器终端节点为数据库集群的只读连接提供负载均衡支持。对读取操作 (如查询) 使用读取器终端节点。不能对写入操作使用读取器终端节点。
数据库集群在可用 Aurora 副本之间分配对读取器终端节点的连接请求。如果数据库集群只包含主数据库实例，则读取器终端节点从主数据库实例为连接请求提供服务。如果为该数据库集群创建了一个或多个 Aurora 副本，则后续与读取器终端节点的连接将在各个副本之间进行负载均衡。
以下示例介绍 Aurora MySQL 数据库集群中的读取器终端节点。
[mydbcluster.cluster-ro-123456789012.us-east-1.rds.amazonaws.com:3306](https://link.zhihu.com/?target=http%3A//mydbcluster.cluster-ro-123456789012.us-east-1.rds.amazonaws.com%3A3306)

## **Amazon Aurora 存储和可靠性**

接下来，您可以了解 Aurora 存储子系统。Aurora 使用分布式的共享存储架构，这是影响 Aurora 集群的性能、可扩展性和可靠性的一个重要因素。

**主题**

- [Aurora 存储概述](https://link.zhihu.com/?target=https%3A//docs.amazonaws.cn/AmazonRDS/latest/AuroraUserGuide/Aurora.Overview.StorageReliability.html%23Aurora.Overview.Storage)
- [集群卷包含的内容](https://link.zhihu.com/?target=https%3A//docs.amazonaws.cn/AmazonRDS/latest/AuroraUserGuide/Aurora.Overview.StorageReliability.html%23aurora-storage-contents)
- [Aurora 存储的增长方式](https://link.zhihu.com/?target=https%3A//docs.amazonaws.cn/AmazonRDS/latest/AuroraUserGuide/Aurora.Overview.StorageReliability.html%23aurora-storage-growth)
- [Aurora 数据存储的计费方式](https://link.zhihu.com/?target=https%3A//docs.amazonaws.cn/AmazonRDS/latest/AuroraUserGuide/Aurora.Overview.StorageReliability.html%23aurora-storage-data-billing)
- [Amazon Aurora 可靠性](https://link.zhihu.com/?target=https%3A//docs.amazonaws.cn/AmazonRDS/latest/AuroraUserGuide/Aurora.Overview.StorageReliability.html%23Aurora.Overview.Reliability)



## **Aurora 存储概述**

Aurora 数据存储在*集群卷* 中，该集群卷是使用固态驱动器 (SSD) 的单个虚拟卷。集群卷由跨一个 AWS 区域中的多个可用区的数据副本组成。由于数据会自动跨可用区复制，因此，数据具有高持久性，同时数据丢失的可能性很小。此复制还确保您的数据库在故障转移期间更可用。这样做是因为数据副本已存在于其他可用区中并且继续响应对数据库集群中数据库实例的数据请求。复制的数量与集群中的数据库实例数量无关。

## **集群卷包含的内容**

Aurora 集群卷包含所有用户数据、架构对象和内部元数据，例如系统表和二进制日志。例如，Aurora 在集群卷中存储一个 Aurora 集群的所有表、索引、二进制大对象 (BLOB)、存储过程等。

Aurora 共享存储架构使您的数据独立于集群中的数据库实例。例如，您可以快速添加数据库实例，因为 Aurora 不会创建表数据的新副本。相反，数据库实例连接到已包含所有数据的共享卷。您可以从集群中删除数据库实例，而无需从集群中删除任何基础数据。仅当您删除整个集群时，Aurora 才会删除数据。

## **Aurora 存储的增长方式**

Aurora 集群卷随着数据库中数据量的增加自动增大。Aurora 集群卷可增大到最大 64 TiB。表大小限制为集群卷的大小。即，Aurora 数据库集群中表的最大表大小为 64 TiB。

当您的主要目标是可靠性和高可用性时，此自动存储扩展与高性能和高度分布式存储子系统相结合，使 Aurora 成为重要企业数据的良好选择。有关平衡存储成本与这些其他优先级的方法，请参阅以下部分。

## **Aurora 数据存储的计费方式**

即使 Aurora 集群卷最大可增大到 64 TiB，您也只需为在 Aurora 集群卷中使用的空间付费。在删除 Aurora 数据时（例如，删除表或分区），整个分配的空间保持不变。在将来数据量增加时，可以自动重用可用空间。

**注意**
由于存储成本基于存储"高水位"（在任何时间点为 Aurora 集群分配的最大容量），因此，您可以避免采用创建大量临时信息的 ETL 做法以控制成本，或者避免在删除不需要的旧数据之前加载大量新数据的 ETL 做法。

如果从 Aurora 集群中删除数据导致分配大量未使用的空间，则重置高水位需要使用 `mysqldump` 之类的工具执行逻辑数据转储和还原以转储和还原到新集群。创建和还原快照*不会* 减少分配的存储，因为基础存储的物理布局在还原的快照中保持不变。

有关 Aurora 数据存储的定价信息，请参阅 [Amazon RDS for Aurora 定价](https://link.zhihu.com/?target=http%3A//www.amazonaws.cn/rds/aurora/pricing)。

## **Amazon Aurora 可靠性**

Aurora 的设计具有可靠、持久和容错的特点。您可通过执行一些操作（例如，添加 Aurora 副本并将这些副本置于不同的可用区中）来构建 Aurora 数据库集群以提高可用性，Aurora 还包括一些自动化功能，这些功能可使其成为一种可靠的数据库解决方案。

**主题**

- [存储自动修复](https://link.zhihu.com/?target=https%3A//docs.amazonaws.cn/AmazonRDS/latest/AuroraUserGuide/Aurora.Overview.StorageReliability.html%23Aurora.Overview.AutoRepair)
- [自动恢复缓存预热](https://link.zhihu.com/?target=https%3A//docs.amazonaws.cn/AmazonRDS/latest/AuroraUserGuide/Aurora.Overview.StorageReliability.html%23Aurora.Overview.CacheWarming)
- [崩溃恢复](https://link.zhihu.com/?target=https%3A//docs.amazonaws.cn/AmazonRDS/latest/AuroraUserGuide/Aurora.Overview.StorageReliability.html%23Aurora.Overview.CrashRecovery)



### **存储自动修复**

由于 Aurora 将多个数据副本保留在三个可用区中，因此大大降低了因磁盘故障导致数据丢失的可能性。Aurora 自动检测集群卷包含的磁盘卷中的故障。如果磁盘卷的某个区段发生故障，Aurora 会立即修复该区段。在 Aurora 修复磁盘区段时，它使用集群卷包含的其他卷中的数据以确保已修复区段中的数据最新。因此，Aurora 将避免数据丢失，并减少了执行时间点还原以从磁盘故障恢复的需求。

### **自动恢复缓存预热**

当数据库在关闭后启动或在发生故障后重新启动时，Aurora 将对缓冲池缓存进行“预热”。即，Aurora 会用内存页面缓存中存储的已知常用查询页面预加载缓冲池。这样，缓冲池便无需从正常的数据库使用“预热”，从而提高性能。

Aurora 页面缓存将通过与数据库不同的单独进程进行管理，这将允许页面缓存独立于数据库进行自动恢复。在出现极少发生的数据库故障时，页面缓存将保留在内存中，这将确保在数据库重新启动时，使用最新状态预热缓冲池。

### **崩溃恢复**

Aurora 设计为几乎立即从崩溃中恢复，并继续提供应用程序数据而没有二进制日志。Aurora 在并行线程上异步执行崩溃恢复，以便在发生崩溃后立即打开并使用数据库。

有关崩溃恢复的更多信息，请参阅[Aurora 数据库集群的容错能力](https://link.zhihu.com/?target=https%3A//docs.amazonaws.cn/AmazonRDS/latest/AuroraUserGuide/Aurora.Managing.Backups.html%23Aurora.Managing.FaultTolerance)。

下面是二进制日志记录与 Aurora MySQL 上的崩溃恢复的注意事项：



- 直接在 Aurora 上启用二进制日志记录会影响崩溃后的恢复时间，因为它强制数据库实例执行二进制日志恢复。

- 所用二进制日志记录的类型影响日志记录的大小和效率。对于相同数量的数据库活动，某些格式会比二进制日志中的其他格式记录更多信息。`binlog_format` 参数的以下设置会产生不同的日志数据量：

- - `ROW` – 最多的日志数据
  - `STATEMENT` – 最少的日志数据
  - `MIXED` – 中等数量的日志数据，通常提供最佳的数据完整性和性能组合


二进制日志数据量影响恢复时间。二进制日志中记录的数据越多，数据库实例在恢复过程中就必须处理更多数据，这会增加恢复时间。

- Aurora 不需要二进制日志来复制数据库集群中的数据或执行时间点恢复 (PITR)。
- 如果您不需要外部复制的二进制日志 (或外部二进制日志流)，我们建议您将 `binlog_format` 参数设置为 `OFF` 以禁用二进制日志记录。这样做可以减少恢复时间。



有关 Aurora 二进制日志记录和复制的更多信息，请参阅[使用 Amazon Aurora 进行复制](https://link.zhihu.com/?target=https%3A//docs.amazonaws.cn/AmazonRDS/latest/AuroraUserGuide/Aurora.Replication.html)。有关不同 MySQL 复制类型的含义的更多信息，请参阅 MySQL 文档中的[基于语句和基于行的复制的优点和缺点](https://link.zhihu.com/?target=https%3A//dev.mysql.com/doc/refman/5.6/en/replication-sbr-rbr.html)。

## **Aurora 的高可用性**

无论数据库集群中的实例是否跨多个可用区，Aurora 都在单个 AWS 区域的多个可用区中存储数据库集群的数据的副本。有关 Aurora 的更多信息，请参阅[管理 Amazon Aurora 数据库集群](https://link.zhihu.com/?target=https%3A//docs.amazonaws.cn/AmazonRDS/latest/AuroraUserGuide/CHAP_Aurora.html)。

在使用单主复制的 Aurora 集群中，当您跨可用区创建读取器实例时，Amazon RDS 会自动同步地预配置和维护这些实例。主数据库实例将跨可用区同步复制到 Aurora 副本，以提供数据冗余、消除 I/O 冻结并在系统备份期间将延迟峰值降至最小。在计划内的系统维护期间，运行高性能的数据库实例可以提高可用性，并帮助保护数据库以防发生故障和可用区中断。有关可用区的更多信息，请参阅[选择区域和可用区](https://link.zhihu.com/?target=https%3A//docs.amazonaws.cn/AmazonRDS/latest/AuroraUserGuide/Concepts.RegionsAndAvailabilityZones.html)。

通过使用 RDS 控制台，您只需在创建数据库集群时指定多可用区，即可创建多可用区部署。如果数据库集群位于单个可用区中，您可以在不同可用区中添加其他数据库实例以使其成为多可用区数据库集群。

您还可以使用 CLI 指定多可用区部署。使用 AWS CLI [describe-db-instances](https://link.zhihu.com/?target=https%3A//docs.amazonaws.cn/cli/latest/reference/rds/describe-db-instances.html) 命令或 Amazon RDS API [DescribeDBInstances](https://link.zhihu.com/?target=https%3A//docs.amazonaws.cn/AmazonRDS/latest/APIReference/API_DescribeDBInstances.html) 操作可显示备用副本的可用区（称为辅助可用区）。

对于使用单主复制的集群，在创建主实例后，最多可以创建 15 个 Aurora 副本。这些只读数据库实例支持对读取密集型应用程序执行 `SELECT` 查询。我们建议您将数据库集群中的主实例和 Aurora 副本分配到多个可用区，以提高数据库集群的可用性。有关更多信息，请参阅 [可用性](https://link.zhihu.com/?target=https%3A//docs.amazonaws.cn/AmazonRDS/latest/AuroraUserGuide/Concepts.RegionsAndAvailabilityZones.html%23Aurora.Overview.Availability)。调用 [create-db-instance](https://link.zhihu.com/?target=https%3A//docs.amazonaws.cn/cli/latest/reference/rds/create-db-instance.html) AWS CLI 命令以在数据库集群中创建 Aurora 副本。包括数据库集群的名称作为 `--db-cluster-identifier` 参数值。您可以选择使用 `--availability-zone` 参数为 Aurora 副本指定可用区。

## **使用 Amazon Aurora 全局数据库**

在下文中，您可以找到 Amazon Aurora 全局数据库的说明。每个 Aurora 全局数据库均跨越多个 AWS 区域，可让用户低延迟读取全局数据，并在发生区域级故障时进行灾难恢复。

**主题**

- [Aurora 全局数据库概览](https://link.zhihu.com/?target=https%3A//docs.amazonaws.cn/AmazonRDS/latest/AuroraUserGuide/aurora-global-database.html%23aurora-global-database-overview)
- [创建 Aurora 全局数据库](https://link.zhihu.com/?target=https%3A//docs.amazonaws.cn/AmazonRDS/latest/AuroraUserGuide/aurora-global-database.html%23aurora-global-database-creating)
- [将 AWS 区域添加到 Aurora 全局数据库](https://link.zhihu.com/?target=https%3A//docs.amazonaws.cn/AmazonRDS/latest/AuroraUserGuide/aurora-global-database.html%23aurora-global-database-attaching)
- [从 Aurora 全局数据库移除集群](https://link.zhihu.com/?target=https%3A//docs.amazonaws.cn/AmazonRDS/latest/AuroraUserGuide/aurora-global-database.html%23aurora-global-database-detaching)
- [删除 Aurora 全局数据库](https://link.zhihu.com/?target=https%3A//docs.amazonaws.cn/AmazonRDS/latest/AuroraUserGuide/aurora-global-database.html%23aurora-global-database-deleting)
- [将数据导入 Aurora 全局数据库](https://link.zhihu.com/?target=https%3A//docs.amazonaws.cn/AmazonRDS/latest/AuroraUserGuide/aurora-global-database.html%23aurora-global-database-etl)
- [管理 Aurora 全局数据库](https://link.zhihu.com/?target=https%3A//docs.amazonaws.cn/AmazonRDS/latest/AuroraUserGuide/aurora-global-database.html%23aurora-global-database-managing)
- [配置 Aurora 全局数据库](https://link.zhihu.com/?target=https%3A//docs.amazonaws.cn/AmazonRDS/latest/AuroraUserGuide/aurora-global-database.html%23aurora-global-database-modifying)
- [连接到 Aurora 全局数据库](https://link.zhihu.com/?target=https%3A//docs.amazonaws.cn/AmazonRDS/latest/AuroraUserGuide/aurora-global-database.html%23aurora-global-database-connecting)
- [Aurora 全局数据库故障转移](https://link.zhihu.com/?target=https%3A//docs.amazonaws.cn/AmazonRDS/latest/AuroraUserGuide/aurora-global-database.html%23aurora-global-database-failover)
- [Aurora 全局数据库的 Performance Insights](https://link.zhihu.com/?target=https%3A//docs.amazonaws.cn/AmazonRDS/latest/AuroraUserGuide/aurora-global-database.html%23aurora-global-database-pi)
- [将 Aurora 全局数据库与其他 AWS 服务结合使用](https://link.zhihu.com/?target=https%3A//docs.amazonaws.cn/AmazonRDS/latest/AuroraUserGuide/aurora-global-database.html%23aurora-global-database-interop)



## **Aurora 全局数据库概览**

Aurora 全局数据库由一个主要 AWS 区域（包含主数据）和一个只读辅助 AWS 区域组成。Aurora 将数据复制到辅助 AWS 区域，典型延迟低于一秒。您可将写入操作直接发送到主要 AWS 区域中的主数据库实例。Aurora 全局数据库会使用专用基础设施复制您的数据，以使数据库资源可完全用于处理应用程序工作负载。具有全局足迹的应用程序可以在辅助 AWS 区域使用读取器实例实现低延迟读取。在极少的情况下，AWS 区域中的数据库会降级或被隔离，您可以提升辅助 AWS 区域以在一分钟内承担完全的读写工作负载。

存储主数据的主要 AWS 区域中的 Aurora 集群同时执行读取和写入操作。辅助区域中的集群支持低延迟读取。您可以通过添加一个或多个数据库实例（Aurora 副本）以承担只读工作负载，从而独立扩展辅助集群。为了进行灾难恢复，您可以移除和提升辅助集群，以允许完全读写操作。

只有主集群才能执行写入操作。执行写操作的客户端连接到主集群的数据库实例终端节点。

**主题**

- [Aurora 全局数据库的优势](https://link.zhihu.com/?target=https%3A//docs.amazonaws.cn/AmazonRDS/latest/AuroraUserGuide/aurora-global-database.html%23aurora-global-database.advantages)
- [Aurora 全局数据库的当前限制](https://link.zhihu.com/?target=https%3A//docs.amazonaws.cn/AmazonRDS/latest/AuroraUserGuide/aurora-global-database.html%23aurora-global-database.limitations)



### **Aurora 全局数据库的优势**

Aurora 全局数据库使用专用基础设施在主数据库集群与辅助集群之间复制更改内容。Aurora 全局数据库可为您提供以下优势：



- Aurora 全局数据库执行的复制是受限制的，不会对主数据库集群造成性能影响。数据库实例的资源完全专用于承担读取和写入工作负载。
- 更改在 AWS 区域之间以最小延迟（通常低于 1 秒）进行复制。
- 辅助集群支持快速故障转移以进行灾难恢复。您通常可以提升辅助集群并使其一分钟内可供写入。
- 远程 AWS 区域中的应用程序在从辅助集群进行读取时，经历的查询延迟会更低。
- 您可以向辅助集群添加最多 16 个 Aurora 副本，这样，您可以将读取扩展到超过单个 Aurora 集群的能力。



### **Aurora 全局数据库的当前限制**

当前以下限制适用于 Aurora 全局数据库：



- Aurora 全局数据库仅适用于与 MySQL 5.6 兼容的 Aurora。

- 无法将 `db.t2` 或 `db.t3` 实例类用于 Aurora 全局数据库。您可以选择 `db.r4` 或 `db.r5` 实例类。

- Aurora 全局数据库当前不可用于欧洲（斯德哥尔摩）和亚太地区（香港）区域。

- 辅助集群必须与主集群位于不同的 AWS 区域中。

- 您无法从与辅助集群处于相同区域中的主集群创建跨区域只读副本。有关跨区域只读副本的更多信息，请参阅[跨 AWS 区域复制 Amazon Aurora MySQL 数据库集群](https://link.zhihu.com/?target=https%3A//docs.amazonaws.cn/AmazonRDS/latest/AuroraUserGuide/AuroraMySQL.Replication.CrossRegion.html)。

- Aurora 全局数据库不支持以下功能：

- - 克隆。有关克隆的信息，请参阅[克隆 Aurora 数据库集群中的数据库](https://link.zhihu.com/?target=https%3A//docs.amazonaws.cn/AmazonRDS/latest/AuroraUserGuide/Aurora.Managing.Clone.html)。
  - 回溯。有关回溯的信息，请参阅[回溯 Aurora 数据库集群](https://link.zhihu.com/?target=https%3A//docs.amazonaws.cn/AmazonRDS/latest/AuroraUserGuide/AuroraMySQL.Managing.Backtrack.html)。
  - 并行查询。有关并行查询的信息，请参阅[使用 Amazon Aurora MySQL 的并行查询](https://link.zhihu.com/?target=https%3A//docs.amazonaws.cn/AmazonRDS/latest/AuroraUserGuide/aurora-mysql-parallel-query.html)。
  - Aurora Serverless。有关 Aurora Serverless 的信息，请参阅[使用 Amazon Aurora Serverless](https://link.zhihu.com/?target=https%3A//docs.amazonaws.cn/AmazonRDS/latest/AuroraUserGuide/aurora-serverless.html)。
  - 在全局数据库中停止和启动数据库集群。有关停止和启动 Aurora 集群的信息，请参阅[停止和启动 Amazon Aurora 数据库集群](https://link.zhihu.com/?target=https%3A//docs.amazonaws.cn/AmazonRDS/latest/AuroraUserGuide/aurora-cluster-stop-start.html)。





## **创建 Aurora 全局数据库**

Aurora 全局数据库跨多个 AWS 区域。您创建全局数据库本身以及读写主集群。然后，在另一个 AWS 区域中创建只读辅助集群。



- 要创建新的全局数据库，您创建全局数据库和它所包含的主集群，然后添加辅助集群。
- 如果您有现有 Aurora 集群，您可以拍摄快照并将其还原到新的 Aurora 全局数据库。为此，请按照[将数据导入 Aurora 全局数据库](https://link.zhihu.com/?target=https%3A//docs.amazonaws.cn/AmazonRDS/latest/AuroraUserGuide/aurora-global-database.html%23aurora-global-database-etl)中的过程进行操作。



当您为主集群和辅助集群指定名称时，请选择与现有 Aurora 集群不同的名称，即使这些集群位于其他 AWS 区域中。

**重要**
在创建 Aurora 全局数据库之前，请按照 *Amazon Relational Database Service 用户指南*的[设置 Amazon RDS](https://link.zhihu.com/?target=http%3A//docs.aws.amazon.com/AmazonRDS/latest/UserGuide/CHAP_SettingUp.html) 中的介绍针对您的账户、网络和安全设置完成设置任务。

**注意**
如果您通过 AWS CLI 或 RDS API 创建 Aurora 集群，并打算以后将其添加到 Aurora 全局数据库，则必须对引擎模式参数使用 `global` 值。

**控制台**
**使用 AWS 管理控制台创建 Aurora 全局数据库**

1. 登录 AWS 管理控制台，并针对提供 Aurora 全局数据库的区域打开 Amazon RDS 控制台。

2. 选择**创建数据库**。

3. 在**选择引擎**页面上，选择与 MySQL 5.6 兼容的 Aurora 引擎，并对 **Database location (数据库位置)** 选择 **Global (全局)**。有关示例，请参阅下面的图像。
   **注意**
   确保选择的是**标准创建**。**标准创建**显示 Aurora 全局数据库所需的选项。请勿选择**轻松创建**。

4. 1. 选择 Aurora 作为引擎：

![img](https://pic4.zhimg.com/80/v2-ca3ffda205f713ea19ab9ee9a25331ff_720w.jpg)



1. 1. 选择 MySQL 5.6 兼容性：

![img](https://pic1.zhimg.com/80/v2-10fb5f1a50a3f9af560722b7e3bb5704_720w.jpg)



1. 1. 选择 **Global (全局)** 作为位置：

![img](https://pic4.zhimg.com/80/v2-399765eebc3c08b4db5fe122a739ab5b_720w.jpg)


**注意**
选择 **Global (全局)** 将设置全局数据库和主 Aurora 集群。一旦全局数据库创建且可用，即可添加辅助 AWS 区域。



1. 对于其他 Aurora 集群使用相同的决策流程以填写剩余设置。有关创建 Aurora 数据库集群的更多信息，请参阅[创建 Amazon Aurora 数据库集群](https://link.zhihu.com/?target=https%3A//docs.amazonaws.cn/AmazonRDS/latest/AuroraUserGuide/Aurora.CreateInstance.html)。

![img](https://pic4.zhimg.com/80/v2-7c4820e22ba82827560c747b13a94777_720w.jpg)



1. 选择 **Create**。

当主数据库集群已创建并可用时，按照[将 AWS 区域添加到 Aurora 全局数据库](https://link.zhihu.com/?target=https%3A//docs.amazonaws.cn/AmazonRDS/latest/AuroraUserGuide/aurora-global-database.html%23aurora-global-database-attaching)中的步骤创建辅助集群。
**AWS CLI**
**RDS API**

## **将 AWS 区域添加到 Aurora 全局数据库**

在创建 Aurora 全局数据库及其主集群后，可以通过添加新的 AWS 区域增加其全局足迹。此过程涉及附加辅助 Aurora 集群。辅助集群必须与主集群位于不同的 AWS 区域中。

**注意**
目前，您可以附加到 Aurora 全局数据库的辅助 Aurora 集群的最大数量为一。

**控制台**
**使用 AWS 管理控制台 将 AWS 区域添加到 Aurora 全局数据库**

1. 在 AWS 管理控制台的右上角，选择可提供 Aurora 全局数据库的任何 AWS 区域。
2. 在导航窗格中，选择**数据库**。
3. 选择您要为其创建辅助集群的 Aurora 全局数据库对应的复选框。如果主集群或其中的数据库实例仍处于 `Creating` 状态，请等待它们全部变为 `Available`。
4. 对于 **Actions (操作)**，选择 **Add region (添加区域)**。

![img](https://pic1.zhimg.com/80/v2-a33468d4201764def4d42e29fc2a731c_720w.jpg)



1. 在 **Add a region (添加区域)** 页面上，选择辅助 AWS 区域。

![img](https://pic4.zhimg.com/80/v2-45d48471b65833ce310f810b8e52ecf7_720w.jpg)



1. 在新的 AWS 区域中填写 Aurora 集群的其余字段，然后选择 **Create (创建)**。


**AWS CLI**
**RDS API**

## **从 Aurora 全局数据库移除集群**

从 Aurora 全局数据库移除 Aurora 集群会将其转变回区域集群，具有完全读写能力且不再与主集群同步。

当主集群变为降级或隔离后，您可以从全局数据库移除 Aurora 集群。对 Aurora 全局数据库执行故障转移涉及从原始全局数据库中移除辅助集群，然后将其用作新 Aurora 全局数据库中的主集群。

如果您不再需要全局数据库，在删除全局数据库自身之前，必须先移除辅助集群，然后移除主集群。然后，您可以删除您移除的一个或两个集群，或者继续将其中一个或这两者用作单区域 Aurora 集群。

**控制台**
要使用 AWS 管理控制台从 Aurora 全局数据库移除 Aurora 集群，请在 **Databases (数据库)** 页面上选择此集群。对于 **Actions (操作)**，选择 **Remove from Global (从全局数据库移除)**。移除的集群将成为具有完全读写能力的常规 Aurora 集群。它不再与主集群保持同步。

![img](https://pic4.zhimg.com/80/v2-77737ab9ac6feaf97015b2a7add814ef_720w.jpg)


如果辅助集群仍与全局数据库关联，则无法移除主集群。
在移除或删除辅助集群后，您可以按同样方式移除主集群。
从 Aurora 全局数据库移除集群之后，您仍可以在 **Databases (数据库)** 页面中看到它们，但没有 **Global (全局)**、**Primary (主)** 和 **Secondary (辅助)** 标签。

![img](https://pic3.zhimg.com/80/v2-365bc0416aaceb53098a3d234fbfa83a_720w.jpg)


**AWS CLI**
**RDS API**

## **删除 Aurora 全局数据库**

在可以删除 Aurora 全局数据库之前，必须先删除或移除与全局数据库关联的辅助集群和主集群。有关从全局数据库中移除集群并使之再次成为独立 Aurora 集群的过程，请参阅[从 Aurora 全局数据库移除集群](https://link.zhihu.com/?target=https%3A//docs.amazonaws.cn/AmazonRDS/latest/AuroraUserGuide/aurora-global-database.html%23aurora-global-database-detaching)。

**控制台**
**使用 AWS 管理控制台删除 Aurora 全局数据库**
要使用 AWS 管理控制台删除作为 Aurora 全局数据库一部分的集群，应先移除或删除与全局数据库关联的任何辅助集群，再移除主集群，然后删除全局数据库本身。由于全局数据库通常容纳业务关键数据，因此您不能一步删除全局数据库和关联的集群。有关从全局数据库移除集群的说明，请参阅[从 Aurora 全局数据库移除集群](https://link.zhihu.com/?target=https%3A//docs.amazonaws.cn/AmazonRDS/latest/AuroraUserGuide/aurora-global-database.html%23aurora-global-database-detaching)。有关在移除集群后删除集群的过程，请参阅[删除 Aurora 数据库集群中的数据库实例](https://link.zhihu.com/?target=https%3A//docs.amazonaws.cn/AmazonRDS/latest/AuroraUserGuide/USER_DeleteInstance.html)。从 Aurora 集群中删除主实例将删除集群本身。

1. 确认所有其他集群从 Aurora 全局数据库中移除。
   如果全局数据库中嵌套了任何集群（如下面的屏幕截图所示），则尚无法删除全局数据库。对于与全局数据库关联的每个集群，按照[从 Aurora 全局数据库移除集群](https://link.zhihu.com/?target=https%3A//docs.amazonaws.cn/AmazonRDS/latest/AuroraUserGuide/aurora-global-database.html%23aurora-global-database-detaching)中介绍的过程操作。

![img](https://pic1.zhimg.com/80/v2-ff9ac8f0c90eb299a22d0eb52bdfe74c_720w.jpg)


在全局数据库没有任何关联的集群后，您才可以删除它。下面的屏幕截图显示 `global-db-demo` 集群在移除后不再与全局数据库关联。

![img](https://pic1.zhimg.com/80/v2-3c553049f65206633b82bb91d22889b0_720w.jpg)



1. 从 **Databases (数据库)** 页或全局数据库的详细信息页面上，依次选择 **Actions (操作)** 和 **Delete (删除)**。


**AWS CLI**
**RDS API**

## **将数据导入 Aurora 全局数据库**

要将数据导入 Aurora 全局数据库，请使用以下方法之一：



- 创建 Aurora MySQL 集群或 Amazon RDS MySQL 数据库实例的快照，然后将其还原到 Aurora 全局数据库的主集群。目前，任何快照都必须来自与 MySQL 5.6 兼容的源。

![img](https://pic3.zhimg.com/80/v2-f5871619faf35aa8b13a13e763177dee_720w.jpg)



- 对主集群使用时间点还原以便将集群数据还原到之前状态。
- 使用逻辑导入技术，例如，使用 `mysqldump` 命令准备数据，然后使用 `mysql` 命令加载这些数据。



Aurora 全局数据库的关键区别在于：您始终将数据导入您指定为主集群的集群。您可以在将集群添加到 Aurora 全局数据库之前或之后，执行初始数据导入。主集群中的所有数据自动复制到辅助集群。

## **管理 Aurora 全局数据库**

您可以对构成 Aurora 全局数据库的各个集群执行大多数的管理操作。在控制台中的**数据库**页面上选择**对相关资源分组**时，将会看到主集群和辅助集群分组到关联的全局数据库对象之下。



![img](https://pic1.zhimg.com/80/v2-15908ff249815d1570b71fbc09df1044_720w.jpg)



要查看适用于整个 Aurora 全局数据库的属性，则选择该全局数据库。



![img](https://pic1.zhimg.com/80/v2-841dd8f47611ba5e363ed956744cb924_720w.jpg)



## **配置 Aurora 全局数据库**

AWS 管理控制台中的 **Databases (数据库)** 页面列出您所有的 Aurora 全局数据库，同时显示每个全局数据库的主集群和辅助集群。Aurora 全局数据库是具有其自己的配置设置的对象，尤其是与主集群和辅助集群关联的 AWS 区域设置。



![img](https://pic1.zhimg.com/80/v2-6b10021d7a2f9867d25e3c78867b1fb8_720w.jpg)



您可以选择一个 Aurora 全局数据库并修改其设置，如下面的屏幕截图中所示。



![img](https://pic3.zhimg.com/80/v2-28a980e1098faa0c0800b39dc5572d7e_720w.jpg)



您可以为 Aurora 全局数据库中的每个 Aurora 集群独立配置参数组。大多数参数的工作方式与其他类型的 Aurora 集群相同。`aurora_enable_repl_bin_log_filtering` 和 `aurora_enable_replica_log_compression` 配置设置没有效果。我们建议在全局数据库中的所有集群之间保持设置一致，以避免在将辅助集群提升到主集群时出现意外的行为变化。例如，对于时区和字符集使用相同设置，可避免在不同集群作为主集群时出现不一致的行为。

## **连接到 Aurora 全局数据库**

对于只读查询流量，您连接到 AWS 区域中 Aurora 集群的读取器终端节点。

要运行数据操作语言 (DML) 或数据定义语言 (DDL) 语句，应连接到主集群的集群终端节点。终端节点可能位于与您的应用程序不同的 AWS 区域中。

当您在 AWS 管理控制台中查看 Aurora 全局数据库时，您可以看到与其所有集群关联的通用终端节点，如下面的屏幕截图所示。只有一个与主集群关联的集群终端节点可用于写入操作。主集群和每个辅助集群具有读取器终端节点，可用于只读查询。选择一个位于您所在的 AWS 区域或离您最近的 AWS 区域中的读取器终端节点，可最大限度地减少延迟。



![img](https://pic1.zhimg.com/80/v2-4150412825465d4222ed3b2be451dc6c_720w.jpg)



## **Aurora 全局数据库故障转移**

Aurora 全局数据库引入了比默认 Aurora 集群更高级别的故障转移功能。如果一个 AWS 区域中的整个集群变得不可用，则可以提升全局数据库中的另一个集群，使之具有读写功能。

如果另一个 AWS 区域中的集群成为主集群效果更好，您可以手动启用故障转移机制。例如，您可以提高辅助集群的容量，然后将其提升成为主集群。或者 AWS 区域间的活动平衡可能发生变化，这样，将主集群切换到另一个 AWS 区域可能会对写入操作带来较低延迟。

下面的步骤概括当您在 Aurora 全局数据库中提升辅助集群时的事件序列：



1. 主集群变得不可用。

2. 从 Aurora 全局数据库移除辅助集群。这会将它提升到完全读写能力。要了解如何从全局数据库中移除 Aurora 集群，请参阅[从 Aurora 全局数据库移除集群](https://link.zhihu.com/?target=https%3A//docs.amazonaws.cn/AmazonRDS/latest/AuroraUserGuide/aurora-global-database.html%23aurora-global-database-detaching)。

3. 重新配置应用程序，以使写入流量转向新提升的集群。
   **重要**
   此时，您停止向变为不可用的集群发出任何 DML 语句或其他写入操作。要避免集群之间的数据不一致（也称为“分裂大脑”场景），请避免写入以前的主集群，即使其重新回到在线状态。

4. 创建新的 Aurora 全局数据库，并将新提升的集群用作读写主集群。要了解如何创建 Aurora 全局数据库，请参阅[创建 Aurora 全局数据库](https://link.zhihu.com/?target=https%3A//docs.amazonaws.cn/AmazonRDS/latest/AuroraUserGuide/aurora-global-database.html%23aurora-global-database-creating)。

5. 将遇到问题的集群的 AWS 区域添加到 Aurora 全局数据库。现在，AWS 区域包含新的辅助集群。要了解如何将 AWS 区域添加到 Aurora 全局数据库，请参阅[将 AWS 区域添加到 Aurora 全局数据库](https://link.zhihu.com/?target=https%3A//docs.amazonaws.cn/AmazonRDS/latest/AuroraUserGuide/aurora-global-database.html%23aurora-global-database-attaching)。

6. 当停机问题已解决且您准备就绪再次将原始 AWS 区域指定为主集群时，按相反顺序执行相同步骤：

7. 1. 从 Aurora 全局数据库移除辅助集群。
   2. 让该集群成为原始 AWS 区域中新的 Aurora 全局数据库的主集群。
   3. 将所有写入流量重定向到原始 AWS 区域中的主集群。
   4. 添加 AWS 区域，以便在与之前相同的 AWS 区域中设置辅助集群。





## **Aurora 全局数据库的 Performance Insights**

您可以同时使用 Amazon RDS Performance Insights 和 Aurora 全局数据库。这样，可将 Performance Insights 报告分别应用于全局数据库中的每个集群。您可以针对全局数据库中的每个集群启用或禁用 Performance Insights 功能。在将新辅助区域添加到已经使用 Performance Insights 的全局数据库时，必须在新添加的集群中启用 Performance Insights。此集群不能从现有全局数据库继承 Performance Insights 设置。

在查看附加到全局数据库的数据库实例的 Performance Insights 页面时，您可以切换 AWS 区域。但是，您可能不能在切换 AWS 区域后立即看到性能详情。在各 AWS 区域中，虽然数据库实例的名称可能会相同，但每个数据库实例的相关 Performance Insights URL 不同。在切换 AWS 区域后，可在 Performance Insights 导航窗格中重新选择数据库实例的名称。

对于与全局数据库关联的数据库实例，各 AWS 区域中影响性能的系数可能不同。例如，各区域中数据库实例的容量可能不同。

有关使用 Performance Insights 的信息，请参阅[使用 Amazon RDS Performance Insights](https://link.zhihu.com/?target=https%3A//docs.amazonaws.cn/AmazonRDS/latest/AuroraUserGuide/USER_PerfInsights.html)。

## **将 Aurora 全局数据库与其他 AWS 服务结合使用**

在某些情况下，您可以将 AWS 服务和 Aurora 全局数据库结合使用。在这些情况下，您在所有关联的集群对应的 AWS 区域中需要相同的权限、外部函数等等。即使全局数据库中的 Aurora 集群可能开始只是作为只读复制目标，您以后也可能将其提升为主集群。要为这种可能性做好准备，请提前针对全局数据库中的所有 Aurora 集群，为其他服务设置任何必需的写入权限。

下面的列表汇总了要针对每项 AWS 服务采取的操作。

**调用 Lambda 函数**对于构成 Aurora 全局数据库的所有 Aurora 集群，请执行[从 Amazon Aurora MySQL 数据库集群中调用 Lambda 函数](https://link.zhihu.com/?target=https%3A//docs.amazonaws.cn/AmazonRDS/latest/AuroraUserGuide/AuroraMySQL.Integrating.Lambda.html)中的过程。
对于 Aurora 全局数据库中的每个集群，将 `aws_default_lambda_role` 集群参数设置为新 IAM 角色的 Amazon 资源名称 (ARN)。
要允许 Aurora 全局数据库中的数据库用户调用 Lambda 函数，请将您在[创建 IAM 角色以允许 Amazon Aurora 访问 AWS 服务](https://link.zhihu.com/?target=https%3A//docs.amazonaws.cn/AmazonRDS/latest/AuroraUserGuide/AuroraMySQL.Integrating.Authorizing.IAM.CreateRole.html)中创建的角色与 Aurora 全局数据库中的每个集群关联。
配置 Aurora 全局数据库中的每个集群，以允许建立到 Lambda 的出站连接。有关说明，请参阅 [启用从 Amazon Aurora MySQL 到其他 AWS 服务的网络通信](https://link.zhihu.com/?target=https%3A//docs.amazonaws.cn/AmazonRDS/latest/AuroraUserGuide/AuroraMySQL.Integrating.Authorizing.Network.html)。**从 S3 加载数据**对于构成 Aurora 全局数据库的所有 Aurora 集群，请执行[将数据从 Amazon S3 存储桶中的文本文件加载到 Amazon Aurora MySQL 数据库集群](https://link.zhihu.com/?target=https%3A//docs.amazonaws.cn/AmazonRDS/latest/AuroraUserGuide/AuroraMySQL.Integrating.LoadFromS3.html)中的过程。
对于全局数据库中的每个 Aurora 集群，将 `aurora_load_from_s3_role` 或 `aws_default_s3_role` 数据库集群参数设置为新 IAM 角色的 Amazon 资源名称 (ARN)。如果没有为 `aurora_load_from_s3_role` 指定 IAM 角色，则 Aurora 使用在 `aws_default_s3_role` 中指定的 IAM 角色。
要允许 Aurora 全局数据库中的数据库用户访问 Amazon S3，请将您在[创建 IAM 角色以允许 Amazon Aurora 访问 AWS 服务](https://link.zhihu.com/?target=https%3A//docs.amazonaws.cn/AmazonRDS/latest/AuroraUserGuide/AuroraMySQL.Integrating.Authorizing.IAM.CreateRole.html)中创建的角色与全局数据库中的每个 Aurora 集群关联。
配置全局数据库中的每个 Aurora 集群，以允许建立到 Amazon S3 的出站连接。有关说明，请参阅[启用从 Amazon Aurora MySQL 到其他 AWS 服务的网络通信](https://link.zhihu.com/?target=https%3A//docs.amazonaws.cn/AmazonRDS/latest/AuroraUserGuide/AuroraMySQL.Integrating.Authorizing.Network.html)。**将查询的数据保存到 S3**对于构成 Aurora 全局数据库的所有 Aurora 集群，请执行[将数据从 Amazon Aurora MySQL 数据库集群保存到 Amazon S3 存储桶中的文本文件](https://link.zhihu.com/?target=https%3A//docs.amazonaws.cn/AmazonRDS/latest/AuroraUserGuide/AuroraMySQL.Integrating.SaveIntoS3.html)中的过程。
对于全局数据库中的每个 Aurora 集群，将 `aurora_select_into_s3_role` 或 `aws_default_s3_role` 数据库集群参数设置为新 IAM 角色的 Amazon 资源名称 (ARN)。如果没有为 `aurora_select_into_s3_role` 指定 IAM 角色，则 Aurora 使用在 `aws_default_s3_role` 中指定的 IAM 角色。
要允许 Aurora 全局数据库中的数据库用户访问 Amazon S3，请将您在[创建 IAM 角色以允许 Amazon Aurora 访问 AWS 服务](https://link.zhihu.com/?target=https%3A//docs.amazonaws.cn/AmazonRDS/latest/AuroraUserGuide/AuroraMySQL.Integrating.Authorizing.IAM.CreateRole.html)中创建的角色与全局数据库中的每个 Aurora 集群关联。
配置全局数据库中的每个 Aurora 集群，以允许建立到 Amazon S3 的出站连接。有关说明，请参阅[启用从 Amazon Aurora MySQL 到其他 AWS 服务的网络通信](https://link.zhihu.com/?target=https%3A//docs.amazonaws.cn/AmazonRDS/latest/AuroraUserGuide/AuroraMySQL.Integrating.Authorizing.Network.html)。

## **执行 Amazon Aurora 概念验证**

在下文中，您可以找到有关如何为 Aurora 设置和运行概念验证的说明。*概念验证*是一种调查，执行此调查可了解 Aurora 是否适用于您的应用程序。概念验证可帮助您了解您自己的数据库应用程序环境中的 Aurora 功能，以及 Aurora 与您当前数据库环境相比较的情况。它还可显示移动数据、移植 SQL 代码、调整性能以及适应您当前管理过程所需完成的工作量。

在本主题中，您可以找到运行概念验证所涉高级过程和决策的概述和分步大纲，如下所示。如需详细说明，您可以使用相关链接转到特定主题的完整文档。

## **Aurora 概念验证概述**

执行 Amazon Aurora 概念验证时，可了解将现有数据和 SQL 应用程序移植到 Aurora 需采取的措施。您可使用代表生产环境的大量数据和活动大规模练习使用 Aurora 的重要内容。这样，旨在确信 Aurora 是否能够完美应对您之前的数据库基础设施无法应对的挑战。在概念验证结束时，您可制订可靠的计划进行更大规模的性能基准测试和应用程序测试。此时，您可了解进行生产部署时所需完成的最重要的工作项。

以下最佳实践建议可帮助您避免导致基准测试出现问题的常见错误。但本主题不介绍执行基准测试和优化性能的分步过程。这些过程因工作负载和所用 Aurora 功能而异。有关详细信息，请参阅与性能相关的文档，比如[管理 Aurora 数据库集群的性能和扩展](https://link.zhihu.com/?target=https%3A//docs.amazonaws.cn/AmazonRDS/latest/AuroraUserGuide/Aurora.Managing.Performance.html)、[Amazon Aurora MySQL 性能增强](https://link.zhihu.com/?target=https%3A//docs.amazonaws.cn/AmazonRDS/latest/AuroraUserGuide/Aurora.AuroraMySQL.Overview.html%23Aurora.AuroraMySQL.Performance)、[管理 Amazon Aurora PostgreSQL](https://link.zhihu.com/?target=https%3A//docs.amazonaws.cn/AmazonRDS/latest/AuroraUserGuide/AuroraPostgreSQL.Managing.html) 和[使用 Amazon RDS Performance Insights](https://link.zhihu.com/?target=https%3A//docs.amazonaws.cn/AmazonRDS/latest/AuroraUserGuide/USER_PerfInsights.html)。

本主题的信息主要适用于组织写入代码、设计架构所使用的应用程序以及支持 MySQL 和 PostgreSQL 开源数据库引擎的应用程序。如果是测试商业应用程序或使用应用程序框架生成的代码，则可能无法灵活应用这些指导原则。在此类情况下，请咨询您的 AWS 代表，了解是否有适合您的应用程序类型的 Aurora 最佳实践或案例研究。

### **1.确定目标**

作为概念验证的一部分评估 Aurora 时，您可以选择要测出哪些测量值以及如何评估练习是否成功。

您必须确保应用程序的所有功能均与 Aurora 兼容。由于 Aurora 与 MySQL 5.6 和 MySQL 5.7 接口兼容，还与 PostgreSQL 9.6 和 PostgreSQL 10.4 接口兼容，因此针对这些引擎开发的大多数应用程序也与 Aurora 兼容。不过，您仍须验证每个应用程序的兼容性。

例如，您设置 Aurora 集群时做出的某些配置选择会影响您是否可以或应该使用特定数据库功能。您可以先确定一种最通用的 Aurora 集群，即*预配置*集群。然后可以确定无服务器或并行查询等专用配置是否有益于您的工作负载。

使用以下问题帮助确定并量化您的目标：



- Aurora 是否支持工作负载的所有功能使用案例？
- 您需要多大的数据集或负载量？ 是否可以调整到该水平？
- 您对特定查询有哪些吞吐量或延迟要求？ 您是否可以达到这些要求？
- 对于计划内或计划外工作负载停机时间，您可接受的最短时间是多久？ 您是否可以达到此目标？
- 哪些运营效率指标是必需的指标？ 您是否可以正常监控这些指标？
- Aurora 是否支持您的特定业务目标，比如降低成本、扩大部署或预置速度？ 您是否可以量化这些目标？
- 您是否可以达到工作负载的所有安全合规要求？



花时间积累有关 Aurora 数据库引擎和平台功能的知识，并参阅本服务文档。注意有助于您达到预期结果的所有功能。其中一个功能可能是工作负载整合，有关说明信息，请参阅 AWS 数据库博客文章[如何计划和优化 Amazon Aurora 与 MySQL 的兼容性以实现工作负载整合](https://link.zhihu.com/?target=http%3A//amazonaws-china.com/blogs/database/planning-and-optimizing-amazon-aurora-with-mysql-compatibility-for-consolidated-workloads/)。另一个功能可能是按需扩展，有关说明信息，请参阅 *Amazon Aurora User Guide*中的[使用 Amazon Aurora Auto Scaling 扩展 Aurora 副本](https://link.zhihu.com/?target=https%3A//docs.amazonaws.cn/AmazonRDS/latest/AuroraUserGuide/Aurora.Integrating.AutoScaling.html)。 其他功能可能是性能改善或简化数据库操作等功能。

### **2.了解工作负载特性**

在预期使用案例环境下评估 Aurora。Aurora 极适用于联机事务处理 (OLTP) 工作负载。您还可以对支持实时 OLTP 数据的集群运行报告，而无需预置单独的数据仓库集群。通过核对以下特性，您可以识别您的使用案例是否属于这些类别：



- 高并发性，具有数十、数百或数千个并行客户端。
- 大量低延迟查询（几毫秒到几秒）。
- 实时短事务。
- 高选择性查询模式，基于索引查找。
- 可利用 Aurora 并行查询的分析查询（对于 HTAP）。



影响数据库选择的一个关键因素是数据的速度。*高速度*包括非常频繁地插入和更新数据。这样的系统可能涉及数千个连接，数十万项并行的数据库读写查询。高速度系统中的查询所影响的行数通常比较少，并且通常是访问同一行中的多个列。

Aurora 经设计可处理高速度数据。根据工作负载，具有单个 r4.16xlarge 数据库实例的 Aurora 集群每秒可以处理 600,000 多个 `SELECT` 语句。此外，根据工作负载，这样的集群每秒可以处理 200,000 个 `INSERT`、`UPDATE` 和 `DELETE` 语句。Aurora 是一种行存储数据库，特别适用于高容量、高吞吐量和高度并行化的 OLTP 工作负载。

Aurora 还可以对处理 OLTP 工作负载的同一集群运行报告查询。Aurora 最多支持 15 个[副本](https://link.zhihu.com/?target=https%3A//docs.amazonaws.cn/AmazonRDS/latest/AuroraUserGuide/Aurora.Replication.html%23Aurora.Replication.Replicas)，每个副本通常滞后于主实例 10–20 毫秒。分析师可以实时查询 OLTP 数据，无需将数据复制到单独的数据仓库集群。鉴于 Aurora 集群应用并行查询功能，您可以将大量处理、筛选和聚合工作分载到大规模分布式 Aurora 存储子系统。

使用此计划阶段可让自己熟悉 Aurora 的功能、其他 AWS 服务、AWS 管理控制台和 AWS CLI。此外，还可以了解这些工具如何与您打算在概念验证中使用的其他工具结合使用。

### **3.使用 AWS 管理控制台或 AWS CLI 进行练习**

下一步是使用 AWS 管理控制台或 AWS CLI 进行练习，以熟悉这些工具和 Aurora。

#### **使用 AWS 管理控制台进行练习**

下面主要介绍最初使用 Aurora 数据库集群的活动，以便您熟悉 AWS 管理控制台环境，练习设置和修改 Aurora 集群。如果将 MySQL 兼容版和 PostgreSQL 兼容版数据库引擎与 Amazon RDS 结合使用，可以在使用 Aurora 时根据这类知识执行操作。

通过利用 Aurora 共享存储模式和功能（比如复制和快照），您可以将整个数据库集群视为另一种可随意操作的对象。您可以在概念验证过程中频繁设置、拆分和更改 Aurora 集群的容量。而不会因早期做出的容量、数据库设置和物理数据布局等相关选择受到限制。

要开始使用，请设置一个空 Aurora 集群。最初试验时，可选择 **provisioned (预配置)** 容量类型和 **regional (区域)** 位置。

使用 SQL 命令行应用程序等客户端程序连接到该集群。首先，使用集群终端节点连接。连接到该终端节点可执行任何写入操作，比如数据定义语言 (DDL) 语句以及提取、转换、加载 (ETL) 流程。随后在概念验证中，可使用读取器终端节点连接查询密集型会话，该终端节点将查询工作负载分配到集群的多个数据库实例中。

通过添加更多 Aurora 副本扩展集群。有关这些步骤的信息，请参阅[使用 Amazon Aurora 进行复制](https://link.zhihu.com/?target=https%3A//docs.amazonaws.cn/AmazonRDS/latest/AuroraUserGuide/Aurora.Replication.html)。通过更改 AWS 实例类扩展或缩减数据库实例。了解 Aurora 如何简化这些类型的操作，以便在初次估计的系统容量不正确时，可以随后做出调整，而无需从头开始操作。

创建快照，并将其还原到其他集群中。

检查集群指标以了解一段时间的活动，以及这些指标如何应用于集群中的数据库实例。

刚开始时，熟悉如何通过 AWS 管理控制台执行这些操作很有用。在了解可以使用 Aurora 执行的操作后，可以继续使用 AWS CLI 自动执行这些操作。在以下部分，您可找到有关概念验证期间这些活动的过程和最佳实践的更多详细信息。

#### **使用 AWS CLI 进行练习**

我们建议自动执行部署和管理步骤，即使是在概念验证设置时也是如此。为此，请尚未熟悉 AWS CLI 的用户先熟悉相关操作。如果将 MySQL 兼容版和 PostgreSQL 兼容版数据库引擎与 Amazon RDS 结合使用，可以在使用 Aurora 时根据这类知识执行操作。

Aurora 通常包含多个排列在集群中的数据库实例组。因此，许多操作都包括确定与特定集群关联的数据库实例，然后对这些实例循环执行管理操作。

例如，您可以自动执行多个步骤，比如创建 Aurora 集群，然后使用更多实例类扩展这些集群，或使用其他数据库实例扩展这些集群。这样可帮助您在概念验证中重复任何阶段，以及使用不同种类或配置的 Aurora 集群了解假设场景。

了解 AWS CloudFormation 等基础设施部署工具的功能和限制。您可能会发现，在概念验证环境下完成的活动不适用于生产环境。例如，AWS CloudFormation 的修改行为将创建新实例，并删除当前实例，包括其数据。有关此行为的更多详细信息，请参阅 *AWS CloudFormation 用户指南*中的[堆栈资源的更新行为](https://link.zhihu.com/?target=https%3A//docs.amazonaws.cn/AWSCloudFormation/latest/UserGuide/using-cfn-updating-stacks-update-behaviors.html)。

### **4.创建 Aurora 集群**

使用 Aurora，可通过将数据库实例添加到集群，并将数据库实例扩展为更强大的实例类来了解假设场景。您还可以使用不同配置设置创建集群，以并行运行相同的工作负载。使用 Aurora，您可灵活设置、拆分和重新配置数据库集群。因此，在概念验证过程的早期阶段练习这些方法很有用。有关创建 Aurora 集群的常规过程，请参阅[创建 Amazon Aurora 数据库集群](https://link.zhihu.com/?target=https%3A//docs.amazonaws.cn/AmazonRDS/latest/AuroraUserGuide/Aurora.CreateInstance.html)。

如果可行，请先使用以下设置创建集群。仅在考虑使用某些特定使用案例时才跳过此步骤。例如，如果您的使用案例需要使用一种专用 Aurora 集群，您可以跳过此步骤。或者，如果您需要特定的数据库引擎和版本组合，也可以跳过此步骤。



- Amazon Aurora。
- MySQL 5.6 兼容性。此数据库引擎和版本组合可广泛兼容其他 Aurora 功能。
- 禁用 **quick create (快速创建)**。对于概念验证，建议您了解选择的所有设置，以便随后创建相同或略微不同的集群。
- 区域。**Global (全局)** 设置适用于特定高可用性场景。您可在初次功能和性能试验后试用此设置。
- 单个写入器，多个读取器。这是使用最广泛的通用型集群设置。此设置在集群的生命周期内保持不变。因此，如果稍后使用无服务器或并行查询等其他类型的集群做试验，需创建其他集群，并比较和对比各自的结果。
- 选择 **Dev/Test (开发/测试)** 模板。对于概念验证活动，此选择并不重要。
- 对于 **Instance Size (实例大小)**，选择 **Memory Optimized (内存优化)** 和其中一个 **xlarge** 实例类。您稍后可以向上或向下调整实例类。
- 在 **Multi-AZ Deployment (多可用区部署)** 下，选择 **Create Replica in Different Zone (在不同区域创建副本)**。许多极有用的 Aurora 内容都涉及多数据库实例集群。先在新集群中添加至少两个数据库实例是明智之举。对第二个数据库实例使用不同的可用区，有助于测试不同的高可用性场景。
- 在为数据库实例选择名称时，需使用通用命名约定。不能将任何集群数据库实例称为“master”或“writer”，因为其他数据库实例会根据需要充当这些角色。我们建议使用诸如 `clustername-az-serialnumber` 之类的名称，例如 `myprodappdb-a-01`。这些代码段可唯一标识数据库实例及其位置。
- 为 Aurora 集群设置较长的备份保留期。使用较长的保留期，可在长达 35 天的时间内执行时间点恢复 (PITR) 操作。在运行包含 DDL 和数据操作语言 (DML) 语句的测试后，可将数据库重置为已知状态。您还可以在误删除或误更改数据的情况下恢复这些数据。
- 在创建集群时，启用其他恢复、记录和监控功能。启用 **Backtrack (回溯)**、**Performance Insights**、**Monitoring (监控)** 和 **Log exports (日志导出)** 下的所有选项。启用这些功能后，您可以测试回溯、增强监控或工作负载的 Performance Insights 等功能的适用性。您还可以在概念验证期间轻松调查性能，执行故障排除操作。



### **5.设置架构**

在 Aurora 集群上，可为您的应用程序设置数据库、表、索引、外键和其他架构对象。如果是从其他 MySQL 兼容版或 PostgreSQL 兼容版数据库系统进行迁移，则要求此阶段的操作简单直接。您可以使用相同的 SQL 语法和命令行或您熟悉的适用于数据库引擎的其他客户端应用程序。

要在您的集群上发送 SQL 语句，请找到其集群终端节点，并提供该值作为连接到客户端应用程序的参数值。您可以在集群详细信息页面的 **Connectivity (连接)** 选项卡上找到集群终端节点。此集群终端节点是标记为 **Writer** 的终端节点。其他标记为 **Reader** 的终端节点表示只读连接，您可向运行报告或其他只读查询的最终用户提供这类连接。有关连接到集群时出现问题的帮助信息，请参阅[连接到 Amazon Aurora 数据库集群](https://link.zhihu.com/?target=https%3A//docs.amazonaws.cn/AmazonRDS/latest/AuroraUserGuide/Aurora.Connecting.html)。

如果是从其他数据库系统移植架构和数据，此时要求做出某些架构更改。这些架构更改要与 Aurora 中的可用 SQL 语法和功能相匹配。此时，您可能要舍弃某些列、约束、触发器或其他架构对象。如果这些对象要求改动 Aurora 兼容性，并且不能帮助您实现概念验证的目标，则这样做特别有用。

如果是从其他数据库系统迁移，且此系统使用的底层引擎与 Aurora 的不同，则需考虑使用 AWS Schema Conversion Tool (AWS SCT) 简化此过程。有关详细信息，请参阅 [AWS Schema Conversion Tool 用户指南](https://link.zhihu.com/?target=https%3A//docs.amazonaws.cn/SchemaConversionTool/latest/userguide/CHAP_Welcome.html)。有关迁移和移植活动的一般详细信息，请参阅 AWS 白皮书 [Aurora 迁移指南](https://link.zhihu.com/?target=https%3A//d1.awsstatic.com/whitepapers/Migration/amazon-aurora-migration-handbook.pdf)。

在此阶段，您可以评估您的架构设置是否存在效率低下的问题，例如在对策略或分区表等其他表结构建立索引时。在具有多数据库实例和极大工作负载的集群上部署应用程序时，这种效率低下的问题会被放大。请考虑是立即微调这类性能问题，还是在稍后的完整基准测试等活动中微调。

#v# **6.导入数据**

在概念验证期间，您需要从以前的数据库系统导入数据或代表示例。如果可行，至少在每个表中设置一些数据。这样，有助于测试所有数据类型和架构功能的兼容性。在练习使用基本 Aurora 功能后，可扩展数据量。在完成概念验证时，您应使用大得足以得出准确结论的数据集测试您的 ETL 工具、查询和全部工作负载。

您可以使用多种方法将物理或逻辑备份数据导入 Aurora。有关详细信息，请根据您在概念验证中使用的数据库引擎参阅[将数据迁移到 Amazon Aurora MySQL 数据库集群](https://link.zhihu.com/?target=https%3A//docs.amazonaws.cn/AmazonRDS/latest/AuroraUserGuide/AuroraMySQL.Migrating.html)或[将数据迁移到与 PostgreSQL 兼容的 Amazon Aurora](https://link.zhihu.com/?target=https%3A//docs.amazonaws.cn/AmazonRDS/latest/AuroraUserGuide/AuroraPostgreSQL.Migrating.html)。

使用您考虑使用的 ETL 工具和方法试验。了解哪种工具和方法最符合您的需求。您需要考虑吞吐量和灵活性。例如，某些 ETL 工具是执行一次性传输，而其他工具是将原来系统中的数据持续复制到 Aurora。

如果是从 MySQL 兼容版系统迁移到 Aurora MySQL，可以使用本机数据传输工具。如果是从 PostgreSQL 兼容版系统迁移到 Aurora PostgreSQL，同样可应用本机数据传输工具。如果是从其他数据库系统迁移，且此系统使用的底层引擎与 Aurora 的不同，则可以使用 AWS Database Migration Service (AWS DMS) 试验。有关 AWS DMS 的详细信息，请参阅[AWS Database Migration Service 用户指南](https://link.zhihu.com/?target=https%3A//docs.amazonaws.cn/dms/latest/userguide/)。

有关迁移和移植活动的详细信息，请参阅 AWS 白皮书 [Aurora 迁移指南](https://link.zhihu.com/?target=https%3A//d1.awsstatic.com/whitepapers/Migration/amazon-aurora-migration-handbook.pdf)。

### **7.移植 SQL 代码**

试验 SQL 和相关应用程序需要完成不同的工作量，具体取决于不同的案例。具体而言，工作量取决于是从 MySQL 兼容版系统还是 PostgreSQL 兼容版系统，抑或是其他类型的系统迁移。



- 如果是从 RDS MySQL 或 PostgreSQL 迁移，则 SQL 更改量很少，以致于您可以使用 Aurora 尝试执行原始 SQL 代码，并手动加入所需更改。
- 同样，如果是从与 MySQL 或 PostgreSQL 兼容的本地数据库迁移，则可以尝试执行原始 SQL 代码，并手动加入更改。
- 如果是来自其他商用数据库，则必需的 SQL 更改量会增大。在这种情况下，需考虑使用 AWS SCT。



在此阶段，您可以评估您的架构设置是否存在效率低下的问题，例如在对策略或分区表等其他表结构建立索引时。请考虑是立即微调这类性能问题，还是在稍后的完整基准测试等活动中微调。

您可以验证应用程序的数据库连接逻辑。为利用 Aurora 分布式处理，您可能需要使用单独的连接来执行读写操作，并使用相对较短的会话来完成查询操作。有关连接的信息，请参阅 [9.连接到 Aurora](https://link.zhihu.com/?target=https%3A//docs.amazonaws.cn/AmazonRDS/latest/AuroraUserGuide/aurora-poc.html%23Aurora.PoC.Connections)。

考虑是否必须妥协和取舍才能解决生产数据库中的问题。花时间研究概念验证计划，以改进您的架构设计和查询。为判定是否可以轻松提高性能、降低运营成本和提高可扩展性，可尝试在不同 Aurora 集群上并行运行原始应用程序和修改的应用程序。

有关迁移和移植活动的详细信息，请参阅 AWS 白皮书 [Aurora 迁移指南](https://link.zhihu.com/?target=https%3A//d1.awsstatic.com/whitepapers/Migration/amazon-aurora-migration-handbook.pdf)。

### **8.指定配置设置**

作为 Aurora 概念验证练习的一部分，您还可以检查您的数据库配置参数。您可能已针对性能和可扩展性在当前环境下优化了 MySQL 或 PostgreSQL 配置设置。此 Aurora 存储子系统是针对带有高速存储子系统的基于云的分布式环境进行调整和优化的。因此，许多以前的数据库引擎设置都不适用了。我们建议使用默认的 Aurora 配置设置进行初步试验。仅当您遇到性能和可扩展性瓶颈时，才重新应用您当前环境中的设置。如果您有兴趣，可在 AWS 数据库博客上的 [Aurora 存储引擎简介](https://link.zhihu.com/?target=http%3A//amazonaws-china.com/blogs/blogs/database/introducing-the-aurora-storage-engine/)中更深入地了解此主题。

Aurora 可让您对特定应用或使用案例轻松重用最佳配置设置。您无需为每个数据库实例编辑单独的配置文件，只需管理分配给整个集群或特定数据库实例的参数集。例如，时区设置适用于集群中的所有数据库实例，而您可以调整每个数据库实例的页面缓存大小设置。

您可先应用默认参数集之一，然后仅对需要微调的参数进行更改。有关使用参数组的详细信息，请参阅 [Amazon Aurora 数据库集群和数据库实例参数](https://link.zhihu.com/?target=https%3A//docs.amazonaws.cn/AmazonRDS/latest/AuroraUserGuide/USER_WorkingWithParamGroups.html%23Aurora.Managing.ParameterGroups)。有关是否适用于 Aurora 集群的配置设置，请根据您的数据库引擎参阅 [Aurora MySQL 参数](https://link.zhihu.com/?target=https%3A//docs.amazonaws.cn/AmazonRDS/latest/AuroraUserGuide/AuroraMySQL.Reference.html%23AuroraMySQL.Reference.ParameterGroups)或 [Amazon Aurora PostgreSQL 参数](https://link.zhihu.com/?target=https%3A//docs.amazonaws.cn/AmazonRDS/latest/AuroraUserGuide/AuroraPostgreSQL.Reference.html%23AuroraPostgreSQL.Reference.ParameterGroups)。

### **9.连接到 Aurora**

您会发现，在进行初始架构和数据设置并运行示例查询时，您可以连接到 Aurora 集群中的不同终端节点。使用的终端节点取决于操作是读取（比如 `SELECT` 语句）还是写入（比如 `CREATE` 或 `INSERT` 语句）。在增加 Aurora 集群上的工作负载和使用 Aurora 功能试验时，请务必让应用程序向相应的终端节点分配每个操作。

通过对写入操作使用集群终端节点，您可始终连接到集群中具有读写功能的数据库实例。默认情况下，Aurora 集群中仅一个数据库实例具有读写功能。此数据库实例被称为*主实例*。如果原始主实例不可用，则 Aurora 会激活故障转移机制，并且其他数据库实例会接任主实例。

同样，通过将 `SELECT` 语句定向到读取器终端节点，可将处理查询的工作分布到集群的各数据库实例中。通过轮询 DNS 解析可将每个读取器连接分配给不同的数据库实例。在只读数据库 Aurora 副本上完成大部分查询工作，可减少主实例的负载，让其能够处理 DDL 和 DML 语句。

使用这些终端节点可减少对硬编码主机名的依赖，并可帮助您的应用程序更快速地从数据库实例故障中恢复。

**注意**
Aurora 还具有您创建的自定义终端节点。在概念验证期间通常不需要使用这些终端节点。

Aurora 副本会受副本滞后影响，即使该滞后通常是 10 到 20 毫秒。您可以监控复制滞后，并确定该值是否在数据一致性要求的范围内。在某些情况下，读取查询可能要求严格的读取一致性（先写后读一致性）。在这些情况下，您可以继续对其使用集群终端节点，而不是读取器终端节点。

为充分利用 Aurora 功能实现分布式并行执行，您可能需要更改连接逻辑。目的是避免将所有读取请求发送到主实例。只读 Aurora 副本将使用所有相同的数据待命，准备处理 `SELECT` 语句。对应用逻辑编码可对每种操作使用相应的终端节点。请遵循以下一般准则：



- 避免对所有数据库会话都使用仅有的一个硬编码连接字符串。
- 如果可行，请将写入操作（比如 DDL 和 DML 语句）包括在客户端应用程序代码的函数中。这样，您可以使用特定连接执行不同种类的操作。
- 对查询操作指定单独的函数。Aurora 会将读取器终端节点的各新连接分配给不同的 Aurora 副本，以使读取密集型应用程序的负载均衡。
- 对于涉及多组查询的操作，在完成每组相关查询时，先断开与读取器终端节点的连接，然后再重新连接。如果软件堆栈提供连接池，请使用该功能。将查询定向到不同连接可帮助 Aurora 将读取工作负载分布到集群的各数据库实例中。



有关 Aurora 连接管理和终端节点的一般信息，请参阅[连接到 Amazon Aurora 数据库集群](https://link.zhihu.com/?target=https%3A//docs.amazonaws.cn/AmazonRDS/latest/AuroraUserGuide/Aurora.Connecting.html)。有关此主题的深入分析，请参阅 [Aurora MySQL 数据库管理员手册 – 连接管理](https://link.zhihu.com/?target=https%3A//d1.awsstatic.com/whitepapers/RDS/amazon-aurora-mysql-database-administrator-handbook.pdf)。

### **10.运行工作负载**

在准备好架构、数据和配置设置后，可以通过运行工作负载开始练习使用集群。在概念验证中使用反映生产工作负载主要内容的工作负载。我们建议始终使用实际测试和工作负载而非 sysbench 或 TPC-C 等合成基准做出性能相关决策。如果可行，请根据自己的架构、查询模式和用量收集测量值。

只要可行，请复制应用程序运行所依据的实际条件。例如，您通常在与 Aurora 集群相同的 AWS 区域和相同的 Virtual Private Cloud (VPC) 中的 Amazon EC2 实例中运行应用程序代码。如果您的生产应用程序是在跨多可用区的多个 EC2 实例中运行，请按同样的条件设置概念验证环境。有关 AWS 区域的更多信息，请参阅 *Amazon RDS 用户指南*中的[区域和可用区](https://link.zhihu.com/?target=https%3A//docs.amazonaws.cn/AmazonRDS/latest/UserGuide/Concepts.RegionsAndAvailabilityZones.html)。 要了解有关 Amazon VPC 服务的更多信息，请参阅 *Amazon VPC 用户指南*中的[什么是 Amazon VPC？](https://link.zhihu.com/?target=https%3A//docs.amazonaws.cn/vpc/latest/userguide/what-is-amazon-vpc.html)。

在验证应用程序的基本功能是否有效并且是否可以通过 Aurora 访问数据后，可以练习使用 Aurora 集群的内容。您可能要尝试使用的某些功能包括具有负载均衡功能的并发连接、并发事务和自动复制。

现在，您应该熟悉数据传输机制了，因此可以使用更大比例的示例数据运行测试。

在此阶段，您可以看到更改配置设置（比如内存限制和连接限制）的效果。您可以重新访问 [8.指定配置设置](https://link.zhihu.com/?target=https%3A//docs.amazonaws.cn/AmazonRDS/latest/AuroraUserGuide/aurora-poc.html%23Aurora.PoC.Config)中所探讨的过程。

您还可以使用创建和还原快照等机制试验。例如，您可以使用不同 AWS 实例类、AWS 副本编号等创建集群。然后在每个集群上，您可以还原相同的快照，其中包含架构和所有数据。有关该循环操作的详细信息，请参阅[创建数据库集群快照](https://link.zhihu.com/?target=https%3A//docs.amazonaws.cn/AmazonRDS/latest/AuroraUserGuide/USER_CreateSnapshotCluster.html)和[从数据库集群快照还原](https://link.zhihu.com/?target=https%3A//docs.amazonaws.cn/AmazonRDS/latest/AuroraUserGuide/USER_RestoreFromSnapshot.html)。

### **11.衡量性能**

此操作范围的最佳实践可确保设置所有适用工具和流程，以便在工作负载操作期间快速隔离异常行为。此外，您还可通过设置这些对象来了解是否可以可靠地确定所有适用原因。

您始终可以通过检查 **Monitoring (监控)** 选项卡，了解集群的当前状态，或检查一段时间的趋势。此选项卡位于每个 Aurora 集群或数据库实例的控制台详细信息页面中。其中以图表形式显示 Amazon CloudWatch 监控服务中的指标。您可以按名称、数据库实例和时间段筛选这些指标。

要在 **Monitoring (监控)** 选项卡上使用更多选项，请在集群设置中启用“Enhanced Monitoring (增强监控)”和 Performance Insights。如果在设置集群时未选择这些选项，您也可以以后启用这些选项。

为测量性能，通常需依赖显示全部 Aurora 集群活动的图表。您可以验证 Aurora 副本的负载和响应时间是否相似。您还可以了解读写主实例和只读 Aurora 副本之间的工作是如何划分的。如果数据库实例之间的负载不均衡，或存在仅影响一个数据库实例的问题，您可以针对该特定实例检查 **Monitoring (监控)** 选项卡。

在设置环境和实际工作负载以模拟生产应用程序后，可以测量 Aurora 的运行性能。要了解的最重要的问题如下：



- Aurora 每秒处理多少个查询？ 您可以检查 **Throughput (吞吐量)** 指标，以了解各种操作的数据。
- Aurora 处理给定查询平均需要多长时间？ 您可以检查 **Latency (延迟)** 指标，以了解各种操作的数据。



为此，请在 [RDS 控制台](https://link.zhihu.com/?target=https%3A//console.aws.amazon.com/rds/home)中查看给定 Aurora 集群的 **Monitoring (监控)** 选项卡，如下所示。







如果可以创建这些指标的基准值，请在当前环境下执行此操作。如果不可行，可通过执行等同于生产应用程序的工作负载来构建 Aurora 集群的相关基准。例如，运行并行用户数量和查询数量相当的 Aurora 工作负载。然后观察使用不同实例类、集群大小、配置设置等试验时这些值的变化情况。

如果吞吐量数值低于预期值，可针对工作负载进一步调查影响数据库性能的因素。同样，如果延迟数值高于预期值，也可以进一步调查。为此，需监控数据库服务器的二级指标（CPU、内存等）。您可以了解这些数据库实例是否接近其限制。您还可以了解数据库实例可使用多少附加容量来处理更多并行查询，对更大的表运行查询等。

**提示**
要检测超出预期范围的指标值，请设置 CloudWatch 警报。

在评估理想的 Aurora 集群大小和容量时，您可以找到既能达到应用程序性能峰值又不会超额配置资源的配置。其中一个重要因素是，为 Aurora 集群中的数据库实例找到适宜的大小。首先，选择一个实例大小，其 CPU 和内存容量与您当前生产环境的配置相似。为该实例大小的工作负载收集吞吐量和延迟数值。然后，将实例扩展到更大规模。了解吞吐量数值是否会增长，延迟数值是否会降低。您也可以缩减实例大小，然后了解延迟和吞吐量数值是否保持不变。目的是在可能最小的实例中实现最高的吞吐量和最短的延迟。

**提示**
使用足够的现有容量调整 Aurora 集群和相关数据库实例的大小，以应对突发的不可预测的流量高峰。对于任务关键型数据库，保留至少 20％ 的备用 CPU 和内存容量。

确保性能测试运行时间足够长，以便在热稳定状态下测量数据库性能。您可能需要运行工作负载几分钟，甚至几小时才能达到此稳定状态。运行开始时出现一些差异是正常的。出现这样的差异是因为每个 Aurora 副本都会根据所处理的 `SELECT` 查询预热其缓存。

Aurora 最适合处理涉及多个并行用户和查询的事务型工作负载。为确保以最佳性能驱动足够多的负载，您可运行使用多线程的基准测试，或同时运行多个实例的性能测试。测量使用数百个，甚至数千个并行客户端线程时的性能。模拟您希望在生产环境下使用的并行线程的数量。您还可以使用更多线程执行额外的压力测试，以测量 Aurora 可扩展性。

### **12.练习使用 Aurora 高可用性**

许多主要的 Aurora 功能都涉及高可用性。这些功能包括自动复制、自动故障转移、自动备份及时间点还原，以及将数据库实例添加到集群的功能。这类功能的安全性与可靠性对任务关键型应用程序极重要。

评估这些功能需要应用特定思维模式。在性能测量等早期活动中，您已观察了系统正常运行时的性能。测试高可用性要求您全面考虑最糟糕情况下的行为。您必须考虑到各种故障，即使很少出现这样的情况。您可以故意引发问题，以确保系统快速准确地恢复正常。

**提示**
对于概念验证，使用相同的 AWS 实例类设置 Aurora 集群中的所有数据库实例。这样，可在脱机使用数据库实例模拟故障时，试验 Aurora 可用性功能，并且不需要对性能和可扩展性进行过多更改。

我们建议在每个 Aurora 集群中至少使用两个实例。Aurora 集群中的数据库实例最多可跨三个可用区 (AZ)。前两个或前三个数据库实例各自位于不同的 AZ 中。在开始使用更大的集群时，可将数据库实例分布到 AWS 区域的所有 AZ 中。这样可提高容错能力。即使某问题影响到某个 AZ，Aurora 也可以将故障转移到其他 AZ 的数据库实例。如果您运行的集群所包含的实例超过三个，请将这些数据库实例尽可能均匀地分布到三个 AZ 上。

**提示**
Aurora 集群存储独立于数据库实例存储。每个 Aurora 集群存储始终跨越三个 AZ。
在测试高可用性功能时，可始终在测试集群中使用容量相同的数据库实例。这样可在一个数据库实例接管另一个数据库实例的任务时，避免出现不可预测的性能、延迟等变化。

要了解如何模拟失败条件以测试高可用性功能，请参阅[使用错误注入查询测试 Amazon Aurora](https://link.zhihu.com/?target=https%3A//docs.amazonaws.cn/AmazonRDS/latest/AuroraUserGuide/AuroraMySQL.Managing.FaultInjectionQueries.html)。

作为概念验证练习的一部分，其中一个目标是找到理想的数据库实例数量，并为这些数据库实例找到最佳实例类。这样需要平衡高可用性和性能要求。

对于 Aurora，集群中的数据库实例越多，高可用性的优势越大。提供更多的数据库实例，还可提高读取密集型应用程序的可扩展性。Aurora 可将 `SELECT` 查询的多个连接分布到只读 Aurora 副本中。

而另一方面，限制数据库实例的数量会减少主节点的复制流量。复制流量会消耗网络带宽，这涉及总体性能和可扩展性问题。因此，对于写入密集型 OLTP 应用程序，宁可使用较少数量的大数据库实例，也不要使用大量小数据库实例。

在典型的 Aurora 集群中，一个数据库实例（主实例）处理所有 DDL 和 DML 语句。而其他数据库实例（Aurora 副本）仅处理 `SELECT` 语句。尽管这些数据库实例的工作量并不完全相同，我们还是建议对集群中的所有数据库实例使用相同的实例类。这样，在发生故障并且 Aurora 将其中一个只读数据库实例提升为新的主实例时，此主实例的容量才会与之前的容量相同。

如果需要在同一集群中使用不同容量的数据库实例，请为这些数据库实例设置故障转移层。这些故障转移层可确定根据故障转移机制提升 Aurora 副本的顺序。将比其他数据库实例大得多或小得多的数据库实例放到较低的故障转移层。这样可确保提升时最后选择这些数据库实例。

练习使用 Aurora 的数据恢复功能，比如自动执行时间点还原、手动快照和还原以及集群回溯。如果适用，可将快照复制到其他 AWS 区域，还原到其他 AWS 区域以模拟 DR 场景。

调查您所在组织的还原时间目标 (RTO)、还原点目标 (RPO) 和地理冗余的要求。大多数组织将这些项目分类到灾难恢复这一大类下。您可在灾难恢复过程中评估本节介绍的 Aurora 高可用性功能，以确保满足您的 RTO 和 RPO 要求。

### **13.后续操作**

在概念验证过程成功结束时，您可根据预期工作负载确认 Aurora 是否是适合您的解决方案。在前面的过程中，您已确定 Aurora 在实际操作环境中的工作情况，并根据成功标准对其进行评估。

在构建数据库环境并使用 Aurora 运行后，您可以继续执行更详细的评估步骤，从而实现最终迁移和生产部署。根据个人情况，其他步骤可能包含也可能包含在概念验证过程中。有关迁移和移植活动的详细信息，请参阅 AWS 白皮书 [Aurora 迁移指南](https://link.zhihu.com/?target=https%3A//d1.awsstatic.com/whitepapers/Migration/amazon-aurora-migration-handbook.pdf)。

在下一步中，需考虑工作负载的相关安全配置，并设法满足生产环境的安全要求。计划要采用的控制措施，以保护对 Aurora 集群主用户凭证的访问权限。定义数据库用户的角色和责任，以控制对 Aurora 集群中存储数据的访问权限。考虑数据库访问应用程序、脚本和第三方工具或服务的要求。了解 AWS 服务和功能，比如 AWS Secrets Manager 和 AWS Identity and Access Management (IAM) 身份验证。

现在，您应该了解使用 Aurora 运行基准测试的过程和最佳实践。您可能发现需要执行更多性能优化操作。有关详细信息，请参阅[管理 Aurora 数据库集群的性能和扩展](https://link.zhihu.com/?target=https%3A//docs.amazonaws.cn/AmazonRDS/latest/AuroraUserGuide/Aurora.Managing.Performance.html)、[Amazon Aurora MySQL 性能增强](https://link.zhihu.com/?target=https%3A//docs.amazonaws.cn/AmazonRDS/latest/AuroraUserGuide/Aurora.AuroraMySQL.Overview.html%23Aurora.AuroraMySQL.Performance)、[管理 Amazon Aurora PostgreSQL](https://link.zhihu.com/?target=https%3A//docs.amazonaws.cn/AmazonRDS/latest/AuroraUserGuide/AuroraPostgreSQL.Managing.html) 和[使用 Amazon RDS Performance Insights](https://link.zhihu.com/?target=https%3A//docs.amazonaws.cn/AmazonRDS/latest/AuroraUserGuide/USER_PerfInsights.html)。如果执行更多优化操作，请确保熟悉您在概念验证期间收集的指标。对于下一步，您可能要使用不同的配置设置、数据库引擎和数据库版本选项创建新集群。或者，您可能要创建多种专用的 Aurora 集群来满足特定使用案例的需求。

例如，您可以针对混合事务/分析处理 (HTAP) 应用程序探索 Aurora 并行查询集群。如果广泛的地理分布对灾难恢复至关重要或者可最大程度缩短延迟，您可探索 Aurora 全局数据库。如果工作负载是间歇性的，或者您是在开发/测试场景中使用 Aurora，则可探索 Aurora 无服务器集群。

此外，生产集群也可能需要处理大量传入连接。要了解这些技术，请参阅 AWS 白皮书 [Aurora MySQL 数据库管理员手册 – 连接管理](https://link.zhihu.com/?target=https%3A//d1.awsstatic.com/whitepapers/RDS/amazon-aurora-mysql-database-administrator-handbook.pdf)。

如果您在概念验证后确定您的使用案例不适合使用 Aurora，请考虑使用以下 AWS 服务：



- 对于纯分析使用案例，工作负载可从列式存储格式以及更适合 OLAP 工作负载的其他功能中获益。可处理这类使用案例的 AWS 服务如下：

- - [Amazon Redshift](https://link.zhihu.com/?target=https%3A//docs.amazonaws.cn/redshift/)
  - [Amazon EMR](https://link.zhihu.com/?target=https%3A//docs.amazonaws.cn/emr/)
  - [Amazon Athena](https://link.zhihu.com/?target=https%3A//docs.amazonaws.cn/athena/)



- 许多工作负载都可从 Aurora 与以下一种或多种服务的组合中获益。您可以使用以下服务在这些服务之间迁移数据：

- - [AWS Glue](https://link.zhihu.com/?target=https%3A//docs.amazonaws.cn/glue/)
  - [AWS DMS](https://link.zhihu.com/?target=https%3A//docs.amazonaws.cn/dms/)
  - [从 Amazon S3 导入](https://link.zhihu.com/?target=https%3A//docs.amazonaws.cn/AmazonRDS/latest/AuroraUserGuide/AuroraMySQL.Integrating.LoadFromS3.html)，如 *Amazon Aurora User Guide*所述
  - [导出到 Amazon S3](https://link.zhihu.com/?target=https%3A//docs.amazonaws.cn/AmazonRDS/latest/AuroraUserGuide/AuroraMySQL.Integrating.SaveIntoS3.html)，如 *Amazon Aurora User Guide*所述
  - 许多其他热门 ETL 工具