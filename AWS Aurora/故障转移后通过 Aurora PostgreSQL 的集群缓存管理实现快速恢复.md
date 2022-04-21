# 故障转移后通过 Aurora PostgreSQL 的集群缓存管理实现快速恢复

- https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/AuroraPostgreSQL.cluster-cache-mgmt.html

为了在发生故障转移时快速恢复 Aurora PostgreSQL 集群中的写入器数据库实例，请使用 Amazon Aurora PostgreSQL 的集群缓存管理。集群缓存管理可确保在发生故障转移时保持应用程序性能。

在典型的故障转移情况下，您可能会在故障转移后看到暂时但较大的性能下降。发生这种降级是因为当故障转移数据库实例启动时，缓冲区缓存为空。空缓存也称为*冷缓存*。冷缓存会降低性能，因为数据库实例必须从较慢的磁盘读取，而不是利用缓冲区缓存中存储的值。

通过集群缓存管理，您可以将特定的读取器数据库实例设置为故障转移目标。集群缓存管理确保指定读取器缓存中的数据与写入器数据库实例缓存中的数据保持同步。具有预填充值的指定读取器缓存称为*热缓存*。如果发生故障转移，指定的读取器会在升级到新的写入器数据库实例时立即使用其暖缓存中的值。这种方法为您的应用程序提供了更好的恢复性能。

