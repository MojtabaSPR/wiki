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
### Affinity and Anti-Affinity

1. The scheduler selects a node based on the `Affinity` and `Anti-Affinity` fields in the pod spec.
2. Affinity and anti-affinity expands the types of constraints you can define.
3. The affinity/anti-affinity language is more expressive. nodeSelector only selects nodes with all the specified labels. Affinity/anti-affinity gives you more control over the selection logic.
4. You can indicate that a rule is soft or preferred, so that the scheduler still schedules the Pod even if it can't find a matching node.
5. You can constrain a Pod using labels on other Pods running on the node (or other topological domain), instead of just node labels, which allows you to define rules for which Pods can be co-located on a node.

#### Affinity Types

1. **Node affinity** functions like the nodeSelector field but is more expressive and allows you to specify soft rules.
2. **Inter-pod affinity/anti-affinity** allows you to constrain Pods against labels on other Pods

#### Node affinity

1. **requiredDuringSchedulingIgnoredDuringExecution**: The scheduler can't schedule the Pod unless the rule is met. This functions like nodeSelector, but with a more expressive syntax.
2. **preferredDuringSchedulingIgnoredDuringExecution**: The scheduler tries to find a node that meets the rule. If a matching node is not available, the scheduler still schedules the Pod.

```bash
kubectl label nodes node1 disktype=ssd
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: disktype
            operator: In
            values:
            - ssd            
  containers:
  - name: nginx
    image: nginx
    imagePullPolicy: IfNotPresent
```

---


> **OR** <br> 
> If you specify multiple terms in nodeSelectorTerms associated with nodeAffinity types,
> then the Pod can be scheduled onto a node if one of the specified terms can be satisfied (terms are ORed).

```bash
kubectl label nodes node1 disktype=ssd
kubectl label nodes node2 zone=asia
```

```yaml
# the pod can be scheduled onto a node with either disktype=ssd or zone=asia
apiVersion: v1
kind: Pod
metadata:
  name: with-multiple-ORed-terms
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: disktype
            operator: In
            values:
            - ssd
        - matchExpressions:
          - key: zone
            operator: In
            values:
              - asia    
  containers:
  - name: nginx
    image: nginx
    imagePullPolicy: IfNotPresent
```
---

> **AND** <br> 
> If you specify multiple expressions in a single matchExpressions field associated with a term in nodeSelectorTerms,
> then the Pod can be scheduled onto a node only if all the expressions are satisfied (expressions are ANDed).

```bash
kubectl label nodes node1 disktype=ssd
kubectl label nodes node1 zone=asia
```

```yaml
# the pod can be scheduled onto a node only if both disktype=ssd and zone=asia are satisfied

kind: Pod
apiVersion: v1
metadata:
    name: with-multiple-ANDed-expressions
spec:
    affinity: 
        nodeAffinity:
            requiredDuringSchedulingIgnoredDuringExecution:
                nodeSelectorTerms:
                    - matchExpressions:
                        - key: disktype
                          operator: In
                          values:
                            - ssd
                        - key: zone
                          operator: In
                          values:
                            - asia
    containers:
        - name: nginx
          image: nginx:latest
          imagePullPolicy: IfNotPresent
```

---

> **Node affinity weight**
> 1. You can specify a weight between 1 and 100 for each instance of the preferredDuringSchedulingIgnoredDuringExecution affinity type.
> 2. The scheduler finds nodes that meet all the other scheduling requirements of the Pod
> 3. Then iterates through every preferred rule that the node satisfies and adds the value of the weight for that expression to a sum
> 4. The final sum is added to the score of other priority functions for the node
> 5. The node with the highest score is selected for the Pod

```bash
kubectl label nodes node1 kubernetes.io/os=linux
kubectl label nodes node2 kubernetes.io/os=linux
kubectl label nodes node3 kubernetes.io/os=linux

kubectl label nodes node1 label-1=key-1
kubectl label nodes node2 label-2=key-2
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: with-affinity-preferred-weight
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: kubernetes.io/os
            operator: In
            values:
            - linux
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 1
        preference:
          matchExpressions:
          - key: label-1
            operator: In
            values:
            - key-1
      - weight: 50
        preference:
          matchExpressions:
          - key: label-2
            operator: In
            values:
            - key-2
  containers:
  - name: with-node-affinity
    image: registry.k8s.io/pause:2.0

```


>1. In above example, three nodes match the <code> <i>requiredDuringSchedulingIgnoredDuringExecution</i> </code> rule: **node1**, **node2**, **node3**.
>2. **node1** and **node2** match the <code> <i>preferredDuringSchedulingIgnoredDuringExecution</i> </code> rule,
one with the label-1:key-1 label and another with the label-2:key-2 label, so they are selected for next step.
>3. Then the scheduler considers the weight of each node(node1 and node2) and adds the weight to the other scores for that node,
and schedules the Pod onto the node with the highest final score.
