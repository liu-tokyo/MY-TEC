# 简易的 Junit 测试程序

程序简单，仅仅是对 Junit 进行最初步的认识。

---

## 测试环境

- openjdk version "11.0.14.1" 2022-02-08 LTS
- IntelliJ IDEA 2021.3 （最新版本）
- Junit 4.1.3
- hamcrest-core-1.3 （感觉好像是没有用）



## 注意事项

使用 Junit 进行测试的注意事项：

1. 必须是公共方法。
2. 返回值无效。
3. 它没有论据。
4. 用org.junit.Test注释测试方法。



## 测试对象

- **Calculator.java**

  ```java
  package junit.tutorial;
  
  public class Calculator {
  
      /**
       * xとyの乗算結果を返す
       * @param x
       * @param y
       * @return xとyの乗算結果を返す
       */
      public int maltiplication(int x,int y) {
          return x * y;
      }
  
      /**
       * xとyの除算結果を戻す
       * @param x
       * @param y
       * @return xとyの除算結果を戻す
       */
      public int division(int x,int y) {
          return x / y;
      }
  }
  ```



## 测试代码

- **CalculatorTest.java**

  ```java
  package junit.tutorial;
  
  //import static org.hamcrest.CoreMatchers.*;
  import static org.junit.Assert.*;
  
  import org.junit.Test;
  
  public class CalculatorTest {
  
      @Test
      public void test_multiplication() {
          Calculator sut = new Calculator();
          int expected = 10;
          int actual = sut.maltiplication(5, 2);
          //assertThat(actual,is(expected));  // 已经过时
          assertSame(actual, expected);
      }
  
      @Test
      public void test_division() {
          Calculator sut = new Calculator();
          int expected = 2;
          int actual = sut.division(5, 2);
          assertSame(actual, expected);
      }
  }



## 残留问题

引入 **Junit-4.1.3** 一切正常，但是在引入 **Junit-5.8.1** 的情况下，无法编译通过。

- **CalculatorTest.java** - Junit-5.8.1

  ```java
  package junit.tutorial;
  
  import org.junit.jupiter.api.Test;
  
  import static org.junit.jupiter.api.Assertions.assertSame;
  
  public class CalculatorTest {
  
      @Test
      public void test_multiplication() {
          Calculator sut = new Calculator();
          int expected = 10;
          int actual = sut.maltiplication(5, 2);
          //assertThat(actual,is(expected));  // 已经过时
          assertSame(actual, expected);
      }
  
      @Test
      public void test_division() {
          Calculator sut = new Calculator();
          int expected = 2;
          int actual = sut.division(5, 2);
          assertSame(actual, expected);
      }
  }
  ```

  

## 参照资料

- https://qiita.com/ryuutamaehara/items/c8efb304b73cc0542e6f
