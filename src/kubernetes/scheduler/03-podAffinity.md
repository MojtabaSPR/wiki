# Assign Pods to node

## Inter-Pod Affinity and Anti-Affinity

### What are Inter-Pod Affinity and Anti-Affinity?

- Inter-Pod Affinity:

    - Specifies that a pod should or must be scheduled on the same node (or nearby nodes) as another set of pods.
    - Useful for workloads that benefit from proximity, such as caching services or tightly coupled applications.

- Inter-Pod Anti-Affinity:

    - Specifies that a pod should or must be scheduled on a different node (or set of nodes) from another set of pods.
    - Useful for spreading workloads to improve availability or avoid resource contention.
---

### How It Works

1. Defined in Pod Spec:

    - Affinity and anti-affinity rules are added under the <code>affinity</code> field in the pod specification.

2. Key Components:

    - <code>podAffinity</code>: Defines affinity rules.
    - <code>podAntiAffinity</code>: Defines anti-affinity rules.
    - <code>requiredDuringSchedulingIgnoredDuringExecution</code>:
      - Hard constraint: Pod won't be scheduled unless the rule is satisfied.
    - <code>preferredDuringSchedulingIgnoredDuringExecution</code>:
      - Soft constraint: Scheduler tries to satisfy the rule but won't fail if it can't.

---

Examples
1. Inter-Pod Affinity Example
   Ensure that pods with the label <code>app=frontend</code> are scheduled on the same node or in the same zone as pods with the label <code>app=backend</code>.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: frontend
  labels:
    app: frontend
spec:
  affinity:
    podAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        - labelSelector:
            matchExpressions:
              - key: app
                operator: In
                values:
                  - backend
          topologyKey: "topology.kubernetes.io/zone"
  containers:
    - name: nginx
      image: nginx

```

Explanation:

- <code>labelSelector</code>: Matches pods with <code>app=backend</code>.
- <code>topologyKey</code>: Ensures the pod is scheduled in the same <code>zone</code> as the backend pods.


2. Inter-Pod Anti-Affinity Example
   Prevent pods with the label <code>app=frontend</code> from being scheduled on the same node as other <code>app=frontend</code> pods.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: frontend
  labels:
    app: frontend
spec:
  affinity:
    podAntiAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        - labelSelector:
            matchExpressions:
              - key: app
                operator: In
                values:
                  - frontend
          topologyKey: "kubernetes.io/hostname"
  containers:
    - name: nginx
      image: nginx

```


Explanation:

- <code>labelSelector</code>: Matches pods with <code>app=frontend</code>.
- <code>topologyKey</code>: Ensures pods with the same label are not scheduled on the same node (<code>hostname</code>).

