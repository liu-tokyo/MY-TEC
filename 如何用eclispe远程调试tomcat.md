# 如何用eclispe远程调试tomcat

tomcat是一种非常常见的java web应用服务器，有时候服务器可能并不是部署在本地，而是部署在远程其他的机器上，我们用eclispe该如何进行debug调试呢？下面小编就和大家分享一下解决的办法。

1. 在eclispe中新建web应用，名字叫webtest。里面只有一个HelloServlet。Web.xml配置如下。

   

   ![如何用eclispe远程调试tomcat](https://exp-picture.cdn.bcebos.com/92174dbbf82064fb7cef8c928e6104a354e96f78.jpg?x-bce-process=image%2Fresize%2Cm_lfit%2Cw_500%2Climit_1%2Fformat%2Cf_auto%2Fquality%2Cq_80)

2. 修改tomcat的启动脚本startup.bat。复制startup.bat为startup-debug.bat，然后打开startup-debug.bat，找到call "%EXECUTABLE%" start %CMD_LINE_ARGS%这一行，修改为“call "%EXECUTABLE%" jpda start %CMD_LINE_ARGS%”，然后在上面添加三行：

   set JPDA_TRANSPORT=dt_socket

   set JPDA_ADDRESS=9000

   set JPDA_SUSPEND=n

   

   ![如何用eclispe远程调试tomcat](https://exp-picture.cdn.bcebos.com/54a89daee8d7592a110b5dcb9f31dfb6336c6778.jpg?x-bce-process=image%2Fresize%2Cm_lfit%2Cw_500%2Climit_1%2Fformat%2Cf_auto%2Fquality%2Cq_80)

3.  双击“startup-debug.bat”，用debug模式启动tomcat。在tomcat的后台可以看到tomcat已经在9000端口进行监听。

   

   ![如何用eclispe远程调试tomcat](https://exp-picture.cdn.bcebos.com/def72c6c576699cffbb792d0a885e036e3915e78.jpg?x-bce-process=image%2Fresize%2Cm_lfit%2Cw_500%2Climit_1%2Fformat%2Cf_auto%2Fquality%2Cq_80)

4. 在eclipse中，点击菜单项“run”->“debug confiurations”,打开debug confiurations对话框，在里面双击“Remote Java Application”，在右边在Host中的输入tomcat的主机名，Port中输入端口号，也就是9000，然后点击“debug”。当然也可以在name中自定义一个你喜欢的名字。

   ![如何用eclispe远程调试tomcat](https://exp-picture.cdn.bcebos.com/22c4fe36e29147e8cc0484c1b603bbea3f865878.jpg?x-bce-process=image%2Fresize%2Cm_lfit%2Cw_500%2Climit_1%2Fformat%2Cf_auto%2Fquality%2Cq_80)

5. 把webtest导出为webtest.war文件,然后把webtest.war拷贝到tomcat的webapps目录下。然后在eclipse的HelloServlet第一行打一个断点，然后打开浏览器，输入http://localhost:8080/webtest/hello，然后回车。就会看到eclipse停在了断点上。

   

   ![如何用eclispe远程调试tomcat](https://exp-picture.cdn.bcebos.com/3fc72e486143d7d44fd4b9587da75f0f832b5078.jpg?x-bce-process=image%2Fresize%2Cm_lfit%2Cw_500%2Climit_1%2Fformat%2Cf_auto%2Fquality%2Cq_80)

6. 怎么样，是不是很简单，如果觉得有用，请点击投票，小编会继续努力谢谢你的支持哦。