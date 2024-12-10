# Pod Priority and Preemption

---

## Pod Priority

- Pods can have priority. Priority indicates the importance of a Pod relative to other Pods, which affects the order in which Pods are scheduled first.
- Each pod can have a priority value that influences the scheduler’s decision-making process.
- Pods with higher priority are scheduled before pods with lower priority when resources are scarce.

---

## PriorityClass

A PriorityClass is a <b>non-namespaced</b> object that defines a mapping from a priority class name to the integer value of the priority.

```yaml
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: high-priority
value: 1000
globalDefault: false
preemptionPolicy: PreemptLowerPriority
description: "This priority class is for high-priority workloads."

```
**Fields:**

- <code>name</code>: The name of the priority class.
- <code>value</code>: Any 32-bit integer value smaller than or equal to 1 billion, representing the priority (higher = more important).
- <code>globalDefault</code>: If true, this becomes the default priority for all pods without an explicitly assigned priority.
- <code>description</code>: An optional description.
- <code>preemptionPolicy</code>: The Policy for preempting pods with lower priority.
  - <code>PreemptLowerPriority</code>: means that pod can preempt other pods with lower priority (default behavior).
  - <code>Never</code>: means that pod never preempts other pods with lower priority.

Now we can assign this priority to Pods by its name as <code>priorityClassName</code>: high-priority. PriorityClass is a mechanism for indicating the importance of Pods relative
to each other, where the higher value indicates more important Pods.

```yaml
apiVersion: v1
kind: Pod
metadata:
 name: random-generator
 labels:
  env: random-generator
spec:
 containers:
  - image: k8spatterns/random-generator:1.0
    name: random-generator
 priorityClassName: high-priority
```

Pod has a field named <code>Priority</code>, this field is populated by **Admission Controller**, Users are not allowed to set it directly.
Admission controller resolves a <code>PriorityClassName</code> to its corresponding number and populates the <code>Priority</code> field of the pod spec. 
The rest of Kubernetes components look at the <code>Priority</code> field of pod status and work with the integer value. In other words, PriorityClassName will be ignored by the rest of the system.

---

### Preemption

When a new pod has certain scheduling requirements that makes it infeasible on any node in the cluster, scheduler may choose to kill lower priority pods to satisfy the scheduling requirements of the new pod. 
We call this operation "preemption". Preemption is distinguished from "eviction" where kubelet kills a pod on a node because that particular node is running out of resources.

- If a higher-priority pod cannot be scheduled due to resource constraints, the scheduler may preempt lower-priority pods.
- Preemption involves evicting lower-priority pods to free up resources.
- Preempted pods are terminated, and their resources are reclaimed for the higher-priority pod.

**Preemption order:**

- When scheduling a pending pod, scheduler tries to place the pod on a node that does not require preemption.
- If there is no such a node, scheduler may favor a node where the number and/or priority of victims (preempted pods) is smallest.
- After choosing the node, scheduler considers the lowest priority pods for preemption first.
- Scheduler starts from the lowest priority and considers enough pods that should be preempted to allow the pending pod to schedule.
- Scheduler only considers pods that have lower priority than the pending pod.

- When ordering the pods from lowest to highest priority for considering which pod(s) to preempt, among pods with equal priority the pods are ordered by their QoS class: Best Effort, Burstable, Guaranteed.
- Scheduler respects pods' disruption budget when considering them for preemption.
- Scheduler will try to minimize the number of preempted pods. As a result, it may preempt a pod while leaving lower priority pods running if preemption of those lower priority pods is not enough to schedule the pending pod while preemption of the higher priority pod(s) is enough to schedule the pending pod. For example, if node capacity is 10, and pending pod is priority 10 and requires 5 units of resource, and the running pods are {priority 0 request 3, priority 1 request 1, priority 2 request 5, priority 3 request 1}, scheduler will preempt the priority 2 pod only and leaves priority 1 and priority 0 running.
- Scheduler does not have the knowledge of resource usage of pods. It makes scheduling decisions based on the requested resources ("requests") of the pods and when it considers a pod for preemption, it assumes the "requests" to be freed on the node.
  - This means that scheduler will never preempt a Best Effort pod to make more resources available. That's because the requests of Best Effort pods is zero and therefore, preempting them will not release any resources on the node from the scheduler's point of view.
  - The scheduler may still preempt Best Effort pods for reasons other than releasing their resources. For example, it may preempt a Best Effort pod in order to satisfy affinity rules of the pending pod.
  - The amount that needs to be freed (when the issue is resources) is request of the pending pod.


---

### Non-preempting PriorityClass

<code>preemptionPolicy</code> defaults to <code>PreemptLowerPriority</code>, which will allow pods of that PriorityClass to preempt lower-priority pods (as is existing default behavior).
If <code>preemptionPolicy</code> is set to <code>Never</code>, pods in that PriorityClass will be non-preempting.

An example use case is for data science workloads. A user may submit a job that they want to be prioritized above other workloads, but do not wish to discard existing work by preempting running pods. The high priority job with preemptionPolicy: Never will be scheduled ahead of other queued pods, as soon as sufficient cluster resources "naturally" become free.

```yaml
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: high-priority-nonpreempting
value: 1000000
preemptionPolicy: Never
globalDefault: false
description: "This priority class will not cause other pods to be preempted."
```
---

## Default PriorityClasses

Kubernetes already ships with two PriorityClasses: 
- <code>system-cluster-critical</code>
- <code>system-node-critical</code>

These are common classes and are used to ensure that critical components are always scheduled first.

```shell
➜  ~ kubectl get priorityclasses.scheduling.k8s.io
NAME                      VALUE        GLOBAL-DEFAULT   AGE
system-cluster-critical   2000000000   false            6d7h
system-node-critical      2000001000   false            6d7h
```

- <code>system-node-critical</code> has the most priority in cluster 
- new priority classes priority value can be smaller than or equal to 1 billion (less than these default classes)

> Guaranteed Scheduling For Critical Add-On Pods:
> 
> Critical add-on pods are pods that Kubernetes considers essential for the cluster’s operation, such as:
> - DNS (e.g., CoreDNS)
> - Metrics collection (e.g., metrics-server)
> - Cluster monitoring tools
> 
> These pods must be scheduled even during resource shortages to maintain cluster functionality.
> These kind of pods (at least in kubeadm) have either <code>system-cluster-critical</code> or <code>system-node-critical</code> priorityClassName by default.

---








