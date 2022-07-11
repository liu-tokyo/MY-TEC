# ConcurrentHashMap简介

## 一、jdk1.8之前的数据结构：

1. 默认情况下会有16个区段 Segment数组 Segment[16]

2. 每次每个区段Segment中会保存若干个散列桶，每次散列桶长度扩容成2^n次方的长度。 多个散列桶相连就构成了散列表。

3. 存入元素： key带入到hashcode方法众获得hashcode值，然后把hashcode值带入到散列算法中获取segment的下标(区段编号)，再根据key带入到定义好的函数中获取Segment对象中散列桶的下标。

   如果此位置有元素就构成链表(JDK1.8及以后会形成红黑树)，如果没有元素就存入。

3. 存取的线程安全问题： 如果多个线程操作同一个Segment，则某个线程给此Segment加锁，另一个线程只能阻塞。

   同时解决了HashTable的问题，HashTable只能由一个线程操作。 ConcurrentHashMap可以让一个线程操作第一个Segment，另一个线程操作另一个Segment。

4. 小矩形块表示散列桶

   绿色的Segment表示ConcurrentHashMap集合众`Segment[16]`数组里的一个对象。

5. 并发问题：

   两个线程给不同的区段Segment中添加元素，这种情况可以并发。 所以ConcurrentHashMap可以保证线程安全（多个线程操作同一个Segment，则某个线程给此Segment加锁，另一个线程只能阻塞）并且在一定程度上提交线程并发执行效率。

   两个线程给同一个区段Segment中添加元素，这种情况不可以并发， 这样JDK1.8进行了改进：

   没有区段了，和HashMap一致了，数组+链表+红黑树 +乐观锁 + synchronized

   ![img](https://img-blog.csdnimg.cn/9b438668c7a545eb9c48e28538c765a6.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5a2k54us5paX5aOr,size_20,color_FFFFFF,t_70,g_se,x_16)

## 二、乐观锁和悲观锁：
- 2.1.**悲观锁**：执行的某个线程总是悲观的认为，在自己执行期间，总有其它线程与之并发执行,认为会产生安全问题，所以为了保证线程安全,在线程刚开始访问对象数据时，后立即给对象加锁，从而保证线程安全.。成了同步的效果(就是排队执行，第一个线程执行完第二个才开始)
  Synchronized就是悲观锁。

- 2.2.**乐观锁**：

  ![img](https://img-blog.csdnimg.cn/c8921ab9be4542bda2852f0ba40695d9.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5a2k54us5paX5aOr,size_19,color_FFFFFF,t_70,g_se,x_16)

  ![img](https://img-blog.csdnimg.cn/e978508f35474578bf7b820b9c816b56.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5a2k54us5paX5aOr,size_19,color_FFFFFF,t_70,g_se,x_16)

  ![img](https://img-blog.csdnimg.cn/a15d4aaf09fd4975ae5801cd8ac0e41b.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5a2k54us5paX5aOr,size_19,color_FFFFFF,t_70,g_se,x_16)

  ![img](https://img-blog.csdnimg.cn/560120044a484256a1aff21a51e72884.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5a2k54us5paX5aOr,size_19,color_FFFFFF,t_70,g_se,x_16)

  ![img](https://img-blog.csdnimg.cn/f21c2d6bfe5c4016bc3b6b2c57a32835.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5a2k54us5paX5aOr,size_19,color_FFFFFF,t_70,g_se,x_16)

**ConcurrentHashmap和HashMap区别：**

1. HashMap是非线程安全的，而HashTable和ConcurrentHashmap都是线程安全的。

2. HashMap的key 和value均可以为null；而HashTable和ConcurrentHashMap的key和value均不可以为null。

3. HashTable 和ConcurrentHashMap的区别：

   保证线程安全的方式不同：
   
   - 3.1.HashTable是通过给整张散列表加锁的方式来保证线程安全,这种方式保证了线程安全，但是并发执行效率低下。
   - 3.2.ConcurrentHashMap在JDK1.8之前，采用分段锁机制来保证线程安全的，这种方式可以在保证线程安全的同时，一定程度上提高并发执行效率(当多线程并发访问不同的segment时，多线程就是完全并发的，并发执行效率会提高)
   - 3.3.从JDK1.8开始， ConcurrentHashMap数据结构与1.8中的HashMap保持一致，均为数组+链表+红黑树，是通过乐观锁+Synchronized来保证线程安全的。当多线程并发向同一个散列桶添加元素时。若散列桶为空，此时触发乐观锁机制，线程会获取到桶中的版本号，在添加节点之前，判断线程中获取的版本号与桶中实际存在的版本号是否一致，若一致，则添加成功，若不一致，则让线程自旋。
   
   若散列桶不为空，此时使用Synchronized来保证线程安全，先访问到的线程会给桶中的头节点加锁，从而保证线程安全。
   

## 三、现在的数据结构：
JDK1.8中ConcurrentHashmap保证线程安全的方式：乐观锁+Sysnchronized

多线程并发向同一个散列桶添加元素时若散列桶为空，则触发乐观锁机制，线程获取散列桶中的版本号，在添加元素之前判断线程中的版本号与桶中的版本号是否一致。

如果一致，添加成功；如果不一致，自旋若散列桶不为空，则使用synchroinized。先访问到的线程给头结点解锁，形成链表或红黑树，JDK1.8中ConcrruentHashMap在保证线程安全的同时允许最大诚度的多线程并发执行。

- ConcrruentHashMap中的乐观锁说明：

  ![img](https://img-blog.csdnimg.cn/e5eb5866b02448309aed87f81dde1255.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5a2k54us5paX5aOr,size_19,color_FFFFFF,t_70,g_se,x_16)

- **HashTable和ConcurrentHashMap保证线程安全的方式**

  HashTable是通过给整张散列表加锁的方式来保证线程安全的。这种方式可以保证线程安全，但是并发执行效率极其低下。(同步)

  以上保证线程安全的方式中，有-些可以并发执行的操作是得不到并发的，所以并发执行效率有可提升的空间。

  ```java
  public static void main(String[] args) {
   //    TreeMap map=new TreeMap();//红黑树的一种实现
   //    map.put("tom",90);
   //    map.put("jack",80);
   //    map.put("rose",70);
   //    map.put("amy",88);
   //    map.put("tony",98);
   //    map.put("lisa",92);
   //    System.out.println(map);
  
  
       TreeSet set=new TreeSet();//红黑树的一种实现
       set.add("tom");
       set.add("jack");
       set.add("rose");
       set.add("amy");
       set.add("tony");
  
     }
  ```
  
  
