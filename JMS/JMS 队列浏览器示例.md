# JMS 队列浏览器示例

点对点消息传递队列包含对特定目标队列感兴趣的客户端要使用的消息。如果一个人想简单地监视消息而不实际使用它们，那么掌握将允许人们提前查看待处理的消息。

## 1. 依赖关系

为了向 JMS 消息代理发送和接收 JMS 消息，我们需要包含消息服务库。在这个例子中，我们使用 activeMq，所以我们的 pom.xml 将具有与 spring 和 activeMQ 相关的依赖项。

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.javacodegeeks.jms</groupId>
    <artifactId>springJmsQueue</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <dependencies>
        <dependency>
            <groupId>org.apache.activemq</groupId>
            <artifactId>activemq-all</artifactId>
            <version>5.12.0</version>
        </dependency>
    </dependencies>
     
</project>
```

## 2. 启动 JMS 提供程序

JMS 是一个规范，而不是一个实际的产品。JMS提供程序（如ActiveMQ，IBM，Progress Software甚至Sun）提供了实现该规范的消息传递产品。在我们的示例中，我们将使用 ActiveMQ 作为 JMS 提供程序。开始使用ActiveMQ并不困难。您只需要启动代理并确保它能够接受连接和发送消息。

在下面的示例中，代理作为侦听端口 `61616` 的服务器启动。愿意连接到代理的 JMS 客户机将使用 TCP 协议 （`tcp://localhost:61616`）。由于代理和 JMS 客户端在同一台机器上运行，因此我们使用了 `localhost`。

```java
package com.javacodegeeks.jms;
 
import java.net.URI;
import java.net.URISyntaxException;
 
import org.apache.activemq.broker.BrokerFactory;
import org.apache.activemq.broker.BrokerService;
 
public class BrokerLauncher {
    public static void main(String[] args) throws URISyntaxException, Exception {
        BrokerService broker = BrokerFactory.createBroker(new URI(
                "broker:(tcp://localhost:61616)"));
        broker.start();     
    }
}
```

输出：

```
INFO | JMX consoles can connect to service:jmx:rmi:///jndi/rmi://localhost:1099/jmxrmi
INFO | PListStore:[C:\javacodegeeks_ws\jmsQueueBrowserExample\activemq-data\localhost\tmp_storage] started
INFO | Using Persistence Adapter: KahaDBPersistenceAdapter[C:\javacodegeeks_ws\jmsQueueBrowserExample\activemq-data\localhost\KahaDB]
INFO | Apache ActiveMQ 5.12.0 (localhost, ID:INMAA1-L1005-57531-1450761099016-0:1) is starting
INFO | Listening for connections at: tcp://127.0.0.1:61616
INFO | Connector tcp://127.0.0.1:61616 started
INFO | Apache ActiveMQ 5.12.0 (localhost, ID:INMAA1-L1005-57531-1450761099016-0:1) started
INFO | For help or more information please see: http://activemq.apache.org
WARN | Store limit is 102400 mb (current store usage is 0 mb). The data directory: C:\javacodegeeks_ws\jmsQueueBrowserExample\activemq-data\localhost\KahaDB only has 30295 mb of usable space - resetting to maximum available disk space: 30295 mb
WARN | Temporary Store limit is 51200 mb, whilst the temporary data directory: C:\javacodegeeks_ws\jmsQueueBrowserExample\activemq-data\localhost\tmp_storage only has 30295 mb of usable space - resetting to maximum available 30295 mb.
```

## 3. 创建队列浏览器

QueueBrowser 可用于从管理工具监视队列的内容，或浏览多条消息以查找感兴趣的特定消息。

A 是使用对象中的方法创建的。它需要需要浏览的对象。如果要进一步过滤，也可以传入消息选择器。

消息选择器包含用于筛选消息的表达式。将仅传递其属性与消息选择器表达式匹配的消息。

```java
public interface Session extends Runnable {
...
    QueueBrowser createBrowser(Queue queue) throws JMSException;
 
    QueueBrowser createBrowser(Queue queue, String messageSelector)
        throws JMSException;
...
}
```

```java
QueueBrowser browser = session.createBrowser(queue);
```

## 4. 队列浏览器接口

队列浏览器实现该接口。

```java
public interface QueueBrowser {
    Queue getQueue() throws JMSException;
    String getMessageSelector() throws JMSException;
    Enumeration getEnumeration() throws JMSException;
    void close() throws JMSException;
}
```

