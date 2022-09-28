配置yaml
```yaml
rocketmq:
  name-server: 127.0.0.1:9876
  producer:
    group: lili_group
    send-message-timeout: 30000
```



生产者 发送 topic + tags  (Topic是消息的一级分类，Tag是消息的二级分类。)

```java
/**
 * 发送售后消息
 *
 * @param afterSale 售后对象
 */
private void sendAfterSaleMessage(AfterSale afterSale) {
    //发送售后创建消息
    String destination = rocketmqCustomProperties.getAfterSaleTopic() + ":" + AfterSaleTagsEnum.AFTER_SALE_STATUS_CHANGE.name();
    //发送订单变更mq消息
    rocketMQTemplate.asyncSend(destination, JSONUtil.toJsonStr(afterSale), RocketmqSendCallbackBuilder.commonCallback());
}
```

消费者 定义消息监听器实现RocketMQListener接口接收消息 根据topic和consumerGroup（consumerGroup是RocketMQ的组）

```java
@RocketMQMessageListener(topic = "${lili.data.rocketmq.after-sale-topic}", consumerGroup = "${lili.data.rocketmq.after-sale-group}")
public class AfterSaleMessageListener implements RocketMQListener<MessageExt> {
	@Override
    public void onMessage(MessageExt messageExt) {
        if (AfterSaleTagsEnum.valueOf(messageExt.getTags()) == AfterSaleTagsEnum.AFTER_SALE_STATUS_CHANGE) {
        	...
        }
    }
}
```

重写onMessage方法

 根据不用的tags处理不同的业务逻辑