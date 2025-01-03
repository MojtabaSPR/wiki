# POD


**Pods in a Kubernetes cluster are used in two main ways:**

- Pods that run a single container. 
- Pods that run multiple containers that need to work together. 

---

### Pods augment containers

On the augmentation front, Pods augment containers in all the following ways.
- Labels and annotations
- Restart policies
- Probes (startup probes, readiness probes, liveness probes, and potentially more)
- Affinity and anti-affinity rules
- Termination control
- Security policies
- Resource requests and limits

---

### The anatomy of a Pod

At the highest level, a Pod is an execution environment shared by one or more containers. Shared execution
environment means the Pod has a set of resources that are shared by every container it runs. These resources
include IP address, ports, hostname, sockets, memory, volumes, and more…

Pod is a collection of resources that containers running inside of it inherit and share. These
resources are actually Linux kernel namespaces, and include the following:
- **net namespace**: IP address, port range, routing table…
- **pid namespace**: isolated process tree
- **mnt namespace**: filesystems and volumes…
- **UTS namespace**: Hostname
- **IPC namespace**: Unix domain sockets and shared memory

Each container within a Pod runs in its own cgroup, but they share a
number of Linux namespaces.
Applications running in the same Pod share the same IP address and port
space (network namespace), have the same hostname (UTS namespace),
and can communicate using native interprocess communication channels
over System V IPC or POSIX message queues (IPC namespace).

#### The pod network

Each Pod creates its own network namespace. This means a Pod has its own IP address, a single range of TCP and
UDP ports, and a single routing table. If it’s a single-container Pod, the container has full access to the IP, port
range and routing table. If it’s a multi-container Pod, because containers in a pod run in the same Network
namespace, all containers share the IP, port range and routing table
and can find each other via `localhost`.

#### Pod FileSystem and  volume

When it comes to the filesystem, things are a little different. Because most of the
container’s filesystem comes from the container image, by default, the filesystem of
each container is fully isolated from other containers. However, it’s possible to have
them share file directories using a Kubernetes concept called a `Volume`.


>Note:
>
> Restarting a container in a Pod should not be confused with restarting a Pod.
> A Pod is not a process, but an environment for running container(s).
> A Pod persists until it is deleted.

---

### Controllers for Pod

A controller for the resource handles replication and rollout and automatic healing in case of Pod failure. For example, if a Node fails, a controller notices that Pods on that Node have stopped working and creates a replacement Pod. The scheduler places the replacement Pod onto a healthy Node.
Pods are rarely managed directly. Instead, higher-level controllers like Deployments, StatefulSets, or DaemonSets manage them:

- `Deployments`: Manage stateless applications.
- `StatefulSets`: Manage stateful applications.
- `DaemonSets`: Ensure a Pod runs on every (or some) node(s).


Controllers for workload resources(jobs, deployments, daemonSets ...) create Pods from a pod template and manage those Pods on your behalf.

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: hello
spec:
  template:
    # This is the pod template
    spec:
      containers:
      - name: hello
        image: busybox:1.28
        command: ['sh', '-c', 'echo "Hello, Kubernetes!" && sleep 3600']
      restartPolicy: OnFailure
    # The pod template ends here
```

- If you change the pod template for a workload resource, that resource needs to create replacement Pods that use the updated template.

- Each workload resource(such as jobs, deployments, daemonSets ...) implements its own rules for handling changes to the Pod template.

- When the Pod template for a workload resource is changed, the controller creates new Pods based on the updated template instead of updating or patching the existing Pods.

- Kubernetes doesn't prevent you from managing Pods directly. It is possible to update some fields of a running Pod, in place. However, Pod update operations like patch, and replace have some limitations:

    - Most of the metadata about a Pod is immutable. For example, you cannot change the `namespace`, `name`, `uid`, or `creationTimestamp` fields; the generation field is unique. It only accepts updates that increment the field's current value.

    - If the `metadata.deletionTimestamp` is set, no new entry can be added to the `metadata.finalizers` list.

    - Pod updates may not change fields other than `spec.containers[*].image`, `spec.initContainers[*].image`, `spec.activeDeadlineSeconds` or `spec.tolerations`. `For spec.tolerations`, you can only add new entries.

    - When updating the `spec.activeDeadlineSeconds` field, two types of updates are allowed:

      - setting the unassigned field to a positive number;
      - updating the field from a positive number to a smaller, non-negative number

    