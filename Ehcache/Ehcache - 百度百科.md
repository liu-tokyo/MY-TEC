# Ehcache - 百度百科



EhCache 是一个纯Java的进程内缓存框架，具有快速、精干等特点，是Hibernate中默认的CacheProvider。

## 1. 基本介绍

Ehcache是一种广泛使用的开源Java分布式缓存。主要面向通用缓存，Java EE和轻量级容器。它具有内存和磁盘存储，缓存加载器，缓存扩展，缓存异常处理程序，一个gzip缓存servlet过滤器，支持REST和SOAP api等特点。

Ehcache最初是由Greg Luck于2003年开始开发。2009年，该项目被Terracotta购买。软件仍然是开源，但一些新的主要功能(例如，快速可重启性之间的一致性的)只能在商业产品中使用，例如Enterprise EHCache and BigMemory。维基媒体Foundationannounced目前使用的就是Ehcache技术。

图1是 Ehcache 在应用程序中的位置：

[![图1](https://bkimg.cdn.bcebos.com/pic/5fdf8db1cb1349540ec1440e564e9258d0094af8?x-bce-process=image/resize,m_lfit,w_1280,limit_1/format,f_auto)](https://baike.baidu.com/pic/ehcache/6036099/0/fc5e5f3490e8a914241f1419?fr=lemma&ct=single)图1

## 1. 特性

主要的特性有：

1. 快速

2. 简单

3. 多种缓存策略

4. 缓存数据有两级：内存和磁盘，因此无需担心容量问题

5. 缓存数据会在[虚拟机](https://baike.baidu.com/item/虚拟机)重启的过程中写入磁盘

6. 可以通过RMI、可插入API等方式进行分布式缓存

7. 具有缓存和缓存管理器的侦听接口

8. 支持多[缓存](https://baike.baidu.com/item/缓存)管理器实例，以及一个实例的多个缓存区域

9. 提供Hibernate的缓存实现

## **3. 集成**

可以单独使用，一般在第三方库中被用到的较多，对分布式支持不够好，多个节点不能同步，通常和`redis`一起使用。

## 4. Ehcache和Redis比较

1. ehcache直接在jvm虚拟机中缓存，速度快，效率高；但是缓存共享麻烦，集群分布式应用不方便。

2. redis是通过socket访问到缓存服务器，效率比ehcache低，比数据库快。处理集群和分布式缓存方便，有成熟的方案。如果单个应用对缓存访问要求高可用ehcache，如果是大型系统，存在缓存共享、分布式部署、缓存内容大，建议用redis。

**ehcache也有缓存共享方案，不过是通过RMI或者Jgroup多播方式进行广播缓存通知更新，缓存共享复杂，维护不方便；简单的共享可以，但涉及缓存恢复、大数据缓存，则不适合。**

## 5. getting start

### 5.1 Ehcache配置

开始使用Ehcache，需要先配置cache manager和cache，可通过编程的方式和xml的方式进行配置。

- 编程方式配置

  cachemanager 管理cache

  ```java
  acheManager cacheManager = CacheManagerBuilder.newCacheManagerBuilder()     .withCache("preConfigured",        CacheConfigurationBuilder.newCacheConfigurationBuilder(Long.class, String.class, ResourcePoolsBuilder.heap(10)))；
  
  Cache preConfigured =    cacheManager.getCache("preConfigured", Long.class, String.class);
  ```

- XML配置

  ```xml
  <config>
      <!-- 别名foo -->
      <cache alias="foo">
          <!-- 指定建类型，若不指定则是Object类型 -->
          <key-type>java.lang.String</key-type>
         	<!-- 指定值类型，若不指定则是Object类型 -->
          <value-type>java.lang.String</value-type>
          <resource>
              <heap unite="entries">2000</heap>
              <offheap unit-"mb">600</offheap>
          </resource>
  	</cache>
      <!-- cache模版，可被继承使用 -->
  	<cache-template name="myDefault">
          <key-type>java.lang.Long</key-type>
          <value-type>java.lang.String</value-type>
  	<heap unit="entries">200</heap>
      </cache-template>
      <!-- 继承myDefault模版，并覆盖key类型 -->
      <cache alias="bar" uses-template="myDefault">
          <key-type>java.lang.Number</key-type>
      </cache>
  </config>
  ```

  

### 5.2 Ehcache 层次缓存选项

ehcache支持基于层的缓存。

移出堆内存：

1、当添加一个映射到缓存，需要将key和value序列化2、当读取一个映射，需要将key和value反序列化。

序列化和反序列化影响缓存性能

- 单一层设置

  ehcahce所有的层可单独配置，有

  - heap堆内存，最快的层，不须经过序列化和反序列化
  - offheap对外内存，移出heap需经序列化和反序列化，较heap要满
  - disk磁盘，持久化，在jvm重启时缓存会保留下来
  - clustered集群，需Terracotta Server Array 

- 多层同时配置，需注意
  - 多层配置时，必需有heap层
  - disk和clustered不能同时配置
  - 各层应使用金字塔是配置，即下层存储要大于上层的存储空间

- 层次结构

  ![img](https:////upload-images.jianshu.io/upload_images/12112066-3b9d5c095f125cf6.JPG?imageMogr2/auto-orient/strip|imageView2/2/w/287/format/webp)

- 资源池

  一般使用ResourcePoolBuilder，资源池只是制定配置，不是实际的一个可被缓存使用的池。

  更新资源池，可在正被使用的cache上对资源池的配置进行修改，但可修改是有限的

  销毁持久层，PersistentCacheManager.destroy（）方法

**多层缓存操作中的序列化流**

- put过程

  ![img](https:////upload-images.jianshu.io/upload_images/12112066-33b55ee2f700656f.JPG?imageMogr2/auto-orient/strip|imageView2/2/w/405/format/webp)



- get过程

  ![img](https:////upload-images.jianshu.io/upload_images/12112066-8139d7021bbcb203.JPG?imageMogr2/auto-orient/strip|imageView2/2/w/481/format/webp)



当put一个值到cache，直接会存到author层，最底层

get操作会把值推到cacheing层

只要值被put到author层，全部的高层都置无效

全部访问未命中都会访问到author层为止

## 6. 创建集群支持的缓存管理

- 启用 Terracotta服务，配置集群存储

  ![img](https://upload-images.jianshu.io/upload_images/12112066-17b40c71ac3e990c.JPG?imageMogr2/auto-orient/strip|imageView2/2/w/720/format/webp)

- 集群框架

  因为ehcache对于分布式共享并不是很好的支持，故不详细说明，与redis结合是更好的方式。

