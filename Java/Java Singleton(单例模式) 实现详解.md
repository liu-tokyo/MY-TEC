# Java Singleton(单例模式) 实现详解

## 什么是单例模式？

Intend:Ensure a class only has **one instance**, and provide a **global** point of access to it.

目标：保证一个类只有一个实例，并提供全局访问点

--------（《设计模式：可复用面向对象软件的基础》

就运行机制来说，就是一个类，在运行过程中只存在一份内存空间，外部的对象想使用它，都只会调用那部分内存。

其目的有实现唯一控制访问，节约资源，共享单例实例数据。

单例基础实现有两种思路，

- 1.Eager initialization：在加载类时构造；
- 2.Lazy Initialization：在类使用时构造；
- 3.利用静态内部类实现懒加载。

## 1. Eager initialization

适用于高频率调用，其由于预先创建好Singleton实例会在初始化时使用跟多时间，但在获得实例时无额外开销。

- 其典型代码如下：

  ```
  public class EagerInitSingleton {
      //构建实例
      private static final EagerInitSingleton SINGLE_INSTANCE = new EagerInitSingleton();
      //私有化构造器
      private EagerInitSingleton(){}
      //获得实例
      public static EagerInitSingleton getInstance(){
          return SINGLE_INSTANCE;
      }
  }
  ```

- 换一种思路，由于类内静态块也只在类加载时运行一次，所以也可用它来代替构造单例：

  ```
  public class EagerInitSingleton {
      //构建实例
      //private static final EagerInitSingleton instance = new EagerInitSingleton();
      //此处不构造
      private static StaticBlockSingleton instance;
  
      //使用静态块构造
      static{
          try{
              instance = new StaticBlockSingleton();
          }catch(Exception e){
              throw new RuntimeException("Exception occured in creating singleton instance");
          }
      }
      
      //私有化构造器
      private EagerInitSingleton(){}
      //获得实例
      public static EagerInitSingleton getInstance(){
          return instance;
      }
  }
  ```

## 2. Lazy Initialization

适用于低频率调用，由于只有使用时才构建Singleton实例，在调用时会有系列判断过程所以会有额外开销

### 2.1 Lazy Initialization单线程版 

- 其初步实现如下：

  ```
  public class LazyInitSingleton {
  
      private static LazyInitSingleton SINGLE_INSTANCE = null;
  
      //私有化构造器
      private LazyInitSingleton() {}
  
      //构造实例
      public static LazyInitSingleton getInstance() {
          if (SINGLE_INSTANCE == null) {          
            SINGLE_INSTANCE = new LazyInitSingleton();//判断未构造后再构造
          }
          return SINGLE_INSTANCE;
      }
  }
  ```

### 2.2 Lazy Initialization多线程版

在多线程下，可能多个线程在较短时间内一同调用 getInstance()方法，且判断

```
SINGLE_INSTANCE == null
```

结果都为true，则*2.1 Lazy* *Initialization单线程版* 会构造多个实例，即单例模式失效

作为修正

#### 2.2.1 synchronized关键字

**第一版**

- 可考虑使用synchronized关键字同步获取方法

  ```
  public class  LazyInitSingleton {
  
      private static  LazyInitSingleton SINGLE_INSTANCE = null;
  
      //私有化构造器
      private  LazyInitSingleton() {}
  
      //构造实例，加入synchronized关键字
      public static synchronized LazyInitSingleton getInstance() {
          if (SINGLE_INSTANCE == null) {          
            SINGLE_INSTANCE = new  LazyInitSingleton();//判断未构造后再构造
          }
          return SINGLE_INSTANCE;
      }
  }
  ```

#### 2.2.2 synchronized关键字

**第二版（double checked locking 二次判断锁）**

以上可实现线程安全，但由于使用了synchronized关键字实现锁定控制，getInstance()方法性能下降，造成瓶颈。分析到需求构建操作只限于未构建判断后第一次调用getInstance()方法，即构建为低频操作，所以完全可以在判断已经构建后直接返回，而不需要使用锁。

- 仅在判断需要构建后才进行锁定：

  ```
  public class  LazyInitSingleton {
  
      private static  LazyInitSingleton SINGLE_INSTANCE = null;
  
      //私有化构造器
      private  LazyInitSingleton() {}
  
      //构造实例
      public static synchronized   LazyInitSingleton getInstance() {
          if (SINGLE_INSTANCE == null) { 
              synchronized(LazyInitSingleton.class){
                  if (SINGLE_INSTANCE == null) {
                      SINGLE_INSTANCE = new  LazyInitSingleton();//判断未构造后再构造
                  }
              }
          }
          return SINGLE_INSTANCE;
      }
  }
  ```

  

## 3. 利用静态内部类实现懒加载

JVM仅在使用时加载静态资源，当类加载时，静态内部类不会加载，仅当静态内部类在使用时会被加载，且实现顺序初始化即加载是线程安全的，利用这一性质，我们可以实现懒加载。

