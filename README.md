## 工作中相关技术的点点滴滴



### 在线教程

- 菜鸟：

  https://www.runoob.com/

- vue5.com：
  http://www.vue5.com/


```
https://profile.oracle.com/myprofile/account/verify.jspx?key=6B26942E9B1D9BE897AC5B74B1D65F591E3172C170B57FFE0DCB9E81012D6BFDFBF607B7830C82D7CF75DD26D417304F014E3B160AC8B295B2DACA016EF2F299
```



### OJDBC版本关系

https://docs.oracle.com/en/database/oracle/oracle-database/21/jjdbc/JDBC-getting-started.html#GUID-926E5324-D89A-4A00-B1AE-975C1089F0EA



## IBM MQ 官方下载地址

- https://public.dhe.ibm.com/ibmdl/export/pub/software/websphere/messaging/mqadv/



```
xcopy /D /E /C /I /H /R /K /Y C:\sample F:\sample
```



```
<dependency>
  <groupId>org.springframework.data</groupId>
  <artifactId>spring-data-neo4j-tx</artifactId>
  <version>2.2.0.RELEASE</version>
</dependency>
```



https://github.com/spring-attic/toolsuite-distribution/wiki/Spring-Tool-Suite-3

https://stackoverflow.com/questions/22753380/rollback-or-time-out-for-jmstemplate-send



## 対策

次の環境変数を設定する。

- Windowsの場合
  - /path/to/gradle/bin/gradle.bat
- その他の場合(Git Bashなどを使う時でも)
  - /path/to/gradle/bin/gradle

```
set DEFAULT_JVM_OPTS="-Dfile.encoding=UTF-8"
```



```
apply plugin: 'jacoco'

def coverageSourceDirs = [
        "${rootDir.absolutePath}/app/src",
        "${rootDir.absolutePath}/submodule_1/src",
        "${rootDir.absolutePath}/submodule_2/src",
]

def coverageClassDirs = [
        fileTree(dir: "${rootDir.absolutePath}/app/build/intermediates/javac/SNMAPP__10009Debug/compileSNMAPP__10009DebugJavaWithJavac/classes", excludes: androidExclusion),
        fileTree(dir: "${rootDir.absolutePath}/submodule_1/build/intermediates/javac/debug/compileDebugJavaWithJavac/classes", excludes: androidExclusion),
        fileTree(dir: "${rootDir.absolutePath}/submodule_2/build/intermediates/javac/debug/compileDebugJavaWithJavac/classes", excludes: androidExclusion),
]

task jacocoTestReport_test(type: JacocoReport) {
    group = "Reporting"
    description = "Generate Jacoco coverage reports runing tests."

    reports {
        xml.enabled = true
        html.enabled = true
        html.destination file("${rootDir.absolutePath}/app/build/reports/jacoco")
    }
    sourceDirectories = files(coverageSourceDirs)
    classDirectories = files(coverageClassDirs)
    executionData = files("${rootDir.absolutePath}/app/build/outputs/code_coverage/SNMAPP__10009DebugAndroidTest/connected/coverage.exec")
}
```



## 常见Bean映射工具的性能比对

https://www.cnblogs.com/javaguide/p/11861749.html
