# Pod Lifecycle

Pods are only scheduled once in their lifetime; assigning a Pod to a specific node is called binding, and the process of selecting which node to use is called scheduling. 
Once a Pod has been scheduled and is bound to a node, Kubernetes tries to run that Pod on the node. 
The Pod runs on that node until it stops, or until the Pod is terminated; if Kubernetes isn't able to start the Pod on the selected node (for example, if the node crashes before the Pod starts), then that particular Pod never starts.


A given Pod (as defined by a UID) is never "rescheduled" to a different node; instead, that Pod can be replaced by a new, near-identical Pod.
If you make a replacement Pod, it can even have same name (as in `.metadata.name`) that the old Pod had, but the replacement would have a different `.metadata.uid` from the old Pod.


## Pod Status Object

Pod has an object named `status`. status has multiple fields like:

conditions , phase, QosClass, podIP, hostIP, containerStatuses, reason ...

each of these fields indicate a state of the Pod.

```shell
~ kubectl get pods pod2 -o jsonpath={".status"} | json_pp
```

```yaml
{
   "conditions" : [
      {
         "lastProbeTime" : null,
         "lastTransitionTime" : "2024-12-03T15:12:44Z",
         "status" : "True",
         "type" : "PodReadyToStartContainers"
      },
      {
         "lastProbeTime" : null,
         "lastTransitionTime" : "2024-12-03T15:12:13Z",
         "status" : "True",
         "type" : "Initialized"
      },
      {
         "lastProbeTime" : null,
         "lastTransitionTime" : "2024-12-03T15:13:16Z",
         "status" : "True",
         "type" : "Ready"
      },
      {
         "lastProbeTime" : null,
         "lastTransitionTime" : "2024-12-03T15:13:16Z",
         "status" : "True",
         "type" : "ContainersReady"
      },
      {
         "lastProbeTime" : null,
         "lastTransitionTime" : "2024-12-03T15:12:13Z",
         "status" : "True",
         "type" : "PodScheduled"
      }
   ],
   "containerStatuses" : [
      {
         "containerID" : "cri-o://aee8bdb9f40c5380d61d405f480b4827c10fa8635a3490107f9f28b3576c8199",
         "image" : "docker.io/library/nginx:latest",
         "imageID" : "docker.io/library/nginx@sha256:3d696e8357051647b844d8c7cf4a0aa71e84379999a4f6af9b8ca1f7919ade42",
         "lastState" : {},
         "name" : "nginx",
         "ready" : true,
         "restartCount" : 0,
         "started" : true,
         "state" : {
            "running" : {
               "startedAt" : "2024-12-03T15:13:16Z"
            }
         },
         "volumeMounts" : [
            {
               "mountPath" : "/var/run/secrets/kubernetes.io/serviceaccount",
               "name" : "kube-api-access-jnfj7",
               "readOnly" : true,
               "recursiveReadOnly" : "Disabled"
            }
         ]
      }
   ],
   "hostIP" : "192.168.60.232",
   "hostIPs" : [
      {
         "ip" : "192.168.60.232"
      }
   ],
   "phase" : "Running",
   "podIP" : "10.75.1.2",
   "podIPs" : [
      {
         "ip" : "10.75.1.2"
      }
   ],
   "qosClass" : "BestEffort",
   "startTime" : "2024-12-03T15:12:13Z"
}
```

### Pod Phases

Pod phase is a field in the `status` object that indicates the current state of the Pod.

A Pod's lifecycle has several phases that describe its current state. These phases are high-level descriptions and not detailed enough for fine-grained control.


| Value        | 	Description                                                                                                                                                                                                                                                       |
|--------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `Pending`	   | The Pod has been accepted by the Kubernetes cluster, but one or more of the containers has not been set up and made ready to run. This includes time a Pod spends waiting to be scheduled as well as the time spent downloading container images over the network. |
| `Running`	   | The Pod has been bound to a node, and all of the containers have been created. At least one container is still running, or is in the process of starting or restarting.                                                                                            |                                                                                                                                                                                                                                              |
| `Succeeded`	 | All containers in the Pod have terminated in success, and will not be restarted.                                                                                                                                                                                   |
| `Failed`	    | All containers in the Pod have terminated, and at least one container has terminated in failure. That is, the container either exited with non-zero status or was terminated by the system, and is not set for automatic restarting.                               |
| `Unknown`	   | For some reason the state of the Pod could not be obtained. This phase typically occurs due to an error in communicating with the node where the Pod should be running.                                                                                            |


- If a node dies or is disconnected from the rest of the cluster, Kubernetes applies a policy for setting the phase of all Pods on the lost node to Failed.


### Container states

Another important part of a Pod's lifecycle is the state of each container inside the Pod.
As well as the phase of the Pod overall, Kubernetes tracks the state of each container inside a Pod. 
You can use container lifecycle hooks to trigger events to run at certain points in a container's lifecycle.

