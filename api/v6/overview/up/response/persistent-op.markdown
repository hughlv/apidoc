<a name="persistent-op"></a>
## 异步数据处理

在生成上传凭证时，开发者可以通过在[开发策略]()中指定`persistentOp`和`persistentNotifyUrl`字段来设置异步数据处理动作。当资源上传完成后，设置的数据处理动作就会被以异步方式启动。七牛云存储将立刻将响应内容返回给客户端，并不会等待数据处理动作完成。

关于异步数据处理相关的详细内容，请参见[处理结果持久化（pfop）]()中的相关描述。