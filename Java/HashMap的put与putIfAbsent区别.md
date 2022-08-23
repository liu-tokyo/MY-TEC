# put与putIfAbsent区别

put与putIfAbsent区别，put在放入数据时，如果放入数据的key已经存在与Map中，最后放入的数据会覆盖之前存在的数据，而putIfAbsent在放入数据时，如果存在重复的key，那么putIfAbsent不会放入值。

- 底层实现：

  ```java
  public V put(K key, V value) { 
       if (value == null) 
            throw new NullPointerException(); 
       int hash = hash(key.hashCode()); 
       return segmentFor(hash).put(key, hash, value, false); 
  } 
  public V putIfAbsent(K key, V value) { 
       if (value == null) 
            throw new NullPointerException(); 
       int hash = hash(key.hashCode()); 
       return segmentFor(hash).put(key, hash, value, true); 
  }
  ```

- put事例：

  ```java
  Map<Integer, String> map = new HashMap<>();
  map.put(1, "张三");
  map.put(2, "李四");
  map.put(1, "王五");
  map.forEach((key,value)->{
    System.out.println("key = " + key + ", value = " + value);
  });
  ```

  输出结果：

  ```java
  key = 1, value = 王五
  key = 2, value = 李四
  ```

- putIfAbsent事例：

  ```coffeescript
  Map<Integer, String> putIfAbsent = new HashMap<>();
  putIfAbsent.putIfAbsent(1, "张三");
  putIfAbsent.putIfAbsent(2, "李四");
  putIfAbsent.putIfAbsent(1, "王五");
  putIfAbsent.forEach((key,value)->{
	System.out.println("key = " + key + ", value = " + value);
  });
  ```

  输出结果：

  ```java
  key = 1, value = 张三
  key = 2, value = 李四
  ```

 



转载于:https://my.oschina.net/u/2608890/blog/785603