如果 Host A 在一个网络中，而 Host B 在另一个网络中，它们想要相互通信，需要一个 路由器（Router） 进行数据转发。这种情况通常涉及 子网（Subnet） 或 不同的网络（如不同的局域网 LAN 或广域网 WAN）。网络结构通常如下：

# 1. 基本的网络结构

[Host A] --- [Router A] --- (Internet / 广域网) --- [Router B] --- [Host B]
或者：

[Host A] --- [Switch / 交换机] --- [Router] --- (不同网段) --- [Switch] --- [Host B]

# 2. 详细的网络工作原理
Host A 发送数据

Host A 发现目标 IP 地址（Host B）不在自己的子网内，于是将数据包发给它的默认网关（Router A）。
Router A 进行路由转发

Router A 查找路由表，决定如何转发数据（可能通过 Internet、专线或 VPN）。
数据包经过多个路由器传输，直到到达 Router B。
Router B 解析并转发数据

Router B 收到数据包后，检查目的 IP 是否属于它的内部网络。
如果是，则通过交换机或直连方式将数据包传给 Host B。
Host B 响应

Host B 发送响应数据，数据包沿着相同或不同的路径返回 Host A。
# 3. 网络中的 IP 地址
每个设备在不同的网络中会有不同的 IP 地址。例如：

Host A: 192.168.1.100（子网：192.168.1.0/24）
Router A: 192.168.1.1（子网网关）
Router A（外网接口）: 200.100.50.1（公网 IP）
Router B（外网接口）: 200.100.50.2（公网 IP）
Router B: 192.168.2.1（子网网关）
Host B: 192.168.2.100（子网：192.168.2.0/24）

