# Dozer 宣布停止维护

> https://blog.csdn.net/youanyyou/article/details/120886004

最近栈长分享了两篇 MapStruct 玩法：

- [MapStruct 基础玩法](https://mp.weixin.qq.com/s/VI5h8gppa_rdXVgo5HoWpQ)

- [MapStruct 高级玩法](https://mp.weixin.qq.com/s/pnDjW9jXXNswqMOIxMpT7g)

旨在优雅的代替满屏的 get/set 以及 BeanUtils 工具类，然后栈长也收到了一些留言，其中很多朋友就是推荐使用 Dozer 的：

栈长并没有用过 Dozer，朋友们一再推荐，一时搞得我非常好奇，这到底是何方神器，所以很想体验一下这个神器。。

不过当我打开 Dozer Github 时：

> ![img](https://img-blog.csdnimg.cn/20211021142935978.png)
>
> Dozer 项目当前不再维护了，并且将来很大可能被弃用，然后新用户不建议使用了，老用户也推荐大家迁移到 MapStruct 和 ModelMapper 等类库上面去。

栈长看了历史修改记录，是 2021/04/07 这天提交的不再维护的记录，事情已经过去大半年了，整个项目也已经大半年没有更新了。。

既然 Dozer 已经不再维护，并且即将被弃用了，我也就没有体验的必要了，当然也不推荐大家使用了，免得入坑！

## 评测报告

如果大家项目中有用到 Dozer 的，也建议考虑迁移到别的 Bean 映射工具，比如：MapStruct、Orika、ModelMapper、JMapper 等等，至于它们的性能如何，栈长找到了一篇国外的评测报告：

- https://www.baeldung.com/java-performance-mapping-frameworks

- 实测结果：

  | Framework  Name | p0.90 | p0.999 | p1.0 |
  | --------------- | ----- | ------ | ---- |
  | JMapper         | 10-3  | 0.008  | 64   |
  | MapStruct       | 10-3  | 0.010  | 68   |
  | Orika           | 0.006 | 0.278  | 32   |
  | ModelMapper     | 0.083 | 2.398  | 97   |
  | Dozer           | 0.146 | 4.526  | 118  |

  我们可以看到性能最好的显然属于 JMapper，MapStruct 紧随其后，Dozer 性能最差，当然这个评测数据仅供参考，不同的版本、环境可能还会有不同的表现。

## 搜索趋势

我们再来看下 Google 搜索趋势：

> ![img](https://img-blog.csdnimg.cn/20211021142936769.png)
>
> 可以看到，在全球过去的一年时间，MapStruct 独占鳌头，然后就是 ModelMapper 紧随其后！

## 搜索趋势 - 中国

> ![img](https://img-blog.csdnimg.cn/20211021142937436.png)
>
> 上图调整到了中国，数据很少，显然中国地区使用 Google 搜索的相对不多，但也能看到 MapStruct 确实是使用最多的，另外就是 Dozer、ModelMapper 了。

## 小结

所以，用哪个大家心中应该有个数了，个人建议尽量用主流的、用多比较多的，比如 MapStruct，毕竟它是最主流的，大家感兴趣的话可以关注公众号：Java技术栈，栈长会陆续分享更多实用教程。

至于那些坚持写满屏的 get/ set 和 BeanUtils 的也没有毛病，只要代码运行不出错，怎么写都没有问题的。不管用什么，实际工作中也不是个人能选择的，需要遵守整体技术团队的规范。

话说你们公司用的哪个呢？欢迎投票分享！

所以，你还在用 Dozer 吗？赶紧发给身边的同事看看吧，及时迁移到别的主流类库上，不然时间久了可能给系统带来隐患。

好了，今天的分享就到这里了，后面栈长会分享更多好玩的 Java 技术和最新的技术资讯，关注公众号Java技术栈第一时间推送，我也将主流 Java 面试题和参考答案都整理好了，在公众号后台回复关键字 "面试" 进行刷题。

## 参考

- [超轻量级对象复制转换-比dozer快100倍](https://blog.csdn.net/zla85/article/details/48317831?spm=1001.2101.3001.6650.2&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7ERate-2-48317831-blog-120886004.235%5Ev38%5Epc_relevant_sort_base1&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7ERate-2-48317831-blog-120886004.235%5Ev38%5Epc_relevant_sort_base1&utm_relevant_index=5)
- [干掉 BeanUtils！试试这款 Bean 自动映射工具，真心强大](https://mp.weixin.qq.com/s/VI5h8gppa_rdXVgo5HoWpQ)