Kubernetes Taints and Tolerations

Taints and tolerations in Kubernetes work together to prevent pods from being scheduled onto specific nodes unless they explicitly allow it.

# 1. Taints (Applied to Nodes)
A taint is applied to a node to indicate that only certain pods should be scheduled on it. Taints have three parts:

<key>=<value>:<effect>
Key: A label-like identifier.
Value: An optional string.
Effect: Determines how the taint works. Possible values:
NoSchedule → New pods without a matching toleration won't be scheduled on this node.
PreferNoSchedule → Kubernetes tries to avoid scheduling pods here but allows it if necessary.
NoExecute → Existing pods without a matching toleration will be evicted.

## Adding a Taint to a Node

kubectl taint nodes <node-name> key=value:NoSchedule

kubectl taint nodes worker-node environment=production:NoSchedule

This means only pods with a matching toleration can run on worker-node.

Removing a Taint

kubectl taint nodes <node-name> key=value:NoSchedule-
Example:

kubectl taint nodes worker-node environment=production:NoSchedule-

# 2. Tolerations (Applied to Pods)
A toleration allows a pod to be scheduled on a node with a matching taint.

Adding a Toleration to a Pod
Tolerations are defined in the pod specification (.spec.tolerations).

Example:

apiVersion: v1
kind: Pod
metadata:
  name: toleration-example
spec:
  tolerations:
    - key: "environment"
      operator: "Equal"
      value: "production"
      effect: "NoSchedule"
  containers:
    - name: my-app
      image: nginx

This pod can be scheduled on nodes with the taint environment=production:NoSchedule.
Toleration Operators
Equal → Matches the key and value exactly.
Exists → Ignores the value and matches only the key.
Example using Exists:


tolerations:
  - key: "environment"
    operator: "Exists"
    effect: "NoSchedule"
This allows the pod to run on any node with the environment taint, regardless of value.

# 3. Taints and Tolerations Together
Taints restrict nodes by preventing unwanted pods from running on them.
Tolerations allow pods to bypass taints and be scheduled on these nodes.

# The PreferNoSchedule effect 
The PreferNoSchedule effect in Kubernetes is a soft restriction on scheduling. It means that Kubernetes will try to avoid placing pods on a node with this taint, but it will still schedule them there if no better options are available.

How It Works
If a node has a PreferNoSchedule taint, Kubernetes prefers not to place pods there.
However, if no other nodes are available or if other scheduling constraints force it, Kubernetes can still schedule pods on that node.
Example Scenario
## 1. Apply a Taint with PreferNoSchedule

kubectl taint nodes node1 dedicated=database:PreferNoSchedule
Now, Kubernetes prefers not to schedule any pod on node1 unless there are no better options.

## 2. Pod Without a Matching Toleration
If a pod does not have a toleration for dedicated=database:PreferNoSchedule, Kubernetes will try to schedule it on other nodes.

If there are available nodes without this taint, the pod will go there.
If all other nodes are full, Kubernetes may still place the pod on node1 as a last resort.
## 3. Pod With a Matching Toleration
If a pod has a matching toleration, it can be scheduled on node1 without restrictions:

tolerations:
  - key: "dedicated"
    operator: "Equal"
    value: "database"
    effect: "PreferNoSchedule"
Now, the pod will not be blocked from running on node1.
