# 遗嘱消息

MQTT 遗嘱消息可以在客户端意外断线时将“遗嘱”优雅地发送给第三方订阅者，以实现离线通知、设备状态更新等业务。其中意外断线指客户端断开前未向服务器发送 DISCONNECT 消息，比如：

- 因网络故障或网络波动，设备在保持连接周期内未能通讯，连接被服务端关闭
- 设备意外掉电
- 设备尝试进行不被允许的操作而被服务端关闭连接，例如订阅自身权限以外的主题等

遗嘱消息在 MQTT 客户端向服务器端 CONNECT 请求时设置，可选属性包括是否发送遗嘱消息 (Will Message)标志，和遗嘱消息主题 (Topic) 与内容(Payload) 以及 Properties。

值得一提的，遗嘱消息发布的时间可能会有延迟：通常意外断线时，服务器无法立即检测到断线行为，需要通过连接保活心跳机制并经过一定周期后才会触发；MQTT 5.0 提供的遗嘱延迟间隔（Will Delay Interval）属性也会影响发布时间。

::: tip
EMQX 中可选在某些场景下使用 [规则引擎 - 客户端断开事件](../data-integration/rules.md) 替代遗嘱消息，以更灵活的方式处理客户端离线事件。
:::

::: tip

更多有关 MQTT 遗嘱消息机制内容请参考：

- [MQTT 遗嘱消息（Will Message）的使用](https://www.emqx.com/zh/blog/use-of-mqtt-will-message)
:::