- getQueue()– 返回需要浏览的队列
- getMessageSelector()– 返回用于过滤队列中的消息的消息选择器表达式。
- getEnumeration()返回将用于循环访问队列的枚举对象
- close()– 每当不再需要浏览器时，需要待用该函数，来释放 JMS Browse 相关的所有资源。

## 5. 队列浏览器示例

为了浏览消息，我们需要在`Session`上调用对象`createQueueBrowser( )`。我们传入了需要浏览的`Queue`对象。

该`QueueBrowser`对象包含一个`java.util.Enumeration`用于循环访问队列。

我们将一些消息发送到队列，然后创建一个队列浏览器对象来枚举消息。接下来，我们将关闭队列浏览器对象。最后，我们将从队列中接收所有消息。

```java
package com.javacodegeeks.jms;
 
import java.net.URISyntaxException;
import java.util.Enumeration;
 
import javax.jms.Connection;
import javax.jms.ConnectionFactory;
import javax.jms.Message;
import javax.jms.MessageConsumer;
import javax.jms.MessageProducer;
import javax.jms.Queue;
import javax.jms.QueueBrowser;
import javax.jms.Session;
import javax.jms.TextMessage;
 
import org.apache.activemq.ActiveMQConnectionFactory;
 
public class JmsQueueBrowseExample {
    public static void main(String[] args) throws URISyntaxException, Exception {
        Connection connection = null;
        try {
            // Producer
            ConnectionFactory connectionFactory = new ActiveMQConnectionFactory(
                    "tcp://localhost:61616");
            connection = connectionFactory.createConnection();
            Session session = connection.createSession(false,
                    Session.AUTO_ACKNOWLEDGE);
            Queue queue = session.createQueue("browseQueue");
            MessageProducer producer = session.createProducer(queue);
            String task = "Task";
            for (int i = 0; i < 10; i++) {
                String payload = task + i;
                Message msg = session.createTextMessage(payload);
                System.out.println("Sending text '" + payload + "'");
                producer.send(msg);
            }
 
            MessageConsumer consumer = session.createConsumer(queue);
            connection.start();
 
            System.out.println("Browse through the elements in queue");
            QueueBrowser browser = session.createBrowser(queue);
            Enumeration e = browser.getEnumeration();
            while (e.hasMoreElements()) {
                TextMessage message = (TextMessage) e.nextElement();
                System.out.println("Browse [" + message.getText() + "]");
            }
            System.out.println("Done");
            browser.close();
 
            for (int i = 0; i < 10; i++) {
                TextMessage textMsg = (TextMessage) consumer.receive();
                System.out.println(textMsg);
                System.out.println("Received: " + textMsg.getText());
            }
            session.close();
        } finally {
            if (connection != null) {
                connection.close();
            }
        }
    }
 
}
```

结果输出：

