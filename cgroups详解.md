🚀 cgroups（Control Groups）详解
# 1. 什么是 cgroups？
cgroups（Control Groups，控制组） 是 Linux 内核的一项资源管理机制，用于限制、隔离和统计进程（tasks）的CPU、内存、I/O、网络等资源。

📌 简单来说：

它允许你限制某个进程最多只能使用 1GB 内存，或者最多占用 50% 的 CPU，从而防止某些进程占用过多资源。
它是 Docker、Kubernetes、LXC 等容器技术的核心基础，可以隔离不同的进程组，让它们在共享内核的同时互不干扰。
# 2. cgroups 的主要功能
功能	      作用	                        示例
CPU 限制	  限制进程可使用的 CPU 计算资源	限制某进程最多使用 50% 的 CPU
内存限制	    设定进程的最大内存占用	      限制进程最多使用 512MB 内存
I/O 限制	  控制磁盘读写速率	            限制某个进程最多 10MB/s 读写
网络带宽限制	限制进程的网络流量	          限制某进程最多 1Mbps
进程数量限制	限制某个组中的进程数量	      限制某个 cgroup 只能创建 100 个进程
设备访问控制	允许/禁止进程访问某些设备	    禁止某个进程访问 /dev/sda

# 3. cgroups 的版本
Linux 支持两个主要版本的 cgroups：

版本	特点	适用场景
cgroups v1	每个子系统（如 CPU、内存）可以独立管理	传统 Docker、LXC
cgroups v2	统一的层级结构，更强的资源共享	现代 Linux（默认），Kubernetes
📌 区别：

cgroups v1：每个资源控制器（如 CPU、内存）有独立的层级，管理复杂。
cgroups v2：所有资源控制器使用统一层级，更易管理，支持更精细的 QoS 控制。
⚠️ 当前 Linux 发行版（如 Ubuntu 22.04、CentOS 9）默认使用 cgroups v2，但 Docker 仍默认运行在 cgroups v1，可手动切换。

# 4. cgroups 的基本结构
cgroups 采用分层结构，每个层级下可以有多个进程组（cgroup），每个 cgroup 都可以管理特定进程的资源。

文件路径（cgroups v1）：

/sys/fs/cgroup/
其中包含：

cpu/（CPU 限制）
memory/（内存限制）
blkio/（I/O 限制）
cpuset/（CPU 绑定）
devices/（设备访问控制）
freezer/（暂停进程）

/sys/fs/cgroup/cpu/my_cgroup/       # my_cgroup 组的 CPU 配置
/sys/fs/cgroup/memory/my_cgroup/    # my_cgroup 组的内存配置
# 5. 使用 cgroups（v1）
我们可以手动创建 cgroups 并限制进程的资源：

(1) 创建 cgroup

mkdir /sys/fs/cgroup/memory/my_cgroup
(2) 限制内存使用
bash
Copy
Edit
echo 500M > /sys/fs/cgroup/memory/my_cgroup/memory.limit_in_bytes
(3) 将某个进程加入 cgroup
假设某个进程的 PID 为 1234：

echo 1234 > /sys/fs/cgroup/memory/my_cgroup/cgroup.procs
✅ 该进程现在最多只能使用 500MB 内存。

(4) 限制 CPU 使用
bash
Copy
Edit
mkdir /sys/fs/cgroup/cpu/my_cgroup
echo 50000 > /sys/fs/cgroup/cpu/my_cgroup/cpu.cfs_quota_us   # 限制最多使用 50% CPU
echo 100000 > /sys/fs/cgroup/cpu/my_cgroup/cpu.cfs_period_us # 100ms 计算周期
(5) 限制 I/O 速率

mkdir /sys/fs/cgroup/blkio/my_cgroup
echo "8:0 10485760" > /sys/fs/cgroup/blkio/my_cgroup/blkio.throttle.read_bps_device  # 10MB/s 读速率
# 6. Docker 与 cgroups
**Docker 容器就是使用 cgroups + Namespace 进行资源隔离的：**

docker run --memory="500m" --cpus="0.5" ubuntu
Docker 其实是在 /sys/fs/cgroup/ 目录下自动创建 cgroups 来管理容器进程。

查看某个容器的 cgroups 配置：

docker ps
docker inspect --format '{{ .State.Pid }}' <container_id>
cat /sys/fs/cgroup/memory/docker/<container_id>/memory.limit_in_bytes
# 7. Kubernetes（K8s）与 cgroups
Kubernetes 也是基于 cgroups 来限制 Pod 资源：

resources:
  limits:
    memory: "512Mi"
    cpu: "0.5"
  requests:
    memory: "256Mi"
    cpu: "0.2"
Kubernetes 通过 CRI（Container Runtime Interface） 与 cgroups 交互，常见的容器运行时（如 containerd、CRI-O）都会用到 cgroups。

# 8. cgroups v2（更现代的管理方式）
cgroups v2 提供了更简洁的控制方式，所有资源控制器共享一个层级：

/sys/fs/cgroup/unified/
创建一个 cgroup：

mkdir /sys/fs/cgroup/mygroup
echo "+memory" > /sys/fs/cgroup/mygroup/cgroup.subtree_control
echo 500M > /sys/fs/cgroup/mygroup/memory.max
echo $$ > /sys/fs/cgroup/mygroup/cgroup.procs  # 把当前进程加入 cgroup
✅ 现在当前进程最多只能使用 500MB 内存。

切换到 cgroups v2（Ubuntu 22.04）

mount -t cgroup2 none /sys/fs/cgroup
# 9. cgroups 的优势
✅ 精细化控制：可以对进程进行精准的资源限制，如 CPU 限制到 0.5 核。
✅ 隔离性：不同 cgroup 之间的进程不会相互干扰。
✅ 性能优化：可防止进程耗尽 CPU 或内存，提高系统稳定性。
✅ 容器化基础：Docker、K8s、LXC 等技术都依赖 cgroups 进行资源管理。

# 10. 总结
cgroups 是 Linux 内核提供的资源管理机制，可用于限制进程的 CPU、内存、I/O、网络 资源。
cgroups v1 使用多个独立的子系统，cgroups v2 采用统一的层级结构，更简洁。
Docker、Kubernetes 都依赖 cgroups 进行资源限制，从而实现容器的隔离和管理。
手动使用 cgroups 可以创建自定义资源限制规则，但大部分情况下 Docker/K8s 会自动处理。
