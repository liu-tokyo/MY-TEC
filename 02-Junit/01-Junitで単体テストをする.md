# Junitで単体テストをする

## 目次

- [Junitで単体テストをする](#junitで単体テストをする)
  - [目次](#目次)
- [環境](#環境)
- [目標](#目標)
- [Junitとは](#junitとは)
- [Junitによるテストの注意点](#junitによるテストの注意点)
- [テスト対象のクラスを作成する。](#テスト対象のクラスを作成する)
- [テストコードを作成する。](#テストコードを作成する)
- [テストコードに記載すること。](#テストコードに記載すること)
- [テストコードを完成させる。](#テストコードを完成させる)
- [参考文献](#参考文献)

---

## 1. 環境

・windows10
・jdk 1.8.0_201
・Eclipse Version: 2018-12 (4.10.0)
・Junit_4.12.0

## 2. 目標

javaで作成したクラスの単体テストが実行できるようになること。

## 3. Junitとは

xUnitなるテスティングフレームワークのJava版で、作成したクラスに対して
作成したJunitのコードからテスト対象クラスのメソッドを呼び出して動作をテストできる。

一度作成したテストコードは流用が効くため、毎回テストコードを作成する手法に比べて
テストにかかる工数を削減することができる。

## 4. Junitによるテストの注意点

1.publicメソッドであること。
2.戻り値はvoidであること。
3.引数を持たないこと。
4.テストメソッドにはorg.junit.Testアノテーションを付与すること。

## 5. テスト対象のクラスを作成する。

今回は乗算と除算をするクラスCalculator.javaを作成した。

Calculator.java

```
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

## 6. テストコードを作成する。

次にテストクラスを作成する。
新規にクラスを作成してtestと入力してctrl + spaceと入力すると
Eclipseのコンテンツアシストが起動するので、Junit4用のテンプレートを選択すると
簡単にメソッドを作成出来る。

まずは何も入れない状態のテストクラスCalculatorTestクラスを作成する。

CalculatorTest.java

```
package junit.tutorial;

public class CalculatorTest {
}
```

ここに上記コンテンツアシストを使ってテンプレートメソッドを挿入する。

CalculatorTest.java

```
package junit.tutorial;

import static org.junit.Assert.*;

import org.junit.Test;

public class CalculatorTest {
    @Test
    public void testName() throws Exception {

    }
}
```

## 7. テストコードに記載すること。

今回は以下のようにテストコードを作成していく。
最終的に3の結果を確認することで、試験が想定通りに動作したかを判定できる。

1.メソッドで使用する入力値をセットする。
2.メソッドの戻り値の期待値をセットする。
3.期待値と1の戻り値である実測値とをアサーションする。

## 8. テストコードを完成させる。

早速テストコードを完成させる。
メソッド名は日本語とした。
これにより実行結果が何を検証したか一目でわかる。

Calculator.java

```
package junit.tutorial;

import static org.hamcrest.CoreMatchers.*;
import static org.junit.Assert.*;

import org.junit.Test;

public class CalculatorTest {

    @Test
    public void maltiplicationで5と2の乗算結果が取得できる() throws Exception {
        Calculator sut = new Calculator();
        int expected = 10;
        int actual = sut.maltiplication(5, 2);
        assertThat(actual,is(expected));
    }

    @Test
    public void divisionで5と8の除算結果が取得できる() throws Exception {
        Calculator sut = new Calculator();
        int expected = 

    }
}
```

テストコードの期待値を設定している最中に戻り値がintでは除算の結果としてよろしくないことに気が付く。
この事実を受けてテスト対象のクラスを修正する。

Calculator.java

```
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
    public float division(int x,int y) {
        return (float)x / (float)y;
    }
}
```

戻り値の型をfloatにして、returnの値をキャストした。
ではテストコード作成に戻る。

CalculatorTest.java

```
package junit.tutorial;

import static org.hamcrest.CoreMatchers.*;
import static org.junit.Assert.*;

import org.junit.Test;

public class CalculatorTest {

    @Test
    public void maltiplicationで5と2の乗算結果が取得できる() throws Exception {
        Calculator sut = new Calculator();
        int expected = 10;
        int actual = sut.maltiplication(5, 2);
        assertThat(actual,is(expected));
    }

    @Test
    public void divisionで5と8の除算結果が取得できる() throws Exception {
        Calculator sut = new Calculator();
        float expected = 0.625f;
        float actual =  sut.division(5, 8);
        assertThat(actual,is(expected));

    }
}
```

これでテストが実行できるようになった。
ではEclipseでJunitとしてテスト実行して結果を確認してみる。

[![image1.png](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F360148%2F793de725-9b3a-5902-d0fd-0c99e1fd95a4.png?ixlib=rb-4.0.0&auto=format&gif-q=60&q=75&s=76ff763db29d62419f280e9b4641bfac)](https://camo.qiitausercontent.com/d8d42c9eabc9da18f7dbb3c10dabe8a5a7dff11e/68747470733a2f2f71696974612d696d6167652d73746f72652e73332e616d617a6f6e6177732e636f6d2f302f3336303134382f37393364653732352d396233612d353930322d643066642d3063393965316664393561342e706e67)

・画面上でインジゲータがすべて緑色になっていること。
・実行結果が2/2で完了していること。

上記の結果から期待値通りにメソッドが動作したことがわかった。

これがJunitの基本操作となる。

Junitを使った試験の概要についてもキチンと勉強したいと思う。

## 参考文献

[JUnit実践入門 ── 体系的に学ぶユニットテストの技法 WEB+DB PRESS plus]
https://www.amazon.co.jp/dp/B07JL78S95/ref=dp-kindle-redirect?_encoding=UTF8&btkr=1