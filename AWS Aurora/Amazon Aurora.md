# Amazon Aurora

Amazon Aurora 是一种与 MySQL 和 PostgreSQL 兼容的关系数据库，专为云而打造，既具有传统企业数据库的性能和可用性，又具有开源数据库的简单性和成本效益。

Amazon Aurora 的速度最高可以达到标准 MySQL 数据库的五倍、标准 PostgreSQL 数据库的三倍。它可以实现商用数据库的安全性、可用性和可靠性，而成本只有商用数据库的 1/10。Amazon Aurora 由 [Amazon Relational Database Service (RDS)](https://aws.amazon.com/cn/rds/) 完全托管，RDS 可以自动执行各种耗时的管理任务，例如硬件预置以及数据库设置、修补和备份。

Amazon Aurora 采用一种有容错能力并且可以自我修复的分布式存储系统，这一系统可以把每个数据库实例扩展到最高 128TB。它具备高性能和高可用性，支持最多 15 个低延迟读取副本、时间点恢复、持续备份到 Amazon S3，还支持跨三个可用区复制。

## 优势

### 高性能和可扩展性

其吞吐量最高可以达到标准 MySQL 的 5 倍、标准 PostgreSQL 的 3 倍。这种性能与商用数据库相当，而成本只有商用数据库的 1/10。您可以根据需要轻松地将数据库部署从较小的实例类型扩展到较大的实例类型，或者让 Aurora Serverless 自动为您处理扩展。要提高读取容量和性能，您可以跨三个可用区添加最多 15 个低延迟读取副本。Amazon Aurora 可以在需要时自动增加存储，每个数据库实例最高 128TB。

### 高可用性和持久性

Amazon Aurora 旨在提供 99.99% 的可用性，可跨 3 个可用区复制 6 份数据，并能将数据持续备份到 Amazon S3 中。它能以透明的方式从物理存储故障中恢复，实例故障转移用时通常不超过 30 秒。您也可以在几秒钟内回溯到以前的时间点，以从用户错误中恢复。使用全球数据库，单个 Aurora 数据库可以跨越多个 AWS 区域，从而实现快速本地读取和快速灾难恢复。

### 高度安全

Amazon Aurora 可以为您的数据库提供多个级别的安全性。其中包括：使用 Amazon VPC 进行网络隔离，使用您通过 AWS Key Management Service (KMS) 创建和控制的密钥执行静态加密，以及使用 SSL 对动态数据进行加密。在加密的 Amazon Aurora 实例上，底层存储中的数据会被加密，在同一个集群中的自动备份、快照和副本也会被加密。

### 与 MySQL 和 PostgreSQL 兼容

Amazon Aurora 数据库引擎可与现有的 MySQL 和 PostgreSQL 开源数据库完全兼容，还会定期实现对新版本的支持。这意味着您可以使用 MySQL 或 PostgreSQL 导入/导出工具或者快照，将 MySQL 或 PostgreSQL 数据库轻松迁移到 Aurora。这也意味着您用于现有数据库的代码、应用程序、驱动程序和工具能够与 Amazon Aurora 配合使用，只需对其进行少量更改或不需要更改。

### 完全托管

Amazon Aurora 由 Amazon Relational Database Service (RDS) 完全托管。您再也无需担心硬件预置、软件修补、设置、配置或备份等数据库管理任务。Aurora 会自动持续监控您的数据库并将其备份到 Amazon S3，因此可以实现精细的时间点恢复。您可以使用 Amazon CloudWatch、增强监控功能或者 Performance Insights 这种可以帮您快速检测性能问题并且易于使用的工具来监控数据库的性能。

### 迁移支持

Amazon Aurora 可与 MySQL 和 PostgreSQL 兼容，是一种将数据库迁移到云的出色工具。如果您要从 MySQL or PostgreSQL 迁移，请参阅我们的[迁移文档](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Welcome.html)，查看可以使用的工具和选项列表。要从商用数据库引擎迁移，您可以使用 [AWS Database Migration Service](https://aws.amazon.com/cn/dms/) 来实现安全迁移并尽可能缩短停机时间。

## 使用案例

### 企业应用程序

Amazon Aurora 是任何可以使用关系数据库的企业应用程序的出色选择。与商用数据库相比，Amazon Aurora 可帮助您将数据库成本削减 90% 或更多，同时提高数据库的可靠性和可用性。Amazon Aurora 是一项完全托管的服务，可自动执行耗时的任务（如预置、修补、备份、恢复、故障检测和修复），从而帮您节省时间。

## 软件即服务 (SaaS) 应用程序

SaaS 应用程序通常使用多租户架构，在实例和存储扩展性方面需要极大的灵活性，还需要高性能和高可靠性。Amazon Aurora 在一个托管数据库产品中提供了所有这些特性，可帮助 SaaS 公司专注于构建高质量的应用程序，而无需担心支持应用程序的底层数据库

### Web 和移动游戏

Web 和移动游戏需要以极大的规模运行，因此要求数据库具有高吞吐量、大规模存储可扩展性和高可用性。Amazon Aurora 满足了此类高要求应用程序的需求，同时还具有足够的未来成长空间。Amazon Aurora 没有许可限制，因此它能完美匹配这些应用程序的可变使用模式。