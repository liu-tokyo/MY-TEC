# javax.jms.BytesMessage.reset()方法的使用及代码示例

本文整理了Java中`javax.jms.BytesMessage.reset()`方法的一些代码示例，展示了`BytesMessage.reset()`的具体用法。这些代码示例主要来源于`Github`/`Stackoverflow`/`Maven`等平台，是从一些精选项目中提取出来的代码，具有较强的参考意义，能在一定程度帮忙到你。`BytesMessage.reset()`方法的具体详情如下：
包路径：javax.jms.BytesMessage
类名称：BytesMessage
方法名：reset

### BytesMessage.reset介绍

[英]Puts the message body in read-only mode and repositions the stream of bytes to the beginning.
[中]将消息正文置于只读模式，并将字节流重新定位到开头。

代码示例来源：[origin: spring-projects/spring-framework](https://www.saoniuhuo.com/link?url=https://www.tabnine.com/web/assistant/code/rs/5c789024df79be0001ec64dc#L474)

```
@Override
protected Object extractPayload(Message message) throws JMSException {
	Object payload = extractMessage(message);
	if (message instanceof BytesMessage) {
		try {
			// In case the BytesMessage is going to be received as a user argument:
			// reset it, otherwise it would appear empty to such processing code...
			((BytesMessage) message).reset();
		}
		catch (JMSException ex) {
			// Continue since the BytesMessage typically won't be used any further.
			logger.debug("Failed to reset BytesMessage after payload extraction", ex);
		}
	}
	return payload;
}
```

代码示例来源：[origin: wildfly/wildfly](https://www.saoniuhuo.com/link?url=https://github.com/wildfly/wildfly/tree/master/client/shade/src/main/java/org/apache/activemq/artemis/jms/client/ActiveMQBytesMessage.java#L84)

```
/**
* Foreign message constructor
*/
public ActiveMQBytesMessage(final BytesMessage foreign, final ClientSession session) throws JMSException {
	super(foreign, ActiveMQBytesMessage.TYPE, session);
	foreign.reset();
	byte[] buffer = new byte[1024];
	int n = foreign.readBytes(buffer);
	while (n != -1) {
		writeBytes(buffer, 0, n);
		n = foreign.readBytes(buffer);
	}
}
```

代码示例来源：[origin: wildfly/wildfly](https://www.saoniuhuo.com/link?url=https://github.com/wildfly/wildfly/tree/master/client/shade/src/main/java/org/apache/activemq/artemis/jms/client/ActiveMQMessageProducer.java#L578)

```
@Override
public void sendAcknowledged(org.apache.activemq.artemis.api.core.Message clientMessage) {
	if (jmsMessage instanceof StreamMessage) {
		try {
			((StreamMessage) jmsMessage).reset();
		} catch (JMSException e) {
			// HORNETQ-1209 XXX ignore?
		}
	}
	if (jmsMessage instanceof BytesMessage) {
		try {
			((BytesMessage) jmsMessage).reset();
		} catch (JMSException e) {
			// HORNETQ-1209 XXX ignore?
		}
	}
	try {
		producer.connection.getThreadAwareContext().setCurrentThread(true);
		completionListener.onCompletion(jmsMessage);
	} finally {
		producer.connection.getThreadAwareContext().clearCurrentThread(true);
	}
}
```

代码示例来源：[origin: apache/activemq](https://www.saoniuhuo.com/link?url=https://github.com/apache/activemq/tree/master/activemq-client/src/main/java/org/apache/activemq/ActiveMQMessageTransformation.java#L66)

```
bytesMsg.reset();
ActiveMQBytesMessage msg = new ActiveMQBytesMessage();
msg.setConnection(connection);
```

代码示例来源：[origin: apache/nifi](https://www.saoniuhuo.com/link?url=https://github.com/apache/nifi/tree/master/nifi-nar-bundles/nifi-standard-bundle/nifi-standard-processors/src/test/java/org/apache/nifi/processors/standard/TestJmsConsumer.java#L133)

```
/**
 * Test BytesMessage to FlowFile conversion
 *
 * @throws java.lang.Exception ex
 */
@Test
public void testMap2FlowFileBytesMessage() throws Exception {
	TestRunner runner = TestRunners.newTestRunner(GetJMSQueue.class);
	BytesMessage bytesMessage = new ActiveMQBytesMessage();
	String sourceString = "Apache NiFi is an easy to use, powerful, and reliable system to process and distribute data.!";
	byte[] payload = sourceString.getBytes("UTF-8");
	bytesMessage.writeBytes(payload);
	bytesMessage.reset();
	ProcessContext context = runner.getProcessContext();
	ProcessSession session = runner.getProcessSessionFactory().createSession();
	ProcessorInitializationContext pic = new MockProcessorInitializationContext(runner.getProcessor(), (MockProcessContext) runner.getProcessContext());
	JmsProcessingSummary summary = JmsConsumer.map2FlowFile(context, session, bytesMessage, true, pic.getLogger());
	assertEquals("BytesMessage content length should equal to FlowFile content size", payload.length, summary.getLastFlowFile().getSize());
	final byte[] buffer = new byte[payload.length];
	runner.clearTransferState();
	session.read(summary.getLastFlowFile(), new InputStreamCallback() {
		@Override
		public void process(InputStream in) throws IOException {
		StreamUtils.fillBuffer(in, buffer, false);
		}
	});
	String contentString = new String(buffer, "UTF-8");
	assertEquals("", sourceString, contentString);
}
```

代码示例来源：[origin: org.apache.tomee/openejb-core](https://www.saoniuhuo.com/link?url=https://www.tabnine.com/web/assistant/code/rs/5c65ea0c1095a500018dbb5f#L180)

```
@Override
	...
	public void reset() throws JMSException {
		message.reset();
	}
}
```

代码示例来源：[origin: org.jboss.jbossas/jboss-as-connector](https://www.saoniuhuo.com/link?url=https://www.tabnine.com/web/assistant/code/rs/5c6572501095a5000150bd7a#L116)

```
public void reset() throws JMSException
{
	((BytesMessage) message).reset();
}
```

代码示例来源：[origin: org.apache.qpid/qpid-jca](https://www.saoniuhuo.com/link?url=https://www.tabnine.com/web/assistant/code/rs/5c6671231095a50001c42404#L267)

```
/**
* Reset
* @exception JMSException Thrown if an error occurs
*/
public void reset() throws JMSException
{
	if (_log.isTraceEnabled())
	{
		_log.trace("reset()");
	}
	((BytesMessage)_message).reset();
}
```

代码示例来源：[origin: stackoverflow.com](https://www.saoniuhuo.com/link?url=http://stackoverflow.com/a/39560732#0#L0)

```
if (message instanceof BytesMessage){
	BytesMessage byteMessage = (BytesMessage) message;
	byte[] byteData = null;
	byteData = new byte[(int) byteMessage.getBodyLength()];
	byteMessage.readBytes(byteData);
	byteMessage.reset();
	stringMessage =  new String(byteData);
}
```

代码示例来源：[origin: apache/activemq-artemis](https://www.saoniuhuo.com/link?url=https://github.com/apache/activemq-artemis/tree/master/artemis-ra/src/main/java/org/apache/activemq/artemis/ra/ActiveMQRABytesMessage.java#L255)

```
/**
* Reset
*
* @throws JMSException Thrown if an error occurs
*/
@Override
public void reset() throws JMSException {
	if (ActiveMQRALogger.LOGGER.isTraceEnabled()) {
		ActiveMQRALogger.LOGGER.trace("reset()");
	}
	((BytesMessage) message).reset();
}
```

代码示例来源：[origin: org.apache.activemq/artemis-ra](https://www.saoniuhuo.com/link?url=https://www.tabnine.com/web/assistant/code/rs/5c6569451095a500014bde88#L255)

```
/**
* Reset
*
* @throws JMSException Thrown if an error occurs
*/
@Override
public void reset() throws JMSException {
	if (ActiveMQRALogger.LOGGER.isTraceEnabled()) {
		ActiveMQRALogger.LOGGER.trace("reset()");
	}
	((BytesMessage) message).reset();
}
```

代码示例来源：[origin: org.apache.axis2/axis2-transport-jms](https://www.saoniuhuo.com/link?url=https://www.tabnine.com/web/assistant/code/rs/5c655ccf22c5bc0001ddff7b#L58)

```
public InputStream getInputStream() throws IOException {
	try {
		message.reset();
	} catch (JMSException ex) {
		throw new JMSExceptionWrapper(ex);
	}
	return new BytesMessageInputStream(message);
}
```

代码示例来源：[origin: org.hornetq/hornetq-jms](https://www.saoniuhuo.com/link?url=https://www.tabnine.com/web/assistant/code/rs/5c658f421095a5000160b3f4#L73)

```
public HornetQBytesMessage(final BytesMessage foreign, final ClientSession session) throws JMSException
{
	super(foreign, HornetQBytesMessage.TYPE, session);
	foreign.reset();
	byte[] buffer = new byte[1024];
	int n = foreign.readBytes(buffer);
	while (n != -1)
	{
		writeBytes(buffer, 0, n);
		n = foreign.readBytes(buffer);
	}
}
```

代码示例来源：[origin: org.apache.qpid/qpid-amqp-1-0-client-jms](https://www.saoniuhuo.com/link?url=https://www.tabnine.com/web/assistant/code/rs/5c655bb122c5bc0001dd6b3e#L807)

```
private MessageImpl convertBytesMessage(final BytesMessage message) throws JMSException
{
	BytesMessageImpl bytesMessage = createBytesMessage();
	message.reset();
	byte[] buf = new byte[1024];
	int len;
	while ((len = message.readBytes(buf)) != -1)
	{
		bytesMessage.writeBytes(buf, 0, len);
	}
	return bytesMessage;
}
```

代码示例来源：[origin: skyscreamer/nevado](https://www.saoniuhuo.com/link?url=https://github.com/skyscreamer/nevado/tree/master/nevado-jms/src/main/java/org/skyscreamer/nevado/jms/message/NevadoBytesMessage.java#L36)

```
protected NevadoBytesMessage(BytesMessage message) throws JMSException {
	super(message);
	message.reset();
	for(int count = 0 ; count < message.getBodyLength() ; ) {
		byte[] buffer = new byte[10240];
		int numRead = message.readBytes(buffer);
		writeBytes(buffer, 0, numRead);
		count += numRead;
	}
}
```

代码示例来源：[origin: apache/activemq-artemis](https://www.saoniuhuo.com/link?url=https://github.com/apache/activemq-artemis/tree/master/artemis-jms-client-all/src/main/java/org/apache/activemq/artemis/jms/client/ActiveMQBytesMessage.java#L84)

```
/**
* Foreign message constructor
*/
public ActiveMQBytesMessage(final BytesMessage foreign, final ClientSession session) throws JMSException {
	super(foreign, ActiveMQBytesMessage.TYPE, session);
	foreign.reset();
	byte[] buffer = new byte[1024];
	int n = foreign.readBytes(buffer);
	while (n != -1) {
		writeBytes(buffer, 0, n);
		n = foreign.readBytes(buffer);
	}
}
```

代码示例来源：[origin: org.apache.activemq/artemis-jms-client-all](https://www.saoniuhuo.com/link?url=https://www.tabnine.com/web/assistant/code/rs/5c666bc61095a50001c28827#L84)

```
/**
* Foreign message constructor
*/
public ActiveMQBytesMessage(final BytesMessage foreign, final ClientSession session) throws JMSException {
	super(foreign, ActiveMQBytesMessage.TYPE, session);
	foreign.reset();
	byte[] buffer = new byte[1024];
	int n = foreign.readBytes(buffer);
	while (n != -1) {
		writeBytes(buffer, 0, n);
		n = foreign.readBytes(buffer);
	}
}
```

代码示例来源：[origin: org.jboss.eap/wildfly-client-all](https://www.saoniuhuo.com/link?url=https://www.tabnine.com/web/assistant/code/rs/5c65afb31095a5000170c04b#L84)

```
/**
* Foreign message constructor
*/
public ActiveMQBytesMessage(final BytesMessage foreign, final ClientSession session) throws JMSException {
	super(foreign, ActiveMQBytesMessage.TYPE, session);
	foreign.reset();
	byte[] buffer = new byte[1024];
	int n = foreign.readBytes(buffer);
	while (n != -1) {
		writeBytes(buffer, 0, n);
		n = foreign.readBytes(buffer);
	}
}
```

代码示例来源：[origin: apache/activemq-artemis](https://www.saoniuhuo.com/link?url=https://github.com/apache/activemq-artemis/tree/master/artemis-jms-client/src/main/java/org/apache/activemq/artemis/jms/client/ActiveMQBytesMessage.java#L84)

```
/**
* Foreign message constructor
*/
public ActiveMQBytesMessage(final BytesMessage foreign, final ClientSession session) throws JMSException {
 super(foreign, ActiveMQBytesMessage.TYPE, session);
	foreign.reset();
	byte[] buffer = new byte[1024];
	int n = foreign.readBytes(buffer);
	while (n != -1) {
		writeBytes(buffer, 0, n);
		n = foreign.readBytes(buffer);
	}
}
```

代码示例来源：[origin: apache/activemq-artemis](https://www.saoniuhuo.com/link?url=https://github.com/apache/activemq-artemis/tree/master/tests/jms-tests/src/test/java/org/apache/activemq/artemis/jms/tests/message/MessageHeaderTest.java#L656)

```
@Test
public void testCopyOnForeignBytesMessage() throws JMSException {
	ClientMessage clientMessage = new ClientMessageImpl(ActiveMQTextMessage.TYPE, true, 0, System.currentTimeMillis(), (byte) 4, 1000);
	ClientSession session = new FakeSession(clientMessage);
	BytesMessage foreignBytesMessage = new SimpleJMSBytesMessage();
	for (int i = 0; i < 20; i++) {
		foreignBytesMessage.writeByte((byte) i);
	}
	ActiveMQBytesMessage copy = new ActiveMQBytesMessage(foreignBytesMessage, session);
	foreignBytesMessage.reset();
	copy.reset();
	MessageHeaderTestBase.ensureEquivalent(foreignBytesMessage, copy);
}
```

