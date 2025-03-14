下面是一张简单的网络拓扑图，展示了交换机（Switch） 连接两台主机（Host A 和 Host B）的交互过程。

# 网络拓扑结构

  ![网络拓扑图](https://github.com/yangdanfeng115/Cloud-Native-Architecture-/blob/main/images/mac.png)
 

# 数据交互过程
假设 Host A（192.168.1.10, MAC A）想要给 Host B（192.168.1.20, MAC B）发送数据，交换机是如何处理的呢？

## 步骤 1：Host A 发送数据
Host A 发送一个数据帧，目标是 Host B，数据包包含：

源 MAC 地址：MAC A（Host A）

目标 MAC 地址：MAC B（Host B）

源 IP 地址：192.168.1.10

目标 IP 地址：192.168.1.20


## 步骤 2：交换机学习 MAC 地址

交换机收到 Host A 发来的数据包，并检查其 源 MAC 地址（MAC A）。

交换机会将 MAC A 记录到 MAC 地址表中，记住 "MAC A 在某个端口"。


## 步骤 3：交换机查找目标 MAC

交换机查看自己的 MAC 地址表，查找MAC B 是否已经存在：

如果 MAC B 在表中（Host B 之前发送过数据）：

交换机直接将数据帧转发到连接 Host B 的端口。

如果 MAC B 不在表中（第一次通信）：

交换机进行 广播（Flooding），即把数据发送到所有端口（除了 Host A 所在的端口）。

Host B 收到数据并响应，交换机学习到 MAC B 在哪个端口，更新 MAC 地址表。

## 步骤 4：Host B 响应

Host B 收到数据包后，回发一个应答数据包给 Host A。

交换机记录 Host B 的 MAC 地址（MAC B），并将返回数据正确转发给 Host A。

MAC 地址表（示例）

在交换机完成学习后，它的 MAC 地址表可能如下：


MAC 地址	端口号

MAC A	Port 1

MAC B	Port 2

这样，下次 Host A 和 Host B 直接通信时，交换机不会再广播，而是直接转发，提高网络效率。

# 总结

交换机通过 MAC 地址表高效转发数据，不像 Hub 那样广播所有数据。

第一次通信时，交换机会广播，之后会直接转发，优化网络性能。

如果 Host B 没有回应，交换机会丢弃数据包，避免网络堵塞。
