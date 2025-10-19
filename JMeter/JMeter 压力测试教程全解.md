# JMeter 压力测试教程全解

## 安装

- 官方网址

  http://jmeter.apache.org/download_jmeter.cgi

## 介绍
### 一、背景简介

测试人员有时需要分析系统在繁重的负载的整体性能，发现系统中的瓶颈问题，减少发布到生产环境后出问题的几率；预估系统的承载能力，使我们能根据其做出一些应对措施。文档简单介绍一款压力测试工具JMeter。

### 二、简介与安装

#### 2.1 JMeter简介

Apache JMeter是Apache组织开发的基于Java的压力测试工具。用于对软件做压力测试，它最初被设计用于Web应用测试但后来扩展到其他测试领域。 它可以用于测试静态和动态资源例如静态文件、Java 小服务程序、CGI 脚本、Java 对象、数据库， FTP 服务器，等等。JMeter 可以用于对服务器、网络或对象模拟巨大的负载，来自不同压力类别下测试它们的强度和分析整体性能。

#### 2.2 JMeter安装

1）JMeter是使用JAVA写的，所以使用JMeter之前，先安装JAVA环境。此处略过。

2）JMeter可以从Apache官网下载：

https://jmeter.apache.org/download_jmeter.cgi。

下载后解压即可使用。可以将${JMETER_HOME}/bin配置到环境变量中，避免每次都用绝对路径去启动JMeter.

Windows下，双击bin目录下的ApacheJMeter.jar即可用图形化界面操作配置测试任务。

![image-20221227183911476](C:\Users\KST_LIUZHIJUN\AppData\Roaming\Typora\typora-user-images\image-20221227183911476.png)

![image-20221227183951704](C:\Users\KST_LIUZHIJUN\AppData\Roaming\Typora\typora-user-images\image-20221227183951704.png)

![image-20221227184003964](C:\Users\KST_LIUZHIJUN\AppData\Roaming\Typora\typora-user-images\image-20221227184003964.png)

- Linux命令行下，则需要准备好jmx测试脚本，调用如下命令

  ```
  ${JMETER_HOME}/bin/jmeter.sh -n -t Test.jmx -l testReport.jtl
  ```

  其中，${JMETER_HOME}是JMeter的解压目录，-n表示非图形化界面运行，-t指向jmx测试脚本，-l指定压测后产生的日志文件名。

- jmx测试脚本怎么编写？

  可以先在windows下用图形化界面配置好测试任务，点击文件，保存测试计划为，选择保存路径。在该路径下即可获取相应测试脚本。

  ![image-20221227184226721](C:\Users\KST_LIUZHIJUN\AppData\Roaming\Typora\typora-user-images\image-20221227184226721.png)

  Jmx测试脚本其实是一个xml文件，可以在已有的脚本中修改配置，不一定每次都在windows下生成。例如，多少秒内启动的多少线程，修改如下两个值即可，以下表示的是1秒内启动50个线程执行测试。

  ![image-20221227184306772](C:\Users\KST_LIUZHIJUN\AppData\Roaming\Typora\typora-user-images\image-20221227184306772.png)

  命令行生成的测试报告 `testReport.jtl` 可以在windows下用图形界面打开：

  ![image-20221227184315087](C:\Users\KST_LIUZHIJUN\AppData\Roaming\Typora\typora-user-images\image-20221227184315087.png)

  ![image-20221227184339737](C:\Users\KST_LIUZHIJUN\AppData\Roaming\Typora\typora-user-images\image-20221227184339737.png)

### 三、JMeter使用

下面结合某次测试介绍JMeter的简单使用。

- 添加测试计划

  ![image-20221227184423200](C:\Users\KST_LIUZHIJUN\AppData\Roaming\Typora\typora-user-images\image-20221227184423200.png)

  ![image-20221227184436462](C:\Users\KST_LIUZHIJUN\AppData\Roaming\Typora\typora-user-images\image-20221227184436462.png)

- 设置线程数（线程数可以任务是并发用户数）

  ![image-20221227184451224](C:\Users\KST_LIUZHIJUN\AppData\Roaming\Typora\typora-user-images\image-20221227184451224.png)

  上述配置的含义：在10秒内，启动50个线程，循环20次进行压力测试。

  关于Ramp-up period的说明：
  
  - （1）决定多长时间启动所有线程。如果使用50个线程，ramp-up period是10秒，那么JMeter用10秒使所有50个线程启动并运行。每个线程会在上一个线程启动后0.2秒（10/50）启动。Ramp-up需要要充足长以避免在启动测试时有一个太大的工作负载，并且要充足小以至于最后一个线程在第一个完成前启动。 一般设置ramp-up=线程数启动，并上下调整到所需的。
  - （2）用于告知JMeter 要在多长时间内建立全部的线程。默认值是0。如果未指定ramp-up period ，也就是说ramp-up period 为零， JMeter 将立即建立所有线程。假设ramp-up period 设置成T 秒， 全部线程数设置成N个， JMeter 将每隔T/N秒建立一个线程。
  - （3）Ramp-Up Period(in-seconds)代表隔多长时间执行，0代表同时并发。
  
  可以选择永远循环
  
  ![image-20221227184722735](C:\Users\KST_LIUZHIJUN\AppData\Roaming\Typora\typora-user-images\image-20221227184722735.png)

