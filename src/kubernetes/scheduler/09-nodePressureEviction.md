# Node-pressure Eviction


### What is Node Pressure Eviction?
Node pressure eviction is a mechanism Kubernetes uses to proactively terminate pods when a node is under resource pressure (e.g., low memory or disk space). This ensures the stability of the node and its ability to host critical workloads.

---

### Types of Node Pressures
1. Memory Pressure:
    - Evicts pods to prevent the node from running out of memory, which could lead to node crashes.
2. Disk Pressure:
    - Evicts pods to free up ephemeral storage (/var/lib/kubelet) or root filesystem space.
3. PID Pressure:
    - Evicts pods to prevent running out of process IDs (PIDs), ensuring that critical processes can still start.

---


### How Node Pressure Eviction Works
1. Node Monitors Resource Usage:

    - The kubelet monitors memory, disk, and PIDs on the node.

2. Thresholds Trigger Eviction:

    - When resource usage crosses predefined eviction thresholds, the kubelet starts evicting pods.

3. Eviction Order:

    - Pods are evicted based on their priority and quality of service (QoS) class:
        - Priority: Higher-priority pods are evicted last.
        - QoS: BestEffort pods are evicted before Burstable or Guaranteed pods.

---

### Eviction signals and thresholds


The kubelet uses various parameters to make eviction decisions, like the following:
- Eviction signals 
- Eviction thresholds
- Monitoring intervals


#### 1. Eviction Signals:

Eviction signals are the current state of a particular resource at a specific point in time. 
The kubelet uses eviction signals to make eviction decisions by comparing the signals to eviction thresholds, 
which are the minimum amount of the resource that should be available on the node.
The `kubelet` will support the ability to trigger eviction decisions on the following signals.

| Eviction Signal    | Description                                                                     |
|--------------------|---------------------------------------------------------------------------------|
| memory.available   | memory.available := node.status.capacity[memory] - node.stats.memory.workingSet |
| nodefs.available   | nodefs.available := node.stats.fs.available                                     |
| nodefs.inodesFree  | nodefs.inodesFree := node.stats.fs.inodesFree                                   |
| imagefs.available  | imagefs.available := node.stats.runtime.imagefs.available                       |
| imagefs.inodesFree | imagefs.inodesFree := node.stats.runtime.imagefs.inodesFree                     |



#### 2. Eviction thresholds:

You can specify custom eviction thresholds for the kubelet to use when it makes eviction decisions. 
You can configure soft and hard eviction thresholds.

It has two types:

- <code>Soft eviction thresholds</code>: A soft eviction threshold pairs an eviction threshold with a required administrator-specified grace period. The kubelet does not evict pods until the grace period is exceeded. The kubelet returns an error on startup if you do not specify a grace period.
- <code>Hard eviction thresholds</code>: A hard eviction threshold has no grace period. When a hard eviction threshold is met, the kubelet kills pods immediately without graceful termination to reclaim the starved resource.    

An eviction threshold is of the following form:

`<eviction-signal><operator><quantity | int%>`

**Example configuration:**

```yaml
apiVersion: kubelet.config.k8s.io/v1beta1
kind: KubeletConfiguration
evictionHard:
  memory.available: "100Mi"        # Evict pods if less than 100Mi memory is available
  nodefs.available: "10%"         # Evict pods if less than 10% disk space is available
  imagefs.available: "15%"        # Evict pods if less than 15% image filesystem space is available
  pid.available: "5%"             # Evict pods if less than 5% PIDs are available
evictionSoft:
  memory.available: "200Mi"
  memory.availableGracePeriod: "1m"
evictionSoftGracePeriod:   # A set of eviction grace periods (e.g. memory.available=1m30s) that correspond to how long a soft eviction threshold must hold before triggering a pod eviction.
  memory.available: "1m30s"
evictionMaxPodGracePeriod: 60 # Maximum allowed grace period (in seconds) to use when terminating pods in response to a soft eviction threshold being met.
```
More info about changing kubelet configuration https://kubernetes.io/docs/tasks/administer-cluster/kubelet-config-file/

#### 3. monitoring interval

The kubelet evaluates eviction thresholds based on its configured <code>housekeeping-interval</code>, which defaults to 10s.


## Node Conditions

The `kubelet` will support a node condition that corresponds to each eviction signal.

If a hard eviction threshold has been met, or a soft eviction threshold has been met
independent of its associated grace period, the `kubelet` will report a condition that
reflects the node is under pressure.

The following node conditions are defined that correspond to the specified eviction signal.

| Node Condition | Eviction Signal                                                               | Description                                                                                                                  |
|----------------|-------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------|
| MemoryPressure | memory.available                                                              | Available memory on the node has satisfied an eviction threshold                                                             |
| DiskPressure   | nodefs.available, nodefs.inodesFree, imagefs.available, or imagefs.inodesFree | Available disk space and inodes on either the node's root filesystem or image filesystem has satisfied an eviction threshold |

The `kubelet` will continue to report node status updates at the frequency specified by
`--node-status-update-frequency` which defaults to `10s`.

