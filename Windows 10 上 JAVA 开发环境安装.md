# Windows 10 上 JAVA 开发环境安装



## JDK 安装

- **官方下载**

  https://www.oracle.com/technetwork/java/javase/downloads/index.html

  下载合适的版本，默认安装即可。安装路径：`C:\Program Files\Java\jdk-19`

- **环境变量**

  1. 新建 **系统变量**：

     变量名：`JAVA_HOME`  
     变量值：`C:\Program Files\Java\jdk-19`

  2. 设置 系统变量 的 Path，追加如下项目：  
     `%JAVA_HOME%\bin`  
     `%JAVA_HOME%\jre\bin`

  3. 和JAVA_HOME一样，新建一个名为“CLASSPATH”的环境变量（如果已经存在该变量名的话，数值追加即可）：  
     变量名：`CLASSPATH`  
     变量值：`%JAVA_HOME%\lib\dt.jar;%JAVA_HOME%\lib\tools.jar`  
     注意：安装最新的JDK9时，在lib中并没有发现tools.jar和rt.jar，不设置本步骤，在后来的程序中也可以正常执行。应该是在JDK9中已经将Classpath这一步包含在了第一步，以节省配置过程的复杂程度。

- **版本查询**

  在配置好环境变量后，可以进入cmd中检查Java是否安装正确，检查的命令为：

  ```
  java -version
  ```

  本人安装的版本信息如下：

  ```
  C:\Users\liu>java -version
  java version "19.0.1" 2022-10-18
  Java(TM) SE Runtime Environment (build 19.0.1+10-21)
  Java HotSpot(TM) 64-Bit Server VM (build 19.0.1+10-21, mixed mode, sharing)
  ```

  

## Tomcat 安装

- **官网下载**

  http://tomcat.apache.org/

  我个人下载的是 `9.0.70` 版本，没有用最新版本，因为当前开发环境用的是 `9.0.40` 版本。

  加压后放置在 C 盘根目录下 `C:\apache-tomcat-9.0.70` 。

- **目录简介**

  | 目录名  | 说明                                                         | 备注 |
  | ------- | ------------------------------------------------------------ | ---- |
  | bin     | 主要是用来存放tomcat的命令，主要有两大类，一类是以.sh结尾的（linux命令），另一类是以.bat结尾的（windows命令）。 |      |
  | conf    | 主要是用来存放tomcat的一些配置文件。                         |      |
  | lib     | 主要用来存放tomcat运行需要加载的jar包。                      |      |
  | logs    | 用来存放tomcat在运行过程中产生的日志文件。                   |      |
  | temp    | 用户存放tomcat在运行过程中产生的临时文件。                   |      |
  | webapps | 用来存放应用程序，当tomcat启动时会去加载webapps目录下的应用程序。 |      |
  | work    | 用来存放tomcat在运行时的编译后文件。                         |      |

- **环境变量**

  １．新建 **系统变量**

  - 变量名：`CATALINA_HOME`
  - 变量值：`C:\apache-tomcat-9.0.70` （个人真实的安装目录）

  ２．配置 Path 系统变量

  - 追加：`%CATALINA_HOME%\bin`

- **检测配置**

  打开命令提示符，输入： `startup` 。启动Tomcat 后可以在浏览器中输入： `localhost:8080` ，出现一个小猫的界面。如果端口8080被占用，可以改为其它端口。

- **解决乱码**

  **乱码原因**：这是由于Windows下的cmd的默认编码是GBK编码，Tomcat控制台默认输出设置为`UTF-8`编码。

  修改文件：`\conf\logging.properties`

  ```
  #java.util.logging.ConsoleHandler.encoding = UTF-8
  java.util.logging.ConsoleHandler.encoding = GBK
  ```

  ※ 当然，可以修改`CMD`命令行的编码格式为UTF-8，为了不影响其它程序，修改 `Tomcat` 的输出日志编码最合适。

## Gradle 安装

- **官网下载**

  https://gradle.org/gradle-download/

  解压之后，放在 C 盘根目录下：`C:\gradle-7.6` 。

- **环境变量**

  - 新建 **系统变量**

    变量名：`GRADLE_HOME`

    变量值：`C:\gradle-7.6`

  -  系统变量 Path 编辑

    追加内容：`%GRADLE_HOME%\bin`

- **动作确认**

  确认版本

  ```
  gradle -version
  ```

  

## STS 安装

- 官方下载

  https://spring.io/tools

  下载的是自动解压版本，双击之后自动解压；也可以从下面位置下载 ZIP 版本：

  https://github.com/spring-projects/sts4/wiki/Previous-Versions

- 其它参照

  https://blog.51cto.com/tenghui/5253776

- 其它说明

  STS是一个比较综合的开发环境，自带的 JRE，其它内容不知道是否能够省略。

