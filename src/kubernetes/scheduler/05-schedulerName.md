# Specify schedulers for pods

>If the default scheduler does not suit your needs you can implement your own scheduler. Moreover, you can even run multiple schedulers simultaneously alongside the default scheduler and instruct Kubernetes what scheduler to use for each of your pods.
>Based on Kubernetes docs [here](https://kubernetes.io/docs/tasks/extend-kubernetes/configure-multiple-schedulers/).
 

After creating new scheduler, assign it to pod as below:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: annotation-second-scheduler
  labels:
    name: multischeduler-example
spec:
  schedulerName: my-scheduler
  containers:
  - name: pod-with-second-annotation-container
    image: registry.k8s.io/pause:2.0

```