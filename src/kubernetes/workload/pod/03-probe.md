# Container probes 

Container probes are used to check the health of a container.


### Types of probe
The kubelet can optionally perform and react to three kinds of probes on running containers:
- `Liveness Probe`: A liveness probe is used to check if a container is still running. If the liveness probe fails, the kubelet kills the container, and the container is subjected to its restart policy. If a container does not provide a liveness probe, the default state is Success.
- `Readiness Probe`: A readiness probe is used to check if a container is ready to accept traffic. If the readiness probe fails, the endpoints controller removes the endpoint from the service. If a container does not provide a readiness probe, the default state is Success.
- `Startup Probe`: A startup probe is used to check if a container is ready to start accepting traffic. If the startup probe fails, the kubelet kills the container, and the container is subjected to its restart policy. If a container does not provide a startup probe, the default state is Success. All other probes are disabled if a startup probe is provided, until it succeeds.


### Check mechanisms

There are four different ways to check a container using a probe. Each probe must define exactly one of these four mechanisms:

- `exec`:
Executes a specified command inside the container. The diagnostic is considered successful if the command exits with a status code of 0.
- `grpc`:
Performs a remote procedure call using gRPC. The target should implement gRPC health checks. The diagnostic is considered successful if the status of the response is SERVING.
- `httpGet`:
Performs an HTTP GET request against the Pod's IP address on a specified port and path. The diagnostic is considered successful if the response has a status code greater than or equal to 200 and less than 400.
- `tcpSocket`:
Performs a TCP check against the Pod's IP address on a specified port. The diagnostic is considered successful if the port is open. If the remote system (the container) closes the connection immediately after it opens, this counts as healthy.


### Probe outcomes

Each probe has one of three results:

- `Success`
The container passed the diagnostic.

- `Failure`
The container failed the diagnostic.

- `Unknown`
The diagnostic failed (no action should be taken, and the kubelet will make further checks)

---

### When should you use a liveness probe?
If the process in your container is able to crash on its own whenever it encounters an issue or becomes unhealthy, you do not necessarily need a liveness probe; the kubelet will automatically perform the correct action in accordance with the Pod's restartPolicy.

If you'd like your container to be killed and restarted if a probe fails, then specify a liveness probe, and specify a restartPolicy of Always or OnFailure.

### When should you use a readiness probe?

If your app has a strict dependency on back-end services, you can implement both a liveness and a readiness probe. The liveness probe passes when the app itself is healthy, but the readiness probe additionally checks that each required back-end service is available.


---

In this example, we will create a pod with a liveness probe and a readiness probe. The liveness probe will check if the container is still running, and the readiness probe will check if the container is ready to accept traffic. If either of these probes fail, the pod will be restarted.



## Create a pod with probes

To create a pod with probes, we can use the following YAML file:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  containers:
  - name: etcd
    image: registry.k8s.io/etcd:3.5.1-0
    command: [ "/usr/local/bin/etcd", "--data-dir",  "/var/lib/etcd", "--listen-client-urls", "http://0.0.0.0:2379", "--advertise-client-urls", "http://127.0.0.1:2379", "--log-level", "debug"]
    ports:
      - name: data-port
        containerPort: 2379
    livenessProbe:
      exec:
        command: ['cat', '/tmp/healthy']
      initialDelaySeconds: 5
      periodSeconds: 5
    readinessProbe:
      exec:
        command: ['cat', '/tmp/ready']
      initialDelaySeconds: 5
      periodSeconds: 5
    startupProbe:
      httpGet:
        path: /healthz
        port: data-port
      failureThreshold: 30
      periodSeconds: 10
  restartPolicy: Always
```

In this pod, we have a container that runs the `echo` command and then sleeps for 30 seconds. We have also defined two probes:

- A liveness probe that checks if the file `/tmp/healthy` exists. If it does, the probe is successful.
- A readiness probe that checks if the file `/tmp/ready` exists. If it does, the probe is successful.
- A startup probe that checks if the container is ready to start accepting traffic. In this case, the probe performs an HTTP GET request against the `/healthz` path on port `data-port`. The probe will fail if the response status code is not 200. The `failureThreshold` field defines how many times the probe can fail before the container is considered unhealthy. In this case, the probe will fail if it fails 30 times in a row. The `periodSeconds` field defines how often the probe should be executed

The `initialDelaySeconds` and `periodSeconds` fields define how often the probes should be executed. In this case, we have set the initial delay to 5 seconds and the period to 5 seconds.

## Create the pod

To create the pod, run the following command:

```
kubectl apply -f pod-with-probes.yaml
```