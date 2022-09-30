

# SpringBoot中@JmsListener注解属性destination支持配置化

## 一、yml中配置队列名称

自己适配**.properties**

```properties
ibmmq:
  queue-test: TEST_QUEUE   # 队列名称
```

## 二、注入配置读取类

```java
import lombok.Data;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;

/**
 * 队列名称配置
 * 这里切记要@Data，或手动set和get
 */
@Component
@Data
public class QueueNameConfig {

	@Value("${ibmmq.queue-test}")
	private String testQueue;

}
```

注意：必须要@Data，或手动set和get

## 三、@JmsListener消费者

```java
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.jms.annotation.JmsListener;
import org.springframework.jms.listener.adapter.MessageListenerAdapter;
import org.springframework.stereotype.Component;

import javax.jms.Message;
import javax.jms.TextMessage;

/**
 * IBM MQ消费者
 */
@Component
@Slf4j
public class ReceiveMessage extends MessageListenerAdapter {

	/**
	 * destination：监听的队列名称，使用SpEL表达式写入
	 * containerFactory：监听的工厂类，为配置类中所配置的名字
	 *
	 * @param message
	 */
	@Override
	@JmsListener(destination = "#{@queueNameConfig.testQueue}", containerFactory = "jmsListenerContainerFactory")
	public void onMessage(Message message) {
		TextMessage textMessage = (TextMessage) message;  //转换成文本消息
		try {
			String text = textMessage.getText();
			log.info("接收信息：{}", text);
		} catch (Exception e) {
			e.printStackTrace();
		}
	}
}
```
