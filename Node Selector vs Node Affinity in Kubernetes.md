Node Selector vs Node Affinity in Kubernetes

Both Node Selector and Node Affinity are used to control which nodes a pod can be scheduled on based on node labels. However, Node Affinity provides more flexibility than Node Selector.

# 1. Node Selector (Simple & Basic)

nodeSelector is the simplest way to assign pods to specific nodes using key-value label matching.

Example Usage
If a node has this label:

```sh

kubectl label nodes worker-node1 env=production

```

You can schedule a pod only on this node by adding nodeSelector in the pod spec:

```yaml

apiVersion: v1
kind: Pod
metadata:
  name: example-pod
spec:
  nodeSelector:
    env: production
  containers:
    - name: nginx
      image: nginx
Limitations of Node Selector
```
Only supports exact key-value matching (no advanced rules).
No soft constraints (if no matching node exists, the pod remains unscheduled).
Cannot handle multiple conditions (e.g., OR/AND logic).

# 2. Node Affinity (Advanced & Flexible)

Node Affinity is a more powerful version of nodeSelector. It allows complex rules using operators (like In, NotIn, Exists) and soft constraints.

Two Types of Node Affinity
Required (Hard Constraint) - requiredDuringSchedulingIgnoredDuringExecution

Works like nodeSelector but supports multiple conditions and advanced rules.
If no matching node is found, the pod remains unscheduled.
Preferred (Soft Constraint) - preferredDuringSchedulingIgnoredDuringExecution

Tries to place the pod on a preferred node but won't block scheduling if unavailable.
Example: Hard Constraint (Required)
This pod must be scheduled on nodes with env=production OR env=staging:

```yaml

apiVersion: v1
kind: Pod
metadata:
  name: required-affinity-pod
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
          - matchExpressions:
              - key: "env"
                operator: "In"
                values:
                  - "production"
                  - "staging"
  containers:
    - name: nginx
      image: nginx
```

🔹 If no node matches, the pod stays in a pending state.

Example: Soft Constraint (Preferred)
This pod prefers nodes with env=production but can run anywhere if necessary:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: preferred-affinity-pod
spec:
  affinity:
    nodeAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
        - weight: 10
          preference:
            matchExpressions:
              - key: "env"
                operator: "In"
                values:
                  - "production"
  containers:
    - name: nginx
      image: nginx
```
🔹 If a production node is available, it will be chosen, but if not, the pod can run elsewhere.

# 3. Key Differences
Feature	           Node Selector	              Node Affinity
Complex Rules	       ❌ No	                    ✅ Yes (In, NotIn, Exists)
Soft Constraints	   ❌ No	                    ✅ Yes (Preferred Scheduling)
Multiple Conditions	 ❌ No	                    ✅ Yes (AND/OR logic)
Fallback Option	     ❌ No (Hard restriction) 	✅ Yes (Preferred mode)
Granularity	         🔹 Simple matching	        🔹 More detailed control

# When to Use Which?
Use Node Selector 🟢 when you just need a simple, hard rule (key=value).
Use Node Affinity 🔵 when you need flexibility, soft preferences, or advanced rules.