- 3、设置测试接口

  本样例的测试接口是：用http协议给远程发送参数与图片。右击线程组，选择“添加”、“Sampler”、“http请求”

  ![image-20221227184828115](C:\Users\KST_LIUZHIJUN\AppData\Roaming\Typora\typora-user-images\image-20221227184828115.png)

  ![image-20221227184838645](C:\Users\KST_LIUZHIJUN\AppData\Roaming\Typora\typora-user-images\image-20221227184838645.png)

  ![image-20221227184852923](C:\Users\KST_LIUZHIJUN\AppData\Roaming\Typora\typora-user-images\image-20221227184852923.png)

- 4、一般需要查看调用结果，则需添加“察看结果树”

  ![image-20221227184906799](C:\Users\KST_LIUZHIJUN\AppData\Roaming\Typora\typora-user-images\image-20221227184906799.png)

- 5、执行测试计划

  点击绿色右三角，即可执行测试计划。可能会弹框提醒是否保存测试计划，选择“是”，则需要选择保存路径，选择“否”，则直接执行测试计划。如果中途需停止测试，点击“stop”即可。

  ![image-20221227184940480](C:\Users\KST_LIUZHIJUN\AppData\Roaming\Typora\typora-user-images\image-20221227184940480.png)

  ![image-20221227184948281](C:\Users\KST_LIUZHIJUN\AppData\Roaming\Typora\typora-user-images\image-20221227184948281.png)



- 6、查看测试结果

  调用结束后，可以通过设置的“察看结果树”，“响应数据”看到接口返回结果.。例如：本样例接口，如果调用成功则返回“ok”

  ![image-20221227185021071](C:\Users\KST_LIUZHIJUN\AppData\Roaming\Typora\typora-user-images\image-20221227185021071.png)

  查看发出的请求：
  
  ![image-20221227185054686](C:\Users\KST_LIUZHIJUN\AppData\Roaming\Typora\typora-user-images\image-20221227185054686.png)

- 7、查看测试的统计报告：统计失败率、吞吐量、调用次数等

  可以通过添加“聚合报告”实现

  ![image-20221227185129465](C:\Users\KST_LIUZHIJUN\AppData\Roaming\Typora\typora-user-images\image-20221227185129465.png)

  点击绿色右三角，开始执行任务后，点击“聚合报告”，可以查看测试统计：
  
  ![image-20221227185155870](C:\Users\KST_LIUZHIJUN\AppData\Roaming\Typora\typora-user-images\image-20221227185155870.png)

  1)Label - 请求对应的name属性值。
  
  2)Samples - 具有相同标号的样本数，总的发出请求数。

  3)Average - 请求的平均响应时间。

  4)Median - 50%的样本都没有超过这个时间。这个值是指把所有数据按由小到大将其排列，就是排列在第50%的值。

  5)90% Line - 90%的样本都没有超过这个时间。这个值是指把所有数据按由小到大将其排列，就是排列在第90%的值。

  6)95% Line - 95%的样本都没有超过这个时间。这个值是指把所有数据按由小到大将其排列，就是排列在第95%的值。

  7)99% Line - 99%的样本都没有超过这个时间。这个值是指把所有数据按由小到大将其排列，就是排列在第99%的值。

  8)Min - 最小响应时间。

  9)Max - 最大响应时间。

  10)Error % - 本次测试中，有错误请求的百分比。

  11)Throughput - 吞吐量是以每秒/分钟/小时的请求量来度量的。这里表示每秒完成的请求数。

  12)Received KB/sec - 收到的千字节每秒的吞吐量测试。

  13)Sent KB/sec - 发送的千字节每秒的吞吐量测试。

8、读取csv文件

本测试样例，需要多线程读取多个图片文件。实现方式：按行将所有图片文件的路径放在csv文件中，用JMeter去读取。

右击线程组，添加，配置原件，CSV Data Set Config

1）Filename: CSV文件路径。

2）File encoding: CSV文件编码，可选UTF8。默认为ANSI

3）Variable Names: 给csv文件中各列起个名字（有多列时，用英文逗号隔开列名）便于后面引用

4）Delimiter:与 .csv文件的分隔符保持一致。如文件中使用的是逗号分隔，则填写逗号；如使用的是TAB，则填写\t；

5）Allow quoted data?:不知道。

6）Recycle on EOF:到了文件尾是否循环，True—继续从文件第一行开始读取，False—不再循环

7）Stop thread on EOF :到了文件尾是否停止线程，True—停止，False—不停止，注：当Recycle on EOF设置为True时，此项设置无效。