集群缓存管理要求指定的读取器实例与写入器具有相同的实例类类型和大小（`db.r5.2xlarge`或`db.r5.xlarge`，例如）。在创建 Aurora PostgreSQL 数据库集群时请记住这一点，以便您的集群可以在故障转移期间恢复。有关实例类类型和大小的列表，请参阅 [Aurora 的数据库实例类的硬件规格](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/Concepts.DBInstanceClass.html#Concepts.DBInstanceClass.Summary)。

**笔记**

属于 Aurora 全局数据库的 Aurora PostgreSQL 数据库集群不支持集群缓存管理。

**内容**

- 配置集群缓存管理
  - [启用集群缓存管理](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/AuroraPostgreSQL.cluster-cache-mgmt.html#AuroraPostgreSQL.cluster-cache-mgmt.Enable)
  - [设置写入器数据库实例的升级层优先级](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/AuroraPostgreSQL.cluster-cache-mgmt.html#AuroraPostgreSQL.cluster-cache-mgmt.Writer)
  - [为读取器数据库实例设置提升层优先级](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/AuroraPostgreSQL.cluster-cache-mgmt.html#AuroraPostgreSQL.cluster-cache-mgmt.Reader)
- [监控缓冲区缓存](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/AuroraPostgreSQL.cluster-cache-mgmt.html#AuroraPostgreSQL.cluster-cache-mgmt.Monitoring)

## 配置集群缓存管理

要配置集群缓存管理，请按顺序执行以下过程。

**话题**

- [启用集群缓存管理](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/AuroraPostgreSQL.cluster-cache-mgmt.html#AuroraPostgreSQL.cluster-cache-mgmt.Enable)
- [设置写入器数据库实例的升级层优先级](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/AuroraPostgreSQL.cluster-cache-mgmt.html#AuroraPostgreSQL.cluster-cache-mgmt.Writer)
- [为读取器数据库实例设置提升层优先级](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/AuroraPostgreSQL.cluster-cache-mgmt.html#AuroraPostgreSQL.cluster-cache-mgmt.Reader)

**笔记**

完成这些步骤后至少等待 1 分钟，集群缓存管理才能完全运行。

### 启用集群缓存管理

要启用集群缓存管理，请执行以下步骤。



## Console

**启用集群缓存管理**

1. 登录 AWS 管理控制台并在 [https://console.aws.amazon.com/rds/打开 Amazon RDS 控制台](https://console.aws.amazon.com/rds/).

2. 在导航窗格中，选择**参数组**。

3. 在列表中，为您的 Aurora PostgreSQL 数据库集群选择参数组。

   数据库集群必须使用默认参数组以外的参数组，因为您无法更改默认参数组中的值。

4. 对于**参数组操作**，选择 **编辑**。

5. 将`apg_ccm_enabled`集群参数的值设置为**1**。

6. 选择**保存更改**。



## AWS CLI

### 设置写入器数据库实例的升级层优先级

对于集群缓存管理，请确保Aurora PostgreSQL 数据库集群的写入数据库实例的提升优先级**为 0 层。***提升层优先级*是一个值，它指定 Aurora 读取器在失败后提升到写入器数据库实例的顺序。有效值为 0–15，其中 0 是第一优先级，15 是最后一个优先级。有关升级层的更多信息，请参阅[Aurora 数据库集群的容错](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/Concepts.AuroraHighAvailability.html#Aurora.Managing.FaultTolerance)。



## Console

**将写入器数据库实例的升级优先级设置为第 0 层**

1. 登录 AWS 管理控制台并在 [https://console.aws.amazon.com/rds/打开 Amazon RDS 控制台](https://console.aws.amazon.com/rds/).
2. 在导航窗格中，选择 **Databases**。
3. 选择 Aurora PostgreSQL 数据库集群的**写入器**数据库实例。
4. 选择**修改**。出现**修改数据库实例**页面。
5. 在**Additional configuration**面板上，选择 **tier-0**作为**Failover priority**。
6. 选择**继续**并检查修改摘要。
7. 要在保存更改后立即应用更改，请选择 **立即应用**。
8. 选择**Modify DB Instance**以保存您的更改。



## AWS CLI

### 为读取器数据库实例设置提升层优先级

您设置一个读取器数据库实例用于集群缓存管理。为此，请从 Aurora PostgreSQL 集群中选择一个与写入器数据库实例具有相同实例类和大小的读取器。例如，如果作者使用`db.r5.xlarge`，请选择使用相同实例类类型和大小的阅读器。然后将其晋升层级优先级设置为 0。

*提升层优先级*是一个值，它指定 Aurora 读取器在失败后提升到写入器数据库实例的顺序。有效值为 0–15，其中 0 是第一优先级，15 是最后一个优先级。



## Console

**将读取器数据库实例的升级优先级设置为第 0 层**

1. 登录 AWS 管理控制台并在 [https://console.aws.amazon.com/rds/打开 Amazon RDS 控制台](https://console.aws.amazon.com/rds/).
2. 在导航窗格中，选择**Databases**。
3. 选择 Aurora PostgreSQL 数据库集群的**读取器**数据库实例，该实例与写入器数据库实例具有相同的实例类。
4. 选择**修改**。出现**修改数据库实例**页面。
5. 在**Additional configuration**面板上，选择 **tier-0**作为**Failover priority**。
6. 选择**继续**并检查修改摘要。
7. 要在保存更改后立即应用更改，请选择 **立即应用**。
8. 选择**Modify DB Instance**以保存您的更改。



## AWS CLI

## 监控缓冲区缓存

设置集群缓存管理后，您可以监控写入器数据库实例的缓冲区缓存与指定读取器的暖缓冲区缓存之间的同步状态。要检查写入器数据库实例和指定读取器数据库实例上的缓冲区缓存内容，请使用 PostgreSQL `pg_buffercache`模块。有关更多信息，请参阅[PostgreSQL `pg_buffercache`文档](https://www.postgresql.org/docs/current/pgbuffercache.html).

**使用`aurora_ccm_status`功能**

集群缓存管理也提供了该`aurora_ccm_status` 功能。使用`aurora_ccm_status`写入器数据库实例上的函数获取有关指定读取器上缓存预热进度的以下信息：

- `buffers_sent_last_minute`– 在最后一分钟有多少缓冲区已发送到指定的阅读器。
- `buffers_sent_last_scan`– 在缓冲区高速缓存的最后一次完整扫描期间，有多少缓冲区已发送到指定的读取器。
- `buffers_found_last_scan`– 在缓冲区高速缓存的最后一次完整扫描期间，有多少缓冲区已被识别为频繁访问并需要发送。不发送已缓存在指定读取器上的缓冲区。
- `buffers_sent_current_scan`– 在当前扫描期间到目前为止已发送了多少缓冲区。
- `buffers_found_current_scan`– 在当前扫描中，有多少缓冲区被识别为经常访问。
- `current_scan_progress`– 在当前扫描期间到目前为止已访问了多少缓冲区。

下面的示例展示了如何使用该`aurora_ccm_status`函数将其部分输出转换为热率和热百分比。

```sql
SELECT buffers_sent_last_minute*8/60 AS warm_rate_kbps, 
   100*(1.0-buffers_sent_last_scan/buffers_found_last_scan) AS warm_percent 
   FROM aurora_ccm_status();
```