Pod status object has a field named `containerStatuses` that is a list of one entry per container in the manifest and include information about each container's state.

Once the scheduler assigns a Pod to a Node, the kubelet starts creating containers for that Pod using a container runtime. There are three possible container states: Waiting, Running, and Terminated.


| Value      | Description                                                                                                                                                                                                                                                                                                                                                                                                                                       |
|------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Waiting    | If a container is not in either the Running or Terminated state, it is Waiting. A container in the Waiting state is still running the operations it requires in order to complete start up: for example, pulling the container image from a container image registry, or applying Secret data. When you use kubectl to query a Pod with a container that is Waiting, you also see a Reason field to summarize why the container is in that state. |
| Running    | The Running status indicates that a container is executing without issues. If there was a postStart hook configured, it has already executed and finished. When you use kubectl to query a Pod with a container that is Running, you also see information about when the container entered the Running state.                                                                                                                                     |
| Terminated | A container in the Terminated state began execution and then either ran to completion or failed for some reason. When you use kubectl to query a Pod with a container that is Terminated, you see a reason, an exit code, and the start and finish time for that container's period of execution.                                                                                                                                                 |


### Restart Policy
The Pod's restartPolicy defines what happens when a container fails. It can be set to one of the following:

- `Always (default)`:
The container is restarted regardless of its exit status.

- `OnFailure`:
The container is restarted only if it exits with a non-zero exit code.

- `Never`:
The container is never restarted.

The restartPolicy for a Pod applies to app containers in the Pod and to regular init containers.

After containers in a Pod exit, the kubelet restarts them with an exponential `backoff delay` (10s, 20s, 40s, …), that is capped at 300 seconds (5 minutes).

Once a container has executed for 10 minutes without any problems, the kubelet resets the restart backoff timer for that container.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: restart-policy-demo
spec:
  restartPolicy: OnFailure
  containers:
    - name: app
      image: busybox
      command: ["false"]