8）Sharing mode:共享模式，All threads –所有线程，Current thread group—当前线程组，Current thread—当前线程。


这里CSV文件只有一列，我们设置该列变量名为picFile，并在后面引用即可把图片路径作为变量传到请求中：

9、自定义函数处理特定操作

本样例需要在每次将图片发出请求后，删除该本地图片。可以通过添加Bean Shell Sampler实现。


编写java程序：

10、添加随机数变量


将表达式复制到需要用随机数的地方。



基本设置:
参考：https://blog.csdn.net/weixin_42350428/article/category/8003666

使用Jmeter进行RPC压力测试
java Request主要机制是：实现Jmeter自带接口完成java Request请求，具体操作如下：

实现Jmeter自带接口

1.打开Java编译器,我使用的是eclipse, 新建一个项目"TestLength",然后新建一个包"app".

2.从Jmeter的安装目录lib\ext中拷贝两个文件"ApacheJMeter_core.jar"和"ApacheJMeter_java.jar"到"Tester"的项目中,然后引入这两个JAR文件.(具体的引入方法参考各个Java编译器的使用方法)

3.在"app"包中新建一个类,名字叫"TestLength",不过这个类要继承"AbstractJavaSamplerClient"类,如果项目引入步骤二中的两个文件,就可以找到"AbstractJavaSamplerClient"类了.

4.“TestLength"类在继承"AbstractJavaSamplerClient"类的同时也会继承四个方法,分别是"getDefaultParameters”,“setupTest”,"runTest"和"teardownTest"方法."getDefaultParameters"方法主要用于设置传入的参数;"setupTest"方法为初始化方法,用于初始化性能测试时的每个线程."runTest"方法为性能测试时的线程运行体;"teardownTest"方法为测试结束方法,用于结束性能测试中的每个线程.

5.具体代码如下：

package app;

import org.apache.jmeter.config.Arguments;
import org.apache.jmeter.protocol.java.sampler.AbstractJavaSamplerClient;
import org.apache.jmeter.protocol.java.sampler.JavaSamplerContext;
import org.apache.jmeter.samplers.SampleResult;

public class PerformenceTest extends AbstractJavaSamplerClient {
	 private SampleResult results;
     private String testStr;

    //设置传入的参数，可以设置多个，已设置的参数会显示到Jmeter的参数列表中
    @Override
    public Arguments getDefaultParameters() {
    	Arguments params = new Arguments();
        //定义一个参数，显示到Jmeter的参数列表中，第一个参数为参数默认的现实名称，第二个参数为默认值
        params.addArgument("testStr","abc");
        return params;
    }
    //测试前工作
    @Override
    public void setupTest(JavaSamplerContext context) {
    	results = new SampleResult();
        testStr=context.getParameter("testStr","");
        if(testStr!=null && testStr.length()>0){
                results.setSamplerData(testStr);
        }
    }


    // 测试执行的循环体，根据线程数和循环次数的不同可执行多次，类似于LoadRunner中的Action方法
    @Override
    public SampleResult runTest(JavaSamplerContext context) {
    	 int len=0;
         //定义一个事务，表示这是事务的起始点，类似于LoadRunner的lr.start_transaction
         results.sampleStart();
         testStr = context.getParameter("testStr");
         len = testStr.length();
         //定义一个事务，表示这是事务的结束点，类似于LoadRunner的lr.end_transaction
         results.sampleEnd();
         results.setDataEncoding("utf-8");
         if(len<5){
                  System.out.println(testStr);
                 results.setResponseCode("testStr."+testStr);
                  //用于设置运行结果的成功或失败，如果是false表示失败，true表示成功。
                  results.setSuccessful(false);
         }else{
                  results.setResponseCode("testStr."+testStr);
                  results.setSuccessful(true);
         }
         return results;
    }
    //测试后工作
    public void teardownTest(JavaSamplerContext context) {
        super.teardownTest(context);
    }

  //Jmeter不执行main函数。
    public static void main(String[] args) {
        Arguments params = new Arguments();
        params.addArgument("testStr", "1111");//设置参数，并赋予默认值 
        JavaSamplerContext arg0 = new JavaSamplerContext(params);
        PerformenceTest test = new PerformenceTest();
        test.setupTest(arg0);
        test.runTest(arg0);
        test.teardownTest(arg0);
        System.exit(0);
    }


}
6.把上面的例子打包，然后把生成的"TestLength.jar"文件拷贝到Jmeter的安装目录lib\ext下。

7.运行Jmeter，添加一个线程组，然后在该线程组下面添加一个Java请求(在Sampler中)，在Java请求的类名称中选择咱们刚创建的类"app。TestLength"，在下面参数列表的"testStr"后面输入要测试的字符串，然后添加一个监听器(聚合报告)，设置一下模拟的用户数就可以测试了。如果测试不成功，Jmeter会在它自己个输出框中抛出这个字符串。
————————————————
版权声明：本文为CSDN博主「腾讯数据架构师」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/luanpeng825485697/article/details/83787284