```
Sending text 'Task0'
Sending text 'Task1'
Sending text 'Task2'
Sending text 'Task3'
Sending text 'Task4'
Sending text 'Task5'
Sending text 'Task6'
Sending text 'Task7'
Sending text 'Task8'
Sending text 'Task9'
Browse through the elements in queue
Browse [Task0]
Browse [Task1]
Browse [Task2]
Browse [Task3]
Browse [Task4]
Browse [Task5]
Browse [Task6]
Browse [Task7]
Browse [Task8]
Browse [Task9]
Done
ActiveMQTextMessage {commandId = 5, responseRequired = true, messageId = ID:INMAA1-L1005-58661-1450762225866-1:1:1:1:1, originalDestination = null, originalTransactionId = null, producerId = ID:INMAA1-L1005-58661-1450762225866-1:1:1:1, destination = queue://browseQueue, transactionId = null, expiration = 0, timestamp = 1450762226017, arrival = 0, brokerInTime = 1450762226018, brokerOutTime = 1450762226054, correlationId = null, replyTo = null, persistent = true, type = null, priority = 4, groupID = null, groupSequence = 0, targetConsumerId = null, compressed = false, userID = null, content = org.apache.activemq.util.ByteSequence@6b71769e, marshalledProperties = null, dataStructure = null, redeliveryCounter = 0, size = 0, properties = null, readOnlyProperties = true, readOnlyBody = true, droppable = false, jmsXGroupFirstForConsumer = false, text = Task0}
Received: Task0
ActiveMQTextMessage {commandId = 6, responseRequired = true, messageId = ID:INMAA1-L1005-58661-1450762225866-1:1:1:1:2, originalDestination = null, originalTransactionId = null, producerId = ID:INMAA1-L1005-58661-1450762225866-1:1:1:1, destination = queue://browseQueue, transactionId = null, expiration = 0, timestamp = 1450762226025, arrival = 0, brokerInTime = 1450762226025, brokerOutTime = 1450762226054, correlationId = null, replyTo = null, persistent = true, type = null, priority = 4, groupID = null, groupSequence = 0, targetConsumerId = null, compressed = false, userID = null, content = org.apache.activemq.util.ByteSequence@2752f6e2, marshalledProperties = null, dataStructure = null, redeliveryCounter = 0, size = 0, properties = null, readOnlyProperties = true, readOnlyBody = true, droppable = false, jmsXGroupFirstForConsumer = false, text = Task1}
Received: Task1
ActiveMQTextMessage {commandId = 7, responseRequired = true, messageId = ID:INMAA1-L1005-58661-1450762225866-1:1:1:1:3, originalDestination = null, originalTransactionId = null, producerId = ID:INMAA1-L1005-58661-1450762225866-1:1:1:1, destination = queue://browseQueue, transactionId = null, expiration = 0, timestamp = 1450762226027, arrival = 0, brokerInTime = 1450762226028, brokerOutTime = 1450762226054, correlationId = null, replyTo = null, persistent = true, type = null, priority = 4, groupID = null, groupSequence = 0, targetConsumerId = null, compressed = false, userID = null, content = org.apache.activemq.util.ByteSequence@e580929, marshalledProperties = null, dataStructure = null, redeliveryCounter = 0, size = 0, properties = null, readOnlyProperties = true, readOnlyBody = true, droppable = false, jmsXGroupFirstForConsumer = false, text = Task2}
Received: Task2
ActiveMQTextMessage {commandId = 8, responseRequired = true, messageId = ID:INMAA1-L1005-58661-1450762225866-1:1:1:1:4, originalDestination = null, originalTransactionId = null, producerId = ID:INMAA1-L1005-58661-1450762225866-1:1:1:1, destination = queue://browseQueue, transactionId = null, expiration = 0, timestamp = 1450762226030, arrival = 0, brokerInTime = 1450762226030, brokerOutTime = 1450762226054, correlationId = null, replyTo = null, persistent = true, type = null, priority = 4, groupID = null, groupSequence = 0, targetConsumerId = null, compressed = false, userID = null, content = org.apache.activemq.util.ByteSequence@1cd072a9, marshalledProperties = null, dataStructure = null, redeliveryCounter = 0, size = 0, properties = null, readOnlyProperties = true, readOnlyBody = true, droppable = false, jmsXGroupFirstForConsumer = false, text = Task3}
Received: Task3
ActiveMQTextMessage {commandId = 9, responseRequired = true, messageId = ID:INMAA1-L1005-58661-1450762225866-1:1:1:1:5, originalDestination = null, originalTransactionId = null, producerId = ID:INMAA1-L1005-58661-1450762225866-1:1:1:1, destination = queue://browseQueue, transactionId = null, expiration = 0, timestamp = 1450762226033, arrival = 0, brokerInTime = 1450762226033, brokerOutTime = 1450762226054, correlationId = null, replyTo = null, persistent = true, type = null, priority = 4, groupID = null, groupSequence = 0, targetConsumerId = null, compressed = false, userID = null, content = org.apache.activemq.util.ByteSequence@7c75222b, marshalledProperties = null, dataStructure = null, redeliveryCounter = 0, size = 0, properties = null, readOnlyProperties = true, readOnlyBody = true, droppable = false, jmsXGroupFirstForConsumer = false, text = Task4}
Received: Task4
ActiveMQTextMessage {commandId = 10, responseRequired = true, messageId = ID:INMAA1-L1005-58661-1450762225866-1:1:1:1:6, originalDestination = null, originalTransactionId = null, producerId = ID:INMAA1-L1005-58661-1450762225866-1:1:1:1, destination = queue://browseQueue, transactionId = null, expiration = 0, timestamp = 1450762226036, arrival = 0, brokerInTime = 1450762226036, brokerOutTime = 1450762226054, correlationId = null, replyTo = null, persistent = true, type = null, priority = 4, groupID = null, groupSequence = 0, targetConsumerId = null, compressed = false, userID = null, content = org.apache.activemq.util.ByteSequence@4c203ea1, marshalledProperties = null, dataStructure = null, redeliveryCounter = 0, size = 0, properties = null, readOnlyProperties = true, readOnlyBody = true, droppable = false, jmsXGroupFirstForConsumer = false, text = Task5}
Received: Task5
ActiveMQTextMessage {commandId = 11, responseRequired = true, messageId = ID:INMAA1-L1005-58661-1450762225866-1:1:1:1:7, originalDestination = null, originalTransactionId = null, producerId = ID:INMAA1-L1005-58661-1450762225866-1:1:1:1, destination = queue://browseQueue, transactionId = null, expiration = 0, timestamp = 1450762226039, arrival = 0, brokerInTime = 1450762226039, brokerOutTime = 1450762226054, correlationId = null, replyTo = null, persistent = true, type = null, priority = 4, groupID = null, groupSequence = 0, targetConsumerId = null, compressed = false, userID = null, content = org.apache.activemq.util.ByteSequence@27f674d, marshalledProperties = null, dataStructure = null, redeliveryCounter = 0, size = 0, properties = null, readOnlyProperties = true, readOnlyBody = true, droppable = false, jmsXGroupFirstForConsumer = false, text = Task6}
Received: Task6
ActiveMQTextMessage {commandId = 12, responseRequired = true, messageId = ID:INMAA1-L1005-58661-1450762225866-1:1:1:1:8, originalDestination = null, originalTransactionId = null, producerId = ID:INMAA1-L1005-58661-1450762225866-1:1:1:1, destination = queue://browseQueue, transactionId = null, expiration = 0, timestamp = 1450762226042, arrival = 0, brokerInTime = 1450762226042, brokerOutTime = 1450762226054, correlationId = null, replyTo = null, persistent = true, type = null, priority = 4, groupID = null, groupSequence = 0, targetConsumerId = null, compressed = false, userID = null, content = org.apache.activemq.util.ByteSequence@1d251891, marshalledProperties = null, dataStructure = null, redeliveryCounter = 0, size = 0, properties = null, readOnlyProperties = true, readOnlyBody = true, droppable = false, jmsXGroupFirstForConsumer = false, text = Task7}
Received: Task7
ActiveMQTextMessage {commandId = 13, responseRequired = true, messageId = ID:INMAA1-L1005-58661-1450762225866-1:1:1:1:9, originalDestination = null, originalTransactionId = null, producerId = ID:INMAA1-L1005-58661-1450762225866-1:1:1:1, destination = queue://browseQueue, transactionId = null, expiration = 0, timestamp = 1450762226045, arrival = 0, brokerInTime = 1450762226045, brokerOutTime = 1450762226054, correlationId = null, replyTo = null, persistent = true, type = null, priority = 4, groupID = null, groupSequence = 0, targetConsumerId = null, compressed = false, userID = null, content = org.apache.activemq.util.ByteSequence@48140564, marshalledProperties = null, dataStructure = null, redeliveryCounter = 0, size = 0, properties = null, readOnlyProperties = true, readOnlyBody = true, droppable = false, jmsXGroupFirstForConsumer = false, text = Task8}
Received: Task8
ActiveMQTextMessage {commandId = 14, responseRequired = true, messageId = ID:INMAA1-L1005-58661-1450762225866-1:1:1:1:10, originalDestination = null, originalTransactionId = null, producerId = ID:INMAA1-L1005-58661-1450762225866-1:1:1:1, destination = queue://browseQueue, transactionId = null, expiration = 0, timestamp = 1450762226047, arrival = 0, brokerInTime = 1450762226047, brokerOutTime = 1450762226054, correlationId = null, replyTo = null, persistent = true, type = null, priority = 4, groupID = null, groupSequence = 0, targetConsumerId = null, compressed = false, userID = null, content = org.apache.activemq.util.ByteSequence@58ceff1, marshalledProperties = null, dataStructure = null, redeliveryCounter = 0, size = 0, properties = null, readOnlyProperties = true, readOnlyBody = true, droppable = false, jmsXGroupFirstForConsumer = false, text = Task9}
Received: Task9
```

