# Assign Pods to node

### NodeName

1. The scheduler selects a node based on the `NodeName` field in the pod spec.
2. If the `NodeName` field is set, the scheduler will try to schedule the pod on the node with the specified name.
3. If the node with the specified name does not exist, the pod will remain in the `Pending` state until a node with the specified name is created.
4. If the `NodeName` field is not set, the scheduler will use the default scheduling algorithm to select a node for the pod.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    app: nginx
    tier: frontend
spec:
  containers:
  - name: nginx
    image: nginx:1.14.2
  nodeName: node1
```

### NodeSelector

1. The scheduler selects a node based on the `NodeSelector` field in the pod spec.
2. If the `NodeSelector` field is set, the scheduler will try to schedule the pod on the node that matches the specified labels.
3. If no node matches the labels, the pod will remain in the `Pending` state until a node with the specified labels is created.
4. If the `NodeSelector` field is not set, the scheduler will use the default scheduling algorithm to select a node for the pod.

```bash
kubectl label nodes node1 disktype=ssd
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    app: nginx
    tier: frontend
spec:
  containers:
  - name: nginx
    image: nginx:1.14.2
  nodeSelector:
    disktype: ssd
```