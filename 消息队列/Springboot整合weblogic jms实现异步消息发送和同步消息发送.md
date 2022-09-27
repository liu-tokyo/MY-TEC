# Springboot整合weblogic jms实现异步消息发送和同步消息发送

项目背景1：网络分为外联网、内网。外联网不能直接请求内网，只能将请求以消息形式发送到jms server服务器上，内网去监听外联网的jms server服务器，实现内外网信息交互、同步。  
项目背景2：用户支付完成后，需要调用第三方系统（支付宝、微信等）接口，得到支付状态后进行操作。异步

1. 安装weblogic，这个请自行百度。

2. 部署jms服务器，请参看：https://blog.csdn.net/gxlstone/article/details/41378949

3. springboot整合welobic jms代码

   本文参考了几个博主，感谢几位博主的贡献。因为只能填写一个转载地址，各位博主请不要介意。。。。  
   慕课网spring整合activemq，代码基本类似：https://www.imooc.com/learn/856  
   jms理论知识讲解：https://blog.csdn.net/zhangzikui/article/details/24837999  
   weblogic jms的简单使用：https://blog.csdn.net/elim168/article/details/72674355  
   jms的几种监听器区别：https://blog.csdn.net/zylzb/article/details/40511047

   

到这里正文开始，各位猿们，上菜。

## 1. pom.xml配置

pom.xml配置，一个jms的包，两个weblogic客户端的包。weblogic客户端的包请自行查看我发的以上博主的链接。

```xml
<dependency>
	<groupId>org.springframework</groupId>
	<artifactId>spring-jms</artifactId>
	<version>4.3.11.RELEASE</version>
</dependency>

<dependency>
	<groupId>wlfullclient</groupId>
	<artifactId>wlfullclient.jar</artifactId>
	<version>12c</version>
	<scope>system</scope>
	<systemPath>/worktools/apache-maven-jar/weblogic/wlfullclient.jar</systemPath>
</dependency>
<dependency>
	<groupId>wlclient</groupId>
	<artifactId>wlclient.jar</artifactId>
	<version>12c</version>
	<scope>system</scope>
	<systemPath>/worktools/apache-maven-jar/weblogic/wlclient.jar</systemPath>
</dependency>
```

## 2. bean配置类

