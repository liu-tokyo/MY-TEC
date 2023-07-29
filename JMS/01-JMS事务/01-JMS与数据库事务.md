# JMS与数据库事务



## 1. 发送消息时的事务

- 使用spring的JmsTemplate组件来发送消息。这是我们THREAD系统中的配置：

  ```xml
  <bean id="newJmsTemplate" class="org.springframework.jms.core.JmsTemplate">
  	<property name="connectionFactory" ref="jmsConnectionFactory"/>
  	<property name="sessionTransacted" value="true"/>
  	<property name="explicitQosEnabled" value="${activemq.explicitQosEnabled}"/>
  	<property name="timeToLive" value="86400000"/>
  </bean>
  ```

  其中`<property name="sessionTransacted"value="true"/>`这一行就是发送消息时的事务配置。有了这个配置之后，Jms消息发送就与事务绑定上了。

- 而它在事务中的行为还与`transactionManager`有关。这是`transactionManager`配置：

  ```xml
  <beanid="transactionManager" class="org.springframework.orm.hibernate4.HibernateTransactionManager">
  	<propertyname="sessionFactory" ref="sessionFactory"/>
  	<propertyname="nestedTransactionAllowed" value="true"/>
  	<propertyname="globalRollbackOnParticipationFailure" value="false"/>
  <bean>
  ```

  在这个`transactionManager`配置下，Jms消息会在`afterCommit()`操作中提交。如下图所示：

  ![jms事务提交流程](https://note.youdao.com/yws/api/personal/file/WEB8cb5add47cbc50f4e5b22eefd40fc4d3?method=download&shareKey=956c852eb6bfbeac5a78f82ed6af246d)

- 根据网上的文章，如果要保证jms操作和数据库操作真正做到“同生共死”，必须使用`JTATransactionManager`。但是这个东东的性能一直是个问题。

  ```xml
  <tx:jta-transaction-manager/>
  ```



## 2. 接收消息时的重发机制

接收消息后的重发机制与数据库事务之间的关系，并没有像发送消息时那样严格的绑定在一起。也就是说，接收消息后是否重试与数据库事务没有必然的关系。

一般来说，做到以下两点，就可以实现重试。

- 首先，在listener的配置中增加“acknowledge="transacted"”，如下所示：

  ```xml
  <jms:listener-container destination-type="queue" concurrency="4" acknowledge="transacted" connection-factory="connectionFactory">
  	<jms:listenerdestination="queue.thread.autopay" ref="autoPayListener"/>
  </jms:listener-container>
  ```

- 第二，当消息处理失败后，抛出异常。

  ```java
  @Override
  public void onMessage(Message message) {
  	try{
  		// 省略业务处理代码
  	} catch(Exception e) {
  		// 省略日志、监控邮件等代码
  		// 抛出异常，让MQ重发一次消息。
  		throwe instanceofRuntimeException ? (RuntimeException) e : newRuntimeException(e);
  	}
  }
  ```

  默认的消息重发规则配置在org.apache.activemq.RedeliveryPolicy类中。从代码上看，默认会重发6次。如果需要修改默认配置，可以参考THREAD中spring-modules.xml文件中的配置：

  ```xml
  <amq:connectionFactoryid="connectionFactory"brokerURL="tcp://${jms.url}:61616">
  	<amq:redeliveryPolicyMap>
  		<amq:redeliveryPolicyMap>
  			<amq:defaultEntry>
  				<amq:redeliveryPolicymaximumRedeliveries="5"initialRedeliveryDelay="30000"/>
  			</amq:defaultEntry>
  			<amq:redeliveryPolicyEntries>
  				<amq:redeliveryPolicyqueue="queue.thread.autopay"maximumRedeliveries="5" initialRedeliveryDelay="10000"/>
  				<amq:redeliveryPolicyqueue="queue.thread.instantUnionpay"maximumRedeliveries="5" initialRedeliveryDelay="90000"/>
  			</amq:redeliveryPolicyEntries>
  		</amq:redeliveryPolicyMap>
  	</amq:redeliveryPolicyMap>
  </amq:connectionFactory>
  ```

  如果重试次数超过上限，MQ会将消息转到另一个“投递失败队列”（Dead Letter Queue）中。“投递失败队列”的名称一般就是“DLQ.原队列名”。
