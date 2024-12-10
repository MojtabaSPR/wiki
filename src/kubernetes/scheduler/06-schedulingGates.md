# Pod Scheduling Readiness in Kubernetes

Kubernetes introduced **Pod Scheduling Readiness** to provide greater control over when a pod is considered ready for scheduling. This feature allows the use of **scheduling gates**, which delay scheduling until specific conditions are met.

## What is Pod Scheduling Readiness?

- **Pod Scheduling Readiness** enables external signals or conditions to control when a pod is ready to be scheduled.
- It introduces **scheduling gates**, which act as conditions that must be resolved before the Kubernetes scheduler attempts to place the pod on a node.

---

## Key Components

### 1. Scheduling Gates

- Defined in the pod's `spec.schedulingGates` field.
- Each scheduling gate includes a `name` that identifies the specific condition required for scheduling.

### 2. Scheduling Gate Conditions

- External controllers (e.g., custom Kubernetes operators) are responsible for resolving these conditions by removing the corresponding scheduling gates from the pod's specification.

---

## How It Works

1. **Pod Creation**:
    - Pods with scheduling gates are not considered for scheduling until all gates are resolved.

2. **Resolving Gates**:
    - An external controller or operator monitors the system and updates the pod's `schedulingGates` field to remove resolved gates.

3. **Pod Scheduling**:
    - Once all gates are resolved, the pod becomes eligible for scheduling based on Kubernetes' standard scheduling logic.

---

## Use Cases

1. **External Dependency Management**:
    - Delay pod scheduling until required external resources (e.g., databases, services) are ready.

2. **Cluster-Wide Coordination**:
    - Prevent pods from being scheduled during cluster maintenance or upgrades until conditions are met.

3. **Custom Logic for Scheduling**:
    - Implement business-specific scheduling logic using external controllers.

---

## Example

### Create a Pod with a Scheduling Gate

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: example-pod
spec:
  schedulingGates:
    - name: "example.com/gate-ready"
  containers:
    - name: nginx
      image: nginx
```

### Create an External Controller to Resolve the Scheduling Gate

```go
package main

import (
	"context"
	"flag"
	"fmt"
	"net/http"
	"time"

	corev1 "k8s.io/api/core/v1"
	metav1 "k8s.io/apimachinery/pkg/apis/meta/v1"
	"k8s.io/apimachinery/pkg/fields"
	"k8s.io/client-go/kubernetes"
	"k8s.io/client-go/tools/cache"
	"k8s.io/client-go/tools/clientcmd"
)

const (
	schedulingGateName  = "example.com/gate-ready"
	externalResourceURL = "https://localservice.local/health" // Example resource health-check endpoint
)

func main() {
	// Load Kubernetes configuration
	kubeconfig := flag.String("kubeconfig", "./kubeconfig", "absolute path to the kubeconfig file")
	flag.Parse()

	config, err := clientcmd.BuildConfigFromFlags("", *kubeconfig)
	if err != nil {
		panic(fmt.Sprintf("Failed to load kubeconfig: %v", err))
	}

	// Create Kubernetes client
	clientSet, err := kubernetes.NewForConfig(config)
	if err != nil {
		panic(fmt.Sprintf("Failed to create Kubernetes client: %v", err))
	}

	// Create a shared informer to watch pods
	informer := createPodInformer(clientSet)

	stopCh := make(chan struct{})
	defer close(stopCh)

	// Start the informer
	go informer.Run(stopCh)

	// Wait forever
	select {}
}

// createPodInformer sets up an informer to watch for pods with scheduling gates
func createPodInformer(clientSet *kubernetes.Clientset) cache.SharedIndexInformer {
	// Create a list-watch to only watch pods with the specific scheduling gate
	listWatcher := cache.NewListWatchFromClient(
		clientSet.CoreV1().RESTClient(),
		"pods",
		metav1.NamespaceAll,
		fields.Everything(),
	)

	// Create the informer
	informer := cache.NewSharedIndexInformer(
		listWatcher,
		&corev1.Pod{},
		10*time.Minute,
		cache.Indexers{},
	)

	// Add event handlers
	_, _ = informer.AddEventHandler(cache.ResourceEventHandlerFuncs{
		AddFunc:    func(obj interface{}) { handlePod(obj, clientSet) },
		UpdateFunc: func(oldObj, newObj interface{}) { handlePod(newObj, clientSet) },
	})

	return informer
}

// handlePod processes a pod and resolves the scheduling gate if the condition is met
func handlePod(obj interface{}, clientset *kubernetes.Clientset) {
	pod, ok := obj.(*corev1.Pod)
	if !ok {
		fmt.Println("Failed to cast object to pod")
		return
	}

	// Check if the pod has the specific scheduling gate
	hasGate := false
	for _, gate := range pod.Spec.SchedulingGates {
		if gate.Name == schedulingGateName {
			hasGate = true
			break
		}
	}

	if !hasGate {
		return
	}

	fmt.Printf("Found pod with scheduling gate: %s in namespace: %s\n", pod.Name, pod.Namespace)

	// Check if the external resource is ready
	if isExternalResourceReady() {
		resolveSchedulingGate(pod, clientset)
	} else {
		fmt.Printf("External resource is not ready for pod: %s\n", pod.Name)
	}
}

// isExternalResourceReady checks if the external resource (e.g., a database) is ready
func isExternalResourceReady() bool {
	// Example: Implement a health check for the external resource
	// Here, we're simulating a health check with a time delay
	time.Sleep(2 * time.Second) // Simulate a health check delay
	
	httpClient := &http.Client{
		Timeout: time.Second * 20,
	}
	req,err := http.NewRequest(http.MethodGet, externalResourceURL, nil)
	if err != nil {
		return false
	}
	resp, err := httpClient.Do(req)
	if err != nil {
		return false
	}
	defer resp.Body.Close()
	if resp.StatusCode == http.StatusOK {
		return true
	}
	return false // Simulate that the resource is ready
}

// resolveSchedulingGate resolves the scheduling gate for the pod
func resolveSchedulingGate(pod *corev1.Pod, clientSet *kubernetes.Clientset) {
	// Remove the scheduling gate
	pod.Spec.SchedulingGates = []corev1.PodSchedulingGate{}

	// Update the pod
	_, err := clientSet.CoreV1().Pods(pod.Namespace).Update(context.TODO(), pod, metav1.UpdateOptions{})
	if err != nil {
		fmt.Printf("Failed to update pod %s: %v\n", pod.Name, err)
		return
	}

	fmt.Printf("Resolved scheduling gate for pod: %s\n", pod.Name)
}

```

Run the external controller and create the pod:

```shell
$ go run main.go --kubeconfig=$HOME/.kube/config
Found pod with scheduling gate: example-pod
Resolved scheduling gate for pod: example-pod
```

> Above example uses a simple HTTP health check to simulate the external resource.
> The app continuly monitor k8s to find pods with the specific scheduling gate and if found any, checks the external resource to be healthy if true then it resolves the scheduling gate and let the pod to be scheduled.
> In real world scenario, the external resource can be any external service or resource that needs to be checked before scheduling the pod, And if external resource is not healthy then do some action to make the resource healthy or notify someone.