## 6. 带有消息选择器的队列浏览器示例

在此示例中，我们将看到方法的重载版本`Queue`的`createQueueBrowser()`，该方法接受以及`messageSelector` .

消息选择器包含用于筛选消息的表达式。将仅传递其属性与消息选择器表达式匹配的消息。

例如，我们将偶数序列消息的“序列”属性设置为“偶数”。

```java
for (int i = 0; i < 10; i++) {
                String payload = task + i;
                Message msg = session.createTextMessage(payload);
                System.out.println("Sending text '" + payload + "'");
                if (i % 2 == 0) {
                    msg.setStringProperty("sequence", "even");
                }
                producer.send(msg);
            }
```

假设我们只想浏览偶数排序的消息，那么消息选择器将是 .`"sequence = 'even'"`

我们将使用此消息创建`QueueBrowser`选择器。

```java
QueueBrowser browser = session.createBrowser(queue,
                    "sequence = 'even'");
```

JmsBrowseQueueMessageSelectorExample：

```java
package com.javacodegeeks.jms;
 
import java.net.URISyntaxException;
import java.util.Enumeration;
 
import javax.jms.Connection;
import javax.jms.ConnectionFactory;
import javax.jms.Message;
import javax.jms.MessageConsumer;
import javax.jms.MessageProducer;
import javax.jms.Queue;
import javax.jms.QueueBrowser;
import javax.jms.Session;
import javax.jms.TextMessage;
 
import org.apache.activemq.ActiveMQConnectionFactory;
 
public class JmsQueueBrowserWithMessageSelectorExample {
    public static void main(String[] args) throws URISyntaxException, Exception {
        Connection connection = null;
        try {
            // Producer
            ConnectionFactory connectionFactory = new ActiveMQConnectionFactory(
                    "tcp://localhost:61616");
            connection = connectionFactory.createConnection();
            Session session = connection.createSession(false,
                    Session.AUTO_ACKNOWLEDGE);
            Queue queue = session.createQueue("customerQueue");
 
            MessageProducer producer = session.createProducer(queue);
            String task = "Task";
            for (int i = 0; i < 10; i++) {
                String payload = task + i;
                Message msg = session.createTextMessage(payload);
                System.out.println("Sending text '" + payload + "'");
                if (i % 2 == 0) {
                    msg.setStringProperty("sequence", "even");
                }
                producer.send(msg);
            }
 
            MessageConsumer consumer = session.createConsumer(queue);
            connection.start();
 
            System.out.println("Browse through the elements in queue");
            QueueBrowser browser = session.createBrowser(queue,
                    "sequence = 'even'");
            Enumeration e = browser.getEnumeration();
            while (e.hasMoreElements()) {
                TextMessage message = (TextMessage) e.nextElement();
                System.out.println("Browse [" + message.getText() + "]");
            }
            System.out.println("Done");
            browser.close();
 
            for (int i = 0; i < 10; i++) {
                TextMessage textMsg = (TextMessage) consumer.receive();
                System.out.println(textMsg);
                System.out.println("Received: " + textMsg.getText());
            }
            session.close();
        } finally {
            if (connection != null) {
                connection.close();
            }
        }
    }
} 
```