```

**How Pods handle problems with containers:**

Kubernetes reacts to containers exiting due to errors or other reasons in the following sequence:

1. **Initial crash:** Kubernetes attempts an immediate restart based on the Pod restartPolicy.
2. **Repeated crashes:** After the initial crash Kubernetes applies an exponential `backoff delay` for subsequent restarts. This prevents rapid, repeated restart attempts from overloading the system.
3. **CrashLoopBackOff state:** This indicates that the backoff delay mechanism is currently in effect for a given container that is in a crash loop, failing and restarting repeatedly.
4. **Backoff reset:** If a container runs successfully for a certain duration (e.g., 10 minutes), Kubernetes resets the backoff delay, treating any new crash as the first one.




### Pod Conditions

Condition is an array field in the `status` object through which the Pod has or has not passed.

each condition has below fields:

- `lastProbeTime`: The last time the condition transitioned from one status to another.
- `lastTransitionTime`: The last time the condition transitioned from one status to another.
- `message`: A human-readable message indicating details about the transition.
- `reason`: Machine-readable, UpperCamelCase text indicating the reason for the condition's last transition..
- `status`: The status of the condition, one of True, False, Unknown.
- `type`: The type of condition (e.g., PodScheduled, Ready).


The `type` field can be one of the following:

- `PodScheduled`: The pod has been scheduled to a node, but the scheduler has not yet confirmed that the pod is running.
- `Ready`: The pod is able to serve requests and should be added to the load balancing pools of all matching services.
- `Initialized`: All init containers have started successfully.
- `ContainersReady`: All containers in the pod are ready.
- `PodReadyToStartContainers`: The pod is able to start all containers, but is not ready to serve requests.


## Examples

Example1: Pod is in `pending` phase and `PodScheduled` condition type is `False` due to unresolved scheduling gates
```shell
~ kubectl get pods example-pod-scheduling-gate -o jsonpath={".status"} | json_pp
```

```json
{
  "conditions" : [
    {
      "lastProbeTime" : null,
      "lastTransitionTime" : "2024-12-04T11:16:41Z",
      "message" : "Scheduling is blocked due to non-empty scheduling gates",
      "reason" : "SchedulingGated",
      "status" : "False",
      "type" : "PodScheduled"
    }
  ],
  "phase" : "Pending",
  "qosClass" : "BestEffort"
}
```

---

Example2: Pod is pending due to ImagePullBackOff error:
- PodScheduled condition type is `True`
- Initialized condition type is `True`
- PodReadyToStartContainers condition type is `True`
- ContainersReady condition type is `False` (container is not ready due to ImagePullBackOff error)
- Ready condition type is `False` (pod is not ready due to container not ready)

```shell
~ kubectl get pods example-pod-pending -o jsonpath={".status"} | json_pp
```

```json
{
   "conditions" : [
      {
         "lastProbeTime" : null,
         "lastTransitionTime" : "2024-12-11T09:33:22Z",
         "status" : "True",
         "type" : "PodReadyToStartContainers"
      },
      {
         "lastProbeTime" : null,
         "lastTransitionTime" : "2024-12-11T09:33:15Z",
         "status" : "True",
         "type" : "Initialized"
      },
      {
         "lastProbeTime" : null,
         "lastTransitionTime" : "2024-12-11T09:33:15Z",
         "message" : "containers with unready status: [nginx]",
         "reason" : "ContainersNotReady",
         "status" : "False",
         "type" : "Ready"
      },
      {
         "lastProbeTime" : null,
         "lastTransitionTime" : "2024-12-11T09:33:15Z",
         "message" : "containers with unready status: [nginx]",
         "reason" : "ContainersNotReady",
         "status" : "False",
         "type" : "ContainersReady"
      },
      {
         "lastProbeTime" : null,
         "lastTransitionTime" : "2024-12-11T09:33:15Z",
         "status" : "True",
         "type" : "PodScheduled"
      }
   ],
   "containerStatuses" : [
      {
         "image" : "nginx-not-valid",
         "imageID" : "",
         "lastState" : {},
         "name" : "nginx",
         "ready" : false,
         "restartCount" : 0,
         "started" : false,
         "state" : {
            "waiting" : {
               "message" : "initializing source docker://quay.io/nginx-not-valid:latest: reading manifest latest in quay.io/nginx-not-valid: StatusCode: 404, \"<!doctype html>\\n<html lang=en>\\n<title>404 Not Foun...\"",
               "reason" : "ErrImagePull"
            }
         },
         "volumeMounts" : [
            {
               "mountPath" : "/var/run/secrets/kubernetes.io/serviceaccount",
               "name" : "kube-api-access-ksdtf",
               "readOnly" : true,
               "recursiveReadOnly" : "Disabled"
            }
         ]
      }
   ],
   "hostIP" : "192.168.60.232",
   "hostIPs" : [
      {
         "ip" : "192.168.60.232"
      }
   ],
   "phase" : "Pending",
   "podIP" : "10.75.1.5",
   "podIPs" : [
      {
         "ip" : "10.75.1.5"
      }
   ],
   "qosClass" : "BestEffort",
   "startTime" : "2024-12-11T09:33:15Z"
}
```

---

Example3: Running Pod

```shell
~ kubectl get pods pod2 -o jsonpath={".status"} | json_pp
```

```json
{
   "conditions" : [
      {
         "lastProbeTime" : null,
         "lastTransitionTime" : "2024-12-03T15:12:44Z",
         "status" : "True",
         "type" : "PodReadyToStartContainers"
      },
      {
         "lastProbeTime" : null,
         "lastTransitionTime" : "2024-12-03T15:12:13Z",
         "status" : "True",
         "type" : "Initialized"
      },
      {
         "lastProbeTime" : null,
         "lastTransitionTime" : "2024-12-03T15:13:16Z",
         "status" : "True",
         "type" : "Ready"
      },
      {
         "lastProbeTime" : null,
         "lastTransitionTime" : "2024-12-03T15:13:16Z",
         "status" : "True",
         "type" : "ContainersReady"
      },
      {
         "lastProbeTime" : null,
         "lastTransitionTime" : "2024-12-03T15:12:13Z",
         "status" : "True",
         "type" : "PodScheduled"
      }
   ],
   "containerStatuses" : [
      {
         "containerID" : "cri-o://aee8bdb9f40c5380d61d405f480b4827c10fa8635a3490107f9f28b3576c8199",
         "image" : "docker.io/library/nginx:latest",
         "imageID" : "docker.io/library/nginx@sha256:3d696e8357051647b844d8c7cf4a0aa71e84379999a4f6af9b8ca1f7919ade42",
         "lastState" : {},
         "name" : "nginx",
         "ready" : true,
         "restartCount" : 0,
         "started" : true,
         "state" : {
            "running" : {
               "startedAt" : "2024-12-03T15:13:16Z"
            }
         },
         "volumeMounts" : [
            {
               "mountPath" : "/var/run/secrets/kubernetes.io/serviceaccount",
               "name" : "kube-api-access-jnfj7",
               "readOnly" : true,
               "recursiveReadOnly" : "Disabled"
            }
         ]
      }
   ],
   "hostIP" : "192.168.60.232",
   "hostIPs" : [
      {
         "ip" : "192.168.60.232"
      }
   ],
   "phase" : "Running",
   "podIP" : "10.75.1.2",
   "podIPs" : [
      {
         "ip" : "10.75.1.2"
      }
   ],
   "qosClass" : "BestEffort",
   "startTime" : "2024-12-03T15:12:13Z"
}
```