```java
import org.springframework.boot.autoconfigure.condition.ConditionalOnMissingBean;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.jms.connection.CachingConnectionFactory;
import org.springframework.jms.core.JmsTemplate;
import org.springframework.jndi.JndiObjectFactoryBean;
import org.springframework.jndi.JndiTemplate;
import javax.jms.ConnectionFactory;
import java.util.Properties;

/**
 * @author HuRongHua
 * @describe
 * @date 2021/2/7 3:55 下午
 */
@Configuration
public class JsmConfig {

    @Bean
    public JndiTemplate jndiTemplate() {
        Properties properties = new Properties();
        properties.setProperty("java.naming.factory.initial", "weblogic.jndi.WLInitialContextFactory");
        properties.setProperty("java.naming.provider.url", "t3://localhost:7001");
        JndiTemplate jndiTemplate = new JndiTemplate();
        jndiTemplate.setEnvironment(properties);
        return jndiTemplate;
    }

    //weblogic jms提供的连接工厂
    @Bean
    public JndiObjectFactoryBean jmsConnectionFactory() {
        JndiObjectFactoryBean jndiObjectFactoryBean = new JndiObjectFactoryBean();
        //连接工厂的JNDI名
        jndiObjectFactoryBean.setJndiName("ConnectionFactoryJNDI-0");
        jndiObjectFactoryBean.setJndiTemplate(jndiTemplate());
        return jndiObjectFactoryBean;
    }

    //weblogic jms消息目的地
    @Bean("queueSend")
    public JndiObjectFactoryBean jmsQueue() {
        JndiObjectFactoryBean jndiObjectFactoryBean = new JndiObjectFactoryBean();
        //队列的JNDI名
        jndiObjectFactoryBean.setJndiName("DistributedQueueJNDI-0");
        jndiObjectFactoryBean.setJndiTemplate(jndiTemplate());
        return jndiObjectFactoryBean;
    }

    //spring提供的连接池
    public ConnectionFactory getcachingConnectionFactory(){
        CachingConnectionFactory connectionFactory=new CachingConnectionFactory();
        //将jms的连接池注入到spring的连接池中，由spring进行管理
        connectionFactory.setTargetConnectionFactory((ConnectionFactory) jmsConnectionFactory().getObject());
        return connectionFactory;
    }

    //jmsTemplate，消息发送器
    @Bean("jmsTemplate")
    @ConditionalOnMissingBean
    public JmsTemplate jmsTemplate() {
        JmsTemplate jmsTemplate = new JmsTemplate();
        //注入spring提供的连接池
        jmsTemplate.setConnectionFactory(getcachingConnectionFactory());
        //设置消息超时时间
        jmsTemplate.setReceiveTimeout(3000);
        return jmsTemplate;
    }
    
	//消息监听容器
    @Bean
    public DefaultMessageListenerContainer getWeblogicJsmSessionAwareConsumer(){
        DefaultMessageListenerContainer defaultMessageListenerContainer=new DefaultMessageListenerContainer();
        defaultMessageListenerContainer.setConnectionFactory(getcachingConnectionFactory());
        //消息监听器
        defaultMessageListenerContainer.setMessageListener(new JsmSessionAwareConsumer());
        //监听地址
        defaultMessageListenerContainer.setDestination((Destination) Objects.requireNonNull(jmsQueue().getObject()));
        //设置动态线程，加快获取速度
        defaultMessageListenerContainer.setConcurrency("2-4");
        return defaultMessageListenerContainer;
    }
}
```



## 3. 生产者

外联网的生产者，CommonResult为前后台交互的类，可自行修改。

```java
import com.alibaba.fastjson.JSONObject;
import com.alipay.tyzf.util.CommonResult;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.jms.core.JmsTemplate;
import org.springframework.jms.core.MessageCreator;
import org.springframework.stereotype.Component;

import javax.jms.*;
import java.util.HashMap;

/**
 * @author HuRongHua
 * @describe
 * @date 2021/2/7 3:55 下午
 */
@Component
public class JsmProducer {

    private static JmsTemplate jmsTemplate;

    //消息目的地
    private static Queue destination;

    //使用构造方法注入，将bean注入到静态对象里
    @Autowired
    private JsmProducer(JmsTemplate jmsTemplate, Queue destination) {
        JsmProducer.jmsTemplate = jmsTemplate;
        JsmProducer.destination = destination;
    }

    //异步消息
    public void send(final String userinfo) throws JMSException {
        //发送异步消息
        jmsTemplate.send(destination,new MessageCreator() {
            @Override
            public Message createMessage(Session session) throws JMSException {
                return session.createTextMessage(userinfo);
            }
        });
    }

    //同步消息
    //因为内外网交互的类必须是一致，包括包名。所以采用HashMap传递，String接收
    public static CommonResult sendAndReceive(final HashMap mapInfo){
    	//该类是交互类，可自行定义
        CommonResult commonResult = new CommonResult();
        try {
            //发送同步消息
            Message message = jmsTemplate.sendAndReceive(destination,new MessageCreator() {
                @Override
                public Message createMessage(Session session) throws JMSException {
                    return session.createObjectMessage(mapInfo);
                }
            });
                String objectMessage = (String) ((ObjectMessage) message).getObject();
                commonResult = JSONObject.parseObject(objectMessage, CommonResult.class);
        } catch (JMSException e) {
            e.printStackTrace();
        }
        return commonResult;
    }
}
```



## 4. 消费者 - 监听器