结果您只能看到枚举对象中显示的偶数序列消息。

结果输出：

```
Sending text 'Task0'
Sending text 'Task1'
Sending text 'Task2'
Sending text 'Task3'
Sending text 'Task4'
Sending text 'Task5'
Sending text 'Task6'
Sending text 'Task7'
Sending text 'Task8'
Sending text 'Task9'
Browse through the elements in queue
Browse [Task0]
Browse [Task2]
Browse [Task4]
Browse [Task6]
Browse [Task8]
Done
ActiveMQTextMessage {commandId = 7, responseRequired = true, messageId = ID:INMAA1-L1005-57862-1450761420954-1:1:1:1:3, originalDestination = null, originalTransactionId = null, producerId = ID:INMAA1-L1005-57862-1450761420954-1:1:1:1, destination = queue://customerQueue, transactionId = null, expiration = 0, timestamp = 1450761421106, arrival = 0, brokerInTime = 1450761421106, brokerOutTime = 1450761525026, correlationId = null, replyTo = null, persistent = true, type = null, priority = 4, groupID = null, groupSequence = 0, targetConsumerId = null, compressed = false, userID = null, content = org.apache.activemq.util.ByteSequence@5ebec15, marshalledProperties = null, dataStructure = null, redeliveryCounter = 0, size = 0, properties = null, readOnlyProperties = true, readOnlyBody = true, droppable = false, jmsXGroupFirstForConsumer = false, text = Task2}
Received: Task2
ActiveMQTextMessage {commandId = 8, responseRequired = true, messageId = ID:INMAA1-L1005-57862-1450761420954-1:1:1:1:4, originalDestination = null, originalTransactionId = null, producerId = ID:INMAA1-L1005-57862-1450761420954-1:1:1:1, destination = queue://customerQueue, transactionId = null, expiration = 0, timestamp = 1450761421109, arrival = 0, brokerInTime = 1450761421109, brokerOutTime = 1450761525026, correlationId = null, replyTo = null, persistent = true, type = null, priority = 4, groupID = null, groupSequence = 0, targetConsumerId = null, compressed = false, userID = null, content = org.apache.activemq.util.ByteSequence@21bcffb5, marshalledProperties = null, dataStructure = null, redeliveryCounter = 0, size = 0, properties = null, readOnlyProperties = true, readOnlyBody = true, droppable = false, jmsXGroupFirstForConsumer = false, text = Task3}
Received: Task3
ActiveMQTextMessage {commandId = 9, responseRequired = true, messageId = ID:INMAA1-L1005-57862-1450761420954-1:1:1:1:5, originalDestination = null, originalTransactionId = null, producerId = ID:INMAA1-L1005-57862-1450761420954-1:1:1:1, destination = queue://customerQueue, transactionId = null, expiration = 0, timestamp = 1450761421111, arrival = 0, brokerInTime = 1450761421112, brokerOutTime = 1450761525027, correlationId = null, replyTo = null, persistent = true, type = null, priority = 4, groupID = null, groupSequence = 0, targetConsumerId = null, compressed = false, userID = null, content = org.apache.activemq.util.ByteSequence@380fb434, marshalledProperties = null, dataStructure = null, redeliveryCounter = 0, size = 0, properties = null, readOnlyProperties = true, readOnlyBody = true, droppable = false, jmsXGroupFirstForConsumer = false, text = Task4}
Received: Task4
ActiveMQTextMessage {commandId = 10, responseRequired = true, messageId = ID:INMAA1-L1005-57862-1450761420954-1:1:1:1:6, originalDestination = null, originalTransactionId = null, producerId = ID:INMAA1-L1005-57862-1450761420954-1:1:1:1, destination = queue://customerQueue, transactionId = null, expiration = 0, timestamp = 1450761421114, arrival = 0, brokerInTime = 1450761421114, brokerOutTime = 1450761525027, correlationId = null, replyTo = null, persistent = true, type = null, priority = 4, groupID = null, groupSequence = 0, targetConsumerId = null, compressed = false, userID = null, content = org.apache.activemq.util.ByteSequence@668bc3d5, marshalledProperties = null, dataStructure = null, redeliveryCounter = 0, size = 0, properties = null, readOnlyProperties = true, readOnlyBody = true, droppable = false, jmsXGroupFirstForConsumer = false, text = Task5}
Received: Task5
ActiveMQTextMessage {commandId = 11, responseRequired = true, messageId = ID:INMAA1-L1005-57862-1450761420954-1:1:1:1:7, originalDestination = null, originalTransactionId = null, producerId = ID:INMAA1-L1005-57862-1450761420954-1:1:1:1, destination = queue://customerQueue, transactionId = null, expiration = 0, timestamp = 1450761421117, arrival = 0, brokerInTime = 1450761421117, brokerOutTime = 1450761525027, correlationId = null, replyTo = null, persistent = true, type = null, priority = 4, groupID = null, groupSequence = 0, targetConsumerId = null, compressed = false, userID = null, content = org.apache.activemq.util.ByteSequence@3cda1055, marshalledProperties = null, dataStructure = null, redeliveryCounter = 0, size = 0, properties = null, readOnlyProperties = true, readOnlyBody = true, droppable = false, jmsXGroupFirstForConsumer = false, text = Task6}
Received: Task6
ActiveMQTextMessage {commandId = 12, responseRequired = true, messageId = ID:INMAA1-L1005-57862-1450761420954-1:1:1:1:8, originalDestination = null, originalTransactionId = null, producerId = ID:INMAA1-L1005-57862-1450761420954-1:1:1:1, destination = queue://customerQueue, transactionId = null, expiration = 0, timestamp = 1450761421120, arrival = 0, brokerInTime = 1450761421120, brokerOutTime = 1450761525028, correlationId = null, replyTo = null, persistent = true, type = null, priority = 4, groupID = null, groupSequence = 0, targetConsumerId = null, compressed = false, userID = null, content = org.apache.activemq.util.ByteSequence@7a5d012c, marshalledProperties = null, dataStructure = null, redeliveryCounter = 0, size = 0, properties = null, readOnlyProperties = true, readOnlyBody = true, droppable = false, jmsXGroupFirstForConsumer = false, text = Task7}
Received: Task7
ActiveMQTextMessage {commandId = 13, responseRequired = true, messageId = ID:INMAA1-L1005-57862-1450761420954-1:1:1:1:9, originalDestination = null, originalTransactionId = null, producerId = ID:INMAA1-L1005-57862-1450761420954-1:1:1:1, destination = queue://customerQueue, transactionId = null, expiration = 0, timestamp = 1450761421123, arrival = 0, brokerInTime = 1450761421123, brokerOutTime = 1450761525028, correlationId = null, replyTo = null, persistent = true, type = null, priority = 4, groupID = null, groupSequence = 0, targetConsumerId = null, compressed = false, userID = null, content = org.apache.activemq.util.ByteSequence@3fb6a447, marshalledProperties = null, dataStructure = null, redeliveryCounter = 0, size = 0, properties = null, readOnlyProperties = true, readOnlyBody = true, droppable = false, jmsXGroupFirstForConsumer = false, text = Task8}
Received: Task8
ActiveMQTextMessage {commandId = 14, responseRequired = true, messageId = ID:INMAA1-L1005-57862-1450761420954-1:1:1:1:10, originalDestination = null, originalTransactionId = null, producerId = ID:INMAA1-L1005-57862-1450761420954-1:1:1:1, destination = queue://customerQueue, transactionId = null, expiration = 0, timestamp = 1450761421126, arrival = 0, brokerInTime = 1450761421126, brokerOutTime = 1450761525028, correlationId = null, replyTo = null, persistent = true, type = null, priority = 4, groupID = null, groupSequence = 0, targetConsumerId = null, compressed = false, userID = null, content = org.apache.activemq.util.ByteSequence@79b4d0f, marshalledProperties = null, dataStructure = null, redeliveryCounter = 0, size = 0, properties = null, readOnlyProperties = true, readOnlyBody = true, droppable = false, jmsXGroupFirstForConsumer = false, text = Task9}
Received: Task9
ActiveMQTextMessage {commandId = 5, responseRequired = true, messageId = ID:INMAA1-L1005-57943-1450761507228-1:1:1:1:1, originalDestination = null, originalTransactionId = null, producerId = ID:INMAA1-L1005-57943-1450761507228-1:1:1:1, destination = queue://customerQueue, transactionId = null, expiration = 0, timestamp = 1450761507375, arrival = 0, brokerInTime = 1450761507376, brokerOutTime = 1450761525028, correlationId = null, replyTo = null, persistent = true, type = null, priority = 4, groupID = null, groupSequence = 0, targetConsumerId = null, compressed = false, userID = null, content = org.apache.activemq.util.ByteSequence@5f2050f6, marshalledProperties = org.apache.activemq.util.ByteSequence@3b81a1bc, dataStructure = null, redeliveryCounter = 0, size = 0, properties = {sequence=even}, readOnlyProperties = true, readOnlyBody = true, droppable = false, jmsXGroupFirstForConsumer = false, text = Task0}
Received: Task0
ActiveMQTextMessage {commandId = 6, responseRequired = true, messageId = ID:INMAA1-L1005-57943-1450761507228-1:1:1:1:2, originalDestination = null, originalTransactionId = null, producerId = ID:INMAA1-L1005-57943-1450761507228-1:1:1:1, destination = queue://customerQueue, transactionId = null, expiration = 0, timestamp = 1450761507380, arrival = 0, brokerInTime = 1450761507380, brokerOutTime = 1450761525028, correlationId = null, replyTo = null, persistent = true, type = null, priority = 4, groupID = null, groupSequence = 0, targetConsumerId = null, compressed = false, userID = null, content = org.apache.activemq.util.ByteSequence@64616ca2, marshalledProperties = null, dataStructure = null, redeliveryCounter = 0, size = 0, properties = null, readOnlyProperties = true, readOnlyBody = true, droppable = false, jmsXGroupFirstForConsumer = false, text = Task1}
```

## 7. 下载 Eclipse Project

这是一个关于 JMS QueueBrowser 对象的[示例](https://examples.javacodegeeks.com/wp-content/uploads/2015/12/jmsQueueBrowserExample.zip)。