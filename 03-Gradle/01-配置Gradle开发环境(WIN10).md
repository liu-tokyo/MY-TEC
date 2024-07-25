# 配置Gradle开发环境(WIN10)

电脑配置：

- 内存：16GB
- 硬盘：60GB + 30GB
- 使用 Boot 环境，不需要安装 TOMCAT 等内容。

## 1. 安装JDK

- 下载地址  
  https://www.azul.com/downloads/?package=jdk#zulu  
  ※ 使用 Zulu 的 JDK17 版本（2024.07当前最新为 JDK22），因为 Oracle 的 JDK 已经不是免费版本。  
  ※ 使用 ZIP 方式，直接展开即可，当前最新为 `zulu17.52.17-ca-jdk17.0.12-win_x64`。
  
- 配置环境变量  
  `系统变量` 增加 `JAVA_HOME` 变量，数值为 Zulu 的安装路径，本人默认安装在 `C:\app\zulu17` 路径下面。  
  `用户变量` 的 `Path` 项目里面，增加 `%JAVA_HOME%\bin` 的路径。

- 验证JDK环境  
  启动新的 命令提示符，输入 `java -version` 指令，查看 JDK 的版本信息。  

  ```bash
  C:\Users\liu>java -version
  openjdk version "17.0.12" 2024-07-16 LTS
  OpenJDK Runtime Environment Zulu17.52+17-CA (build 17.0.12+7-LTS)
  OpenJDK 64-Bit Server VM Zulu17.52+17-CA (build 17.0.12+7-LTS, mixed mode, sharing)
  ```

## 2. 安装Gradle

- 下载地址  
  https://gradle.org/releases/  
  ※ 使用 `8.2.1` 的二进制版本即可。

- 配置环境变量  
  `系统变量` 增加 `GRADLE_HOME` 变量，数值为 Gradle 的安装路径，本人默认安装在 `C:\app\gradle-8.2.1` 路径下面。  
  `用户变量` 的 `Path` 项目里面，增加 `%GRADLE_HOME%\bin` 的路径。

- 验证Gradle环境  
  启动新的 命令提示符，输入 `gradle -v` 指令，查看 Gradle 的版本信息。  

  ```bash
  C:\Users\liu>gradle -v
  
  ------------------------------------------------------------
  Gradle 8.2.1
  ------------------------------------------------------------
  
  Build time:   2023-07-10 12:12:35 UTC
  Revision:     a38ec64d3c4612da9083cc506a1ccb212afeecaa
  
  Kotlin:       1.8.20
  Groovy:       3.0.17
  Ant:          Apache Ant(TM) version 1.10.13 compiled on January 4 2023
  JVM:          17.0.12 (Azul Systems, Inc. 17.0.12+7-LTS)
  OS:           Windows 10 10.0 amd64
  ```

## 3. 安装IntelliJ IDEA

> 免费版本：IntelliJ IDEA Community Edition

- 下载地址  
  https://www.jetbrains.com/  
  ※ 最新版本即可（2024.07 当前最新版本 `ideaIC-2024.1.4.exe`）。
- 安装  
  默认安装即可，需要一定的时间。
- 确认Gradle全局配置  
  `Customize` → `All setting...` ，在 `Setting` 画面的 `Build, Execution, Deployment` → `Build Tools` → `Gradle` ，右侧画面查看 `General Settings` → `Gradle user home` 。  
  在未配置 `GRADLE_USER_HOME` 为变量名的系统环境变量时，默认值时当前用户目录下的 `.gradle` 目录；否则将读取 `GRADLE_USER_HOME` 变量名的值作为默认值。这项配置在 `IntellJ IDEA` 中的作用如下：
  1. 存放 IntellJ IDEA 自动下载的 gradle 程序包；
  2. 存放项目依赖的 jar 包。

## 4. 创建Gradle项目

> 在不配置 `GRADLE_USER_HOME` 系统环境变量的情况下

- 查看工程配置  
  创建工程之后，在工程名称上鼠标右键，选择 `Open Module Settings` 项目，可以查看该工程的各种设置。  
  `Project Structure` 画面的 `Project Settings` → `Libraries` ，在右侧画面查看 库 的配置信息。

- 配置Maven仓库  
  当项目配置上了本地的Gradle，项目依赖了大量的 jar 包，不过这些依赖有很多都是在 Maven 仓库中存在，那么怎么让 Gradle 使用本地的 Maven 仓库呢？  
  在 GRADLE_HOME 的 init.d 目录下面创建一个 init.gradle 文件，配置如下：

  ```groovy
  allprojects {
  	repositories {
  		// 这里file后面的地址是自己本地Maven仓库的目录地址
  		maven { url "file:///D:/Program Files/Apache/repository"}
  		mavenLocal()
  		maven { url "https://repo.spring.io/release"}
  		maven { url "https://repository.jboss.org/maven2"}
          maven { url "maven { url "https://repository.jboss.org/maven2"}"}
  		mavenCentral()
  		//google()
  		//maven { url "http://repo.mycompany.com/maven2"}
  	}
  	buildscript {
  		repositories {
  			mavenCentral()
              //google()
  		}
  		dependencies {
  		}
  	}
  }
  ```

  当然上面的配置也可以针对牧歌项目的 `build.gradle` 去配置，但是配置在 `GRADLE_HOME` 目录下面时表示全局配置。  
  配置完成之后，重启 `IntellJ IDEA` ，以后 Gradle 项目若使用的是本地 `GRADLE_HOME` 目录的 gradle 程序，那么就会按照上面的配置来加载依赖包。

- 仓库配置区别

  1. `buildScript`块的 `repositories` 主要为了 Gradle 剧本自身的执行，获取脚本依赖插件。

  2. `根`级别的 `repositories` 主要为了当前项目提供所需依赖包，比如 `log4j`、`spring-core` 等依赖包可以从 `mavenCentral` 仓库获得。

  3. `allprojects` 的 `repositories` 用于多项目构建，为所有项目提供共同所需依赖包。

     而子项目可以配置自己的 `repositories` 以获取自己独自需要的依赖包。

## 5. SpringBoot 项目

- 开始URL  
  https://start.spring.io/