内网消息监听器，因为内网项目已经成型，无法进行改造，只能去适应它。onMessage监听方法中部分代码为业务操作，可自行舍弃。

```java
import com.alibaba.fastjson.JSON;
import org.app.pay.core.dto.RestResponse;
import org.app.pay.external.api.sbf.Sbf_Sy_Inf_Controller;
import org.app.pay.inside.dto.InPayDto;
import org.springframework.beans.BeansException;
import org.springframework.context.ApplicationContext;
import org.springframework.context.ApplicationContextAware;
import org.springframework.jms.listener.SessionAwareMessageListener;
import org.springframework.jms.support.JmsUtils;
import org.springframework.stereotype.Component;
import javax.jms.*;
import java.util.HashMap;

/**
 * @author HuRongHua
 * @describe
 * @date 2021/2/3 9:29 下午
 */
//监听外网发送的消息容器
public class JsmSessionAwareConsumer implements SessionAwareMessageListener<Message> {

    @Override
    public void onMessage(Message message, Session session) throws JMSException {
        ObjectMessage textMessage = (ObjectMessage) message;
        //接受外网发送的消息
        HashMap<String,String> mapInfo = (HashMap) textMessage.getObject();
        InPayDto inPayDto=new InPayDto();
        //组装请求信息
        inPayDto.setTran_id(mapInfo.get(""));
        inPayDto.setChannelId(mapInfo.get(""));
        inPayDto.setData(mapInfo.get(""));
        //获取公共请求类
        Inf_Controller service = SpringUtil.getBean(Inf_Controller.class);
        ObjectMessage response = null;
        try {
            //发起请求
            RestResponse restResponse = service.unpayment(inPayDto);
            //返回消息用string，外网用string去接受
            String strResq = JSON.toJSONString(restResponse);
            response = session.createObjectMessage(strResq);
        } catch (Exception e) {
            e.printStackTrace();
        }
        //建立消息消费者（需要返回信息），设置消息目的地
        MessageProducer producer = session.createProducer(message.getJMSReplyTo());
        assert response != null;
        //这一步是设置消息的id，不设置消息无法返回
        response.setJMSCorrelationID(message.getJMSCorrelationID());
        //返回消息
        producer.send(response);
        //关闭连接
        JmsUtils.closeMessageProducer(producer);
    }


    @Component
    public static class SpringUtil implements ApplicationContextAware {

        private static ApplicationContext applicationContext;

        //获取applicationContext
        public static ApplicationContext getApplicationContext() {
            return applicationContext;
        }

        //通过name获取 Bean.
        public static Object getBean(String name) {
            return getApplicationContext().getBean(name);
        }

        //通过class获取Bean
        public static <T> T getBean(Class<T> clazz) {
            return getApplicationContext().getBean(clazz);
        }

        //通过name,以及Clazz返回指定的Bean
        public static <T> T getBean(String name, Class<T> clazz) {
            return getApplicationContext().getBean(name, clazz);
        }

        @Override
        public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
            if (SpringUtil.applicationContext == null) {
                SpringUtil.applicationContext = applicationContext;
            }
        }
    }
}
```

使用方法：
异步消息发送：JsmProducer.send(String)；
同步消息发送：JsmProducer.sendAndReceive(HashMap)；

## 注意事项

1. 消息发送者和消息监听者交互的类必须为相同包路径+类名的类，且需要继承Serializable，不然会报错，因为消息发送和接收需要对消息进行序列化。本文交互的类为HashMap。
2. 生产者和消费者发送的消息必须类型一致，如本文使用的消息类型ObjectMessage。还有其他几种请自行查看，若用的不一致，在接收到消息的时候就会报错。
3. 本文使用的spring-jms版本为4.3.11.RELEASE，低版本的jms可能会没有jmsTemplate.sendAndReceive这个方法。

到这里springBoot整合weblogic jms就算结束了。

https://blog.csdn.net/qq_37971434/article/details/113764057