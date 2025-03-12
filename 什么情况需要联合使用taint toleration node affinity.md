在 Kubernetes 中，Taint & Toleration 和 Node Affinity 都用于控制 Pod 在节点上的调度，但它们适用于不同的场景。通常，在以下 复杂调度需求 下，需要联合使用它们：

# 1. 专用节点（Dedicated Nodes）
场景：你希望特定节点只允许特定的 Pod 运行，其他 Pod 不能调度到该节点。
解决方案：

Taint & Toleration：给节点添加 Taint，确保只有带有对应 Toleration 的 Pod 才能运行。
Node Affinity：确保特定 Pod 优先调度到该节点，而不会被调度到其他节点。

✅ 示例：运行 GPU 工作负载的节点

```bash

kubectl taint nodes gpu-node dedicated=gpu:NoSchedule

```

```yaml

spec:
  tolerations:
    - key: "dedicated"
      operator: "Equal"
      value: "gpu"
      effect: "NoSchedule"
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
          - matchExpressions:
              - key: "hardware-type"
                operator: "In"
                values:
                  - "gpu"
```

🔹 Taint & Toleration 确保只有 GPU 任务可以调度到该节点。

🔹 Node Affinity 确保 GPU 任务不会跑到非 GPU 机器上。

# 2. 预留节点（Reserve Nodes for Specific Workloads）
场景：某些节点主要用于特定任务，但如果没有足够的任务，其他 Pod 仍然可以使用这些节点。
解决方案：

Taint & Toleration（软限制）：使用 PreferNoSchedule，减少普通 Pod 调度到该节点的概率。
Node Affinity：引导特定 Pod 先尝试调度到这些节点。

✅ 示例：大数据分析任务优先调度到特定节点，但普通任务仍可用

```bash

kubectl taint nodes big-data-node dedicated=bigdata:PreferNoSchedule

```

```yaml

spec:
  tolerations:
    - key: "dedicated"
      operator: "Equal"
      value: "bigdata"
      effect: "PreferNoSchedule"
  affinity:
    nodeAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
        - weight: 100
          preference:
            matchExpressions:
              - key: "node-type"
                operator: "In"
                values:
                  - "bigdata"
```
🔹 PreferNoSchedule 允许普通 Pod 运行在该节点上，但会尽量避免。

🔹 preferredDuringSchedulingIgnoredDuringExecution 让大数据任务优先调度到这些节点。

# 3. 保护关键应用，防止意外调度
场景：关键任务（如数据库）需要运行在特定的高性能节点上，而普通任务不应该抢占这些资源。
解决方案：

Taint & Toleration：使用 NoSchedule 让非关键任务无法运行在该节点上。
Node Affinity：确保关键任务总是在这些节点上运行。

✅ 示例：数据库 Pod 只能运行在高性能节点

```bash

kubectl taint nodes db-node critical=db:NoSchedule

```
```yaml

spec:
  tolerations:
    - key: "critical"
      operator: "Equal"
      value: "db"
      effect: "NoSchedule"
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
          - matchExpressions:
              - key: "node-role"
                operator: "In"
                values:
                  - "database"
```
🔹 NoSchedule 确保非数据库 Pod 不能运行在该节点上。

🔹 Node Affinity 确保数据库 Pod 只会调度到数据库节点。

# 4. 防止 Pod 运行在不合适的节点
场景：有些 Pod 可能无法在某些节点上正常运行（例如，ARM vs. x86 兼容性问题）。
解决方案：

Taint & Toleration：给不兼容的节点打 Taint，防止 Pod 被错误地调度。
Node Affinity：确保 Pod 只被调度到正确的架构上。

✅ 示例：x86 架构的 Pod 不能调度到 ARM 节点

```bash

kubectl taint nodes arm-node arch=arm:NoSchedule

```
```yaml

spec:
  tolerations:
    - key: "arch"
      operator: "Equal"
      value: "arm"
      effect: "NoSchedule"
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
          - matchExpressions:
              - key: "kubernetes.io/arch"
                operator: "In"
                values:
                  - "amd64"
```
🔹 NoSchedule 确保 x86 任务不会跑到 ARM 节点上。

🔹 Node Affinity 让 Pod 只调度到支持 x86 的节点上。

# 5. 灾备 & 容错（High Availability & Failover）
场景：主机故障时，Pod 应该尽量调度到备用节点，而不是随意选择其他节点。
解决方案：

Taint & Toleration：给备用节点打 Taint，防止它们被普通任务占用。
Node Affinity：允许关键任务在主机故障时，优先调度到这些备用节点。

✅ 示例：生产环境 Pod 只会在主要服务器上运行，但主服务器故障时可以转移到备用服务器

```bash

kubectl taint nodes backup-node reserved=backup:NoSchedule
```

```yaml

spec:
  tolerations:
    - key: "reserved"
      operator: "Equal"
      value: "backup"
      effect: "NoSchedule"
  affinity:
    nodeAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
        - weight: 100
          preference:
            matchExpressions:
              - key: "node-role"
                operator: "In"
                values:
                  - "primary"
        - weight: 50
          preference:
            matchExpressions:
              - key: "node-role"
                operator: "In"
                values:
                  - "backup"
```
🔹 NoSchedule 防止备用节点被普通任务使用。

🔹 preferredDuringSchedulingIgnoredDuringExecution 让 Pod 优先选择主服务器，只有在主服务器不可用时才调度到备用服务器。

总结
场景	                    Taint & Toleration	             Node Affinity

专用节点（GPU、数据库等）	✅ 确保非目标任务不能调度	      ✅ 确保目标任务优先调度

预留节点（优先但非强制）	✅ PreferNoSchedule 软限制	    ✅ 让 Pod 倾向选择

关键任务防止抢占	        ✅ NoSchedule 确保 Pod 不能误入	✅ 确保 Pod 只运行在目标节点

架构兼容性（ARM vs. x86）✅ NoSchedule 避免错误调度	      ✅ 确保 Pod 只跑在正确架构上

灾备 & 容错	            ✅ 备用节点不被普通任务占用	      ✅ 确保关键任务优先调度到主服务器

💡 最佳实践：Taint 控制 Pod 不能 去哪里，Node Affinity 控制 Pod 应该 去哪里。两者结合可以实现更灵活和精准的 Pod 调度策略。🚀
