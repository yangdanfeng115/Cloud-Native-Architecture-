在同一个 host 上，让两个 container 进行交互，主要涉及 网络配置，可以采用以下几种方式：

# 1. 使用 Docker 默认的 bridge 网络
原理：
Docker 默认会创建一个 bridge 网络（通常叫 bridge），所有未指定网络的容器都会加入其中。
这种方式允许同一 bridge 网络中的容器通过 IP 地址 或 容器名称（DNS 解析） 进行通信。
配置步骤：
```sh

# 启动两个容器，并加入默认 bridge 网络
docker run -d --name container1 busybox sleep 3600
docker run -d --name container2 busybox sleep 3600

# 获取 container1 的 IP 地址
docker inspect -f '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' container1

# 在 container2 中 ping container1
docker exec -it container2 ping <container1的IP>
```
更推荐的方式（基于容器名访问）：

```sh

# 启动两个容器并指定自定义网络（默认带 DNS 解析）
docker network create my_bridge
docker run -d --name container1 --network my_bridge busybox sleep 3600
docker run -d --name container2 --network my_bridge busybox sleep 3600

# 在 container2 中直接使用容器名访问 container1
docker exec -it container2 ping container1
```
✅ 适用场景： 适用于同一个 Docker 网络内部的容器通信。

# 2. 通过 host 网络（共享宿主机网络）
原理：
使用 --network host 选项时，容器 不会 拥有独立的网络命名空间，而是直接使用 宿主机的网络。
适用于需要和宿主机共享端口的情况。
配置步骤：
```sh

docker run -d --name container1 --network host busybox sleep 3600
docker run -d --name container2 --network host busybox sleep 3600

```
由于这两个容器共享宿主机网络，因此它们可以直接通过 localhost 或 宿主机 IP 进行访问。
✅ 适用场景： 适用于需要极低网络延迟的场景，但不同容器可能会端口冲突。

# 3. 通过端口映射（Port Mapping）
原理：
容器可以通过 -p HOST_PORT:CONTAINER_PORT 方式将端口映射到宿主机，使另一个容器可以通过宿主机 IP 访问它。
配置步骤：
```sh

# 启动一个 web 服务器
docker run -d --name web_container -p 8080:80 nginx

# 启动另一个容器访问 web 服务器（宿主机 IP 访问）
docker run -it --rm busybox wget -O- http://host.docker.internal:8080
```
在 Linux 上，宿主机的 IP 一般是 172.17.0.1，可以用：

```sh

docker run -it --rm busybox wget -O- http://172.17.0.1:8080

```
✅ 适用场景： 适用于外部访问，或不同 Docker 网络的容器之间的交互。

# 4. 通过 docker network 创建 overlay 网络（适用于 Swarm 或多主机通信）
原理：
适用于 多个宿主机 之间的容器通信，使用 Docker Swarm Mode 创建 overlay 网络，使不同宿主机上的容器可以相互通信。
配置步骤：
```sh

# 启动 Swarm
docker swarm init

# 创建 overlay 网络
docker network create --driver overlay my_overlay

# 运行容器并加入 overlay 网络
docker service create --name container1 --network my_overlay busybox sleep 3600
docker service create --name container2 --network my_overlay busybox sleep 3600
```
✅ 适用场景： 适用于 多主机 之间的容器通信。

# 总结

![描述](https://github.com/yangdanfeng115/Cloud-Native-Architecture-/blob/main/images/bridge.png)


# 最佳实践：

推荐使用 docker network create 创建 自定义 bridge 网络，使容器可以通过 DNS 名称 直接通信。

避免使用 --network host，除非确实需要共享网络环境。

端口映射适用于容器对外服务，但对于容器间通信，桥接网络更方便。