- 代码如下：

  ```
  public class NestedSingleton {
  
      private NestedSingleton() {}
      //静态内部类，只初始化一次
      private static class SingletonClassHolder {
          static final NestedSingleton SINGLE_INSTANCE = new NestedSingleton();
      }
      //调用静态内部类方法得到单例
      public static NestedSingleton getInstance() {
          return SingletonClassHolder.SINGLE_INSTANCE;
      }
  
  }
  ```

## 4. 使用Enum

由于枚举类是线程安全的，可以直接使用枚举创建单例。但考虑到枚举无法继承，所以只在特定情况下使用。

- 代码如下：

  ```
  public enum EnumSingleton {
      INSTANCE;
  }
  ```

  

## 附1 利用反射机制破解单例

利用反射机制破解单例（非Enum形式的单例，）

- 单例形式的类可被反射破解，从而使用单例失效，即将其Private构造器，通过反射形式暴露出来，并进行实例的构造

  ```
  public static void main(String[] args) {
          
          NestedSingleton nestedSingleton =  NestedSingleton.getInstance();
          NestedSingleton nestedSingleton2 = null;
          try {
              //暴露构造器
              Constructor[] constructors = nestedSingleton.getClass().getDeclaredConstructors();
              Constructor constructor = constructors[1];
              constructor.setAccessible(true);
              nestedSingleton2 = (NestedSingleton)constructor.newInstance();
              
          } catch (Exception e ) {
              // TODO Auto-generated catch block
              e.printStackTrace();
          } 
          System.out.println(nestedSingleton.hashCode());
          System.out.println(nestedSingleton2.hashCode());
      }
  }
  ```

- 为防止以上情况，可在构造器中抛出异常，以阻止新的实例产生

  ```
  public class NestedSingleton {
      
  
      private NestedSingleton() {
          synchronized (NestedSingleton.class) {
              //判断是否已有实例
          if(SingletonClassHolder.SINGLE_INSTANCE  != null){
              throw new RuntimeException("new another instance!");
          }
      }
  
  }
  //静态内部类，只初始化一次
      private static class SingletonClassHolder {
          static final NestedSingleton SINGLE_INSTANCE = new NestedSingleton();
          
      }
  //调用静态内部类方法得到单例
      public static NestedSingleton getInstance() {
          return SingletonClassHolder.SINGLE_INSTANCE;
      }
  }
  ```

  

## 附2 序列化导致的单例失效

- 序列化(Serialization)导致的单例失效实例

  ```
  import java.io.FileInputStream;
  import java.io.FileOutputStream;
  import java.io.IOException;
  import java.io.ObjectInput;
  import java.io.ObjectInputStream;
  import java.io.ObjectOutput;
  import java.io.ObjectOutputStream;
  import java.io.OutputStream;
  
  public class SingletonSerializedTest   {
  
      public static void main(String[] args) throws Exception {
          
          NestedSingleton nestedSingleton0 = NestedSingleton.getInstance();
          ObjectOutput output =null;
          OutputStream outputStream = new FileOutputStream("serializedFile.ser"); 
          output = new ObjectOutputStream(outputStream);
          output.writeObject(nestedSingleton0);
          output.close();
      
          ObjectInput input = new ObjectInputStream(new FileInputStream("serializedFile.ser"));
          NestedSingleton nestedSingleton1 = (NestedSingleton) input.readObject();
          input.close();
           
          System.out.println("nestedSingleton0 hashCode="+nestedSingleton0.hashCode());
          System.out.println("nestedSingleton1 hashCode="+nestedSingleton1.hashCode());
      
      
      }
  }
  ```

  输出结果

  ```
  nestedSingleton0 hashCode=865113938
  nestedSingleton1 hashCode=2003749087
  ```

  显然，产生了两个实例

  如果要避免这个情况，则需要利用ObjectInputStream.readObject()中的机制，它在调用readOrdinaryObject()后会判断类中是否有ReadResolve()方法，如果有就采用类中的ReadResolve()新建实例

  那么以上单例类就可以如下改造

  ```
  import java.io.Serializable;
  
  public class NestedSingleton implements Serializable {
      
      /**
       * 
       */
      private static final long serialVersionUID = 3934012982375502226L;
      private NestedSingleton() {
          synchronized (NestedSingleton.class) {
              //判断是否已有实例
          if(SingletonClassHolder.SINGLE_INSTANCE  != null){
              throw new RuntimeException("new another instance!");
          }
      }
  }
  //静态内部类，只初始化一次
      private static class SingletonClassHolder {
          static final NestedSingleton SINGLE_INSTANCE = new NestedSingleton();   
      }
  //调用静态内部类方法得到单例
      public static NestedSingleton getInstance() {
          return SingletonClassHolder.SINGLE_INSTANCE;
      }
      public void printFn() {
          // TODO Auto-generated method stub
          System.out.print("fine");
      }
       protected Object readResolve() {
          return getInstance();
      } 
  }
  ```

  再次使用序列化反序列化过程验证，得到

  ```
  nestedSingleton0 hashCode=865113938
  nestedSingleton1 hashCode=865113938
  ```

  这样在序列化反序列化过程保证了单例的实现



## 参考

- https://www.cnblogs.com/codflow/p/10105903.html