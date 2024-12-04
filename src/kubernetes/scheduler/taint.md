# Taint and Tolerations


> **Node affinity** is a property of Pods that attracts them to a set of nodes (either as a preference or a hard requirement). Taints are a property of nodes that repel pods from those nodes.

> **Taints** are the opposite -- they allow a node to repel a set of pods.

> **Tolerations** are applied to pods. Tolerations allow the scheduler to schedule pods with matching taints.
> Tolerations allow scheduling but don't guarantee scheduling: the scheduler also evaluates other parameters as part of its function.

> Node only accepts pods with matching tolerations.
 
<br>

```basd
# Add a taint to a node
kubectl taint nodes node1 colour=blue:NoSchedule

# Remove a taint from a node
kubectl taint nodes node1 colour:NoSchedule-
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    env: test
spec:
  containers:
  - name: nginx
    image: nginx
    imagePullPolicy: IfNotPresent
  tolerations:
  - key: "colour"
    operator: "Exists"
    effect: "NoSchedule"
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    env: test
spec:
  containers:
  - name: nginx
    image: nginx
    imagePullPolicy: IfNotPresent
  tolerations:
  - key: "colour"
    operator: "Equal"
    value: "blue"
    effect: "NoSchedule"
```

## Taint Effects
1. <code>NoSchedule</code>
- Pods without matching tolerations will not be scheduled on the node.

2. <code>PreferNoSchedule</code>
- Kubernetes will try to avoid scheduling pods without matching tolerations on the node but will not strictly enforce it.

3. <code>NoExecute</code>
- Currently running pods without matching tolerations will be evicted from the node.


> **Note**
> 
> - The operator field in the toleration can be set to <code>Equal</code> or <code>Exists</code>.
> - The <code>Equal</code> operator matches the value of the taint key. The <code>Exists</code> operator matches any value of the taint key.
> - An empty effect matches all effects with keys that match the toleration.


## Taints created by node-controller

When certain events occur on a node, the node-controller adds or removes some taints on the node.

- <code>node.kubernetes.io/not-ready</code>: Node is not ready. This corresponds to the NodeCondition Ready being "False".
- <code>node.kubernetes.io/unreachable</code>: Node is unreachable from the node controller. This corresponds to the NodeCondition Ready being "Unknown".
- <code>node.kubernetes.io/memory-pressure</code>: Node has memory pressure.
- <code>node.kubernetes.io/disk-pressure</code>: Node has disk pressure. 
- <code>node.kubernetes.io/pid-pressure</code>: Node has PID pressure.
- <code>node.kubernetes.io/network-unavailable</code>: Node's network is unavailable.
- <code>node.kubernetes.io/unschedulable</code>: Node is unschedulable.

> For <code>node.kubernetes.io/not-ready</code> and <code>node.kubernetes.io/unreachable</code>, the node controller adds <code>NoExecute</code> effect by default, so pods will be evicted from those nodes.

The scheduler checks taint (not node condition) when it makes scheduling decisions,
so node-controller or kubelet based on node conditions set different taints on nodes.

### <code>tolerationSeconds</code> 

- By default, kubernetes add two tolerations to pods: <code>node.kubernetes.io/not-ready</code> and <code>node.kubernetes.io/unreachable</code> with <code>NoExecute</code> effect.
These tolerations have a <code>tolerationSeconds</code> field set to 300 seconds.
This means that if a node is not ready or unreachable for 5 minutes, the pods on that node will be evicted.
- You can override this behavior by setting a different value for <code>tolerationSeconds</code> in the pod's tolerations.
For example, you might want to keep an application with a lot of local state bound to node for a long time in the event of network partition, hoping that the partition will recover and thus the pod eviction can be avoided. The toleration you set for that Pod might look like:
```yaml
tolerations:
- key: "node.kubernetes.io/unreachable"
  operator: "Exists"
  effect: "NoExecute"
  tolerationSeconds: 6000
```


### DaemonSet

DaemonSet pods are created with <code>NoExecute</code> tolerations for the following taints with no <code>tolerationSeconds</code>:

- <code>node.kubernetes.io/unreachable</code>
- <code>node.kubernetes.io/not-ready</code>

This ensures that DaemonSet pods are never evicted due to these problems.