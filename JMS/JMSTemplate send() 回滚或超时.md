# JMSTemplate send() 回滚或超时



## 课题

我们当前使用 JmsTemplate 的 send(Destination, messageCreator) 方法将消息发送到 webMethods 队列。然而，有时 send 方法需要很长时间才能返回，这是我们无法承受的，因为我们的超时应该只有 5 秒。我的问题是我们如何确保这一点？看来JmsTemplate没有发送超时。

我认为我们有一个选择是等待 send() 方法的 5 秒响应。如果超过5秒，我们将认为失败。但是，我们需要确保发送的消息(尝试发送)根本不会被处理，因为我们将认为此请求失败。我们如何做到这一点？回滚？谢谢!



## 答案

`JmsTemplate` 是核心 JMS API 的更高级别抽象。该核心 (JMS) API 没有这样的机制。

JMS 发送花费这么长时间是很不寻常的；除非您有非常大的消息和缓慢的网络。

您可以在另一个线程上处理发送，并尝试在 5 秒后中断它，但这仅在 JMS 客户端库代码可中断的情况下才有效。

但是，由于竞争条件，通常不可能可靠地执行您想要的操作。

关于java - JMSTemplate send() 回滚或超时，我们在Stack Overflow上找到一个类似的问题：

- https://stackoverflow.com/questions/22753380/