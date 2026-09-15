# Pods
## Creating a Pod
- Creating pod via imperative `kubectl run` command: `kubectl run kuard --image=gcr.io/kuar-demo/kuard-amd64:blue`
- Observing its information
    - `kubectl get pods` or `kubectl get pods --watch`
    - `kubectl get pods kuard -o json` or `kubectl get pods kuard -o yaml`
    - `kubectl describe pods kuard`
    - `kubectl logs kuard` or `kubectl logs kuard -f`
- Port forwarding, creating HTTP request and observing logs
    - `kubectl port-forward kuard 8081:8080`
    - `curl http://localhost:8081`
    - `kubectl logs kuard`
- Deleting pod: `kubectl delete pods/kuard`


## Creating a Pod Manifest
***kuard-pod.yaml***
```
apiVersion: v1 
kind: Pod 
metadata:  #section for describing the Pod and its labels
  name: kuard 
spec:      #section for describing volumes
  containers:
  - image: gcr.io/kuar-demo/kuard-amd64:blue
    name: kuard 
    ports:
      - containerPort: 8080 
        name: http 
        protocol: TCP
```
- Launch a single instance of kuard: `kubectl apply -f kuard-pod.yaml`

## Pod Details
`kubectl describe pods kuard`

## Deleting a Pod
- Delete using name: `kubectl delete pods/kuard`
- Use the same file that you used to create it: `kubectl delete -f kuard-pod.yaml`
- Delete all pods: `kubectl delete pods --all`

## Accessing Your Pod
- Getting More Information with Logs: `kubectl logs kuard`
- Running Commands in Your Container with exec: `kubectl exec -it kuard -- ash`


## Health Checks
probe= a thorough investigation into a crime or other matter
### Liveness Probe
- Liveness determines if an application is running properly. 
- Containers that fail liveness checks are restarted

***kuard-pod-health.yaml***
```
apiVersion: v1 
kind: Pod 
metadata:
  name: kuard 
spec:
  containers:
    - image: gcr.io/kuar-demo/kuard-amd64:blue
      name: kuard 
      livenessProbe:
        httpGet:      # to perform an HTTP GET request against the /healthy endpoint on port 8080
          path: /healthy 
          port: 8080
        initialDelaySeconds: 5  #will not be called until 5 seconds after all the containers in the Pod are created
        timeoutSeconds: 1   #The probe must respond within the 1-second timeout
        periodSeconds: 10  #Kubernetes will call the probe every 10 seconds
        failureThreshold: 3 #If more than three consecutive probes fail, the container will fail and restart
      ports:
        - containerPort: 8080
          name: http 
          protocol: TCP
```
- Create pod: `kubectl apply -f kuard-pod-health.yaml`
- Forward port: `kubectl port-forward kuard 8080:8080`
### Readiness Probe
- Readiness describes when a container is ready to serve user requests. 
- Containers that fail readiness checks are removed from service load balancers.
- Readiness probes are configured similarly to liveness probes

Combining the readiness and liveness probes helps ensure only healthy containers are running within the cluster

### Startup Probe
- Alternative way of managing slow-starting containers. 
- When a Pod is started, the startup probe is run before any other probing of the Pod is started. The startup probe proceeds until it either times out (in which case the Pod is restarted) or it succeeds, at which time the liveness probe takes over.

### Advanced Probe Configuration
- Probes in Kubernetes have a number of advanced options, including 
    - how long to wait after Pod startup to start probing, 
    - how many failures should be considered a true failure, and 
    - how many successes are necessary to reset the failure count. 
- All of these configurations **receive default values when left unspecified**, but they may be necessary for more advanced use cases such as applications that are inherently flaky or take a long time to start up.


## Resource Management
### Resource Requests: Minimum Required Resources
When a Pod requests the resources required to run its containers, Kubernetes guarantees that these resources are available to the Pod

***kuard-pod-resreq.yaml***
```
apiVersion: v1 
kind: Pod 
metadata:
  name: kuard 
spec:
  containers:
    - image: gcr.io/kuar-demo/kuard-amd64:blue
      name: kuard 
      resources:
        requests:  #request for resources
          #The Pod will not be scheduled on a node that does not have at least these resources available
          cpu: "500m"   #500 milliCPU, or half a CPU core
          memory: "128Mi" #128 MiB of memory
      ports:
        - containerPort: 8080
          name: http 
          protocol: TCP
```
- Create pod: `kubectl apply -f kuard-pod-resreq.yaml`

- Resource Adjustments
    - CPU Adjustments
        - Suppose that we create a Pod with this container that requests 0.5 CPU. Kubernetes schedules this Pod onto a machine with a total of 2 CPU cores. As long as it is the only Pod on the machine, it will consume all 2.0 of the available cores, despite only requesting 0.5 CPU. 
        - If a second Pod with the same container and the same request of 0.5 CPU lands on the machine, then each Pod will receive 1.0 cores. 
        - If a third, identical Pod is scheduled, each Pod will receive 0.66 cores. 
        - Finally, if a fourth identical Pod is scheduled, each Pod will receive the 0.5 core it requested, and the node will be at capacity.
    - Memory Adjustments
        - Memory requests are handled similarly to CPU, but there is an important difference
        -  If a container is over its memory request, the OS can’t just remove memory from the process, because it’s been allocated. 
        - Consequently, when the system runs out of memory, the kubelet terminates containers whose memory usage is greater than their requested memory. These containers are automatically restarted, but with less available memory on the machine for the container to consume.


### Capping Resource Usage with Limits
In addition to setting the resources required by a Pod, which establishes the minimum resources available to it, you can also **set a maximum on a its resource usage** via resource limits.

***kuard-pod-reslim.yaml***
```
apiVersion: v1 
kind: Pod 
metadata:
  name: kuard 
spec:
  containers:
    - image: gcr.io/kuar-demo/kuard-amd64:blue
      name: kuard 
      resources:
        requests:  #request for resources
          #The Pod will not be scheduled on a node that does not have at least these resources available
          cpu: "500m"   #500 milliCPU, or half a CPU core
          memory: "128Mi" #128 MiB of memory
        limits:  #cap for resources
          #the kernel is configured to ensure that consumption cannot exceed these limits
          cpu: "1000m"   #1000 milliCPU, or 1 CPU core
          memory: "256Mi" #256 MiB of memory
      ports:
        - containerPort: 8080
          name: http 
          protocol: TCP
```

- When you establish limits on a container, the kernel is configured to ensure that consumption cannot exceed these limits. 
- A container with a CPU limit of 0.5 cores will only ever get 0.5 cores, even if the CPU is otherwise idle. A container with a memory limit of 256 MB will not be allowed additional memory; for example, **malloc** will fail if its memory usage exceeds 256 MB.

## Persisting Data with Volumes
### Using Volumes with Pods
***kuard-pod-vol.yaml***
```
apiVersion: v1 
kind: Pod 
metadata:  #section for describing the Pod and its labels
  name: kuard 
spec:                               # Specifications for the Pod's behavior
  volumes:                          # Defines storage volumes for the Pod
    - name: "kuard-data"            # Name of the volume
      hostPath:                     # Uses a directory from the worker node's filesystem  
        path: "/var/lib/kuard"       # Path on the node's disk
  containers:                       # List of containers in the Pod         
  - image: gcr.io/kuar-demo/kuard-amd64:blue  # Container image
    name: kuard                     # Name of the container  
    volumeMounts:                   # Mounts the volume into the container
      - mountPath: "/data"          # Path inside the container
        name: "kuard-data"          # References the volume defined above
    ports:                          # Network ports the container exposes
      - containerPort: 8080         # Port number
        name: http                  # Name for the port (useful for service discovery)
        protocol: TCP               # Network protocol
```
### Different Ways of Using Volumes with Pods 
- Communication/synchronization, Cache
  - emptyDir Volume. A temporary folder created when a Pod starts, deleted when the Pod dies
  - Pros: Simple, fast.
  - Cons: Data vanishes if Pod crashes.
- Persistent data
  - Network-attached storage (e.g., AWS EBS, Google Persistent Disk). 
  - Survives Pod crashes
  - Use Case: Databases (e.g., MySQL, PostgreSQL), User uploads
  - Costly
- Mounting the host filesystem
  - hostPath Volume. Shares a folder from the worker node with the Pod
  - Use cases: Accessing node-specific data (e.g., logs, device files)
  - Warning:
    - Tied to one node (breaks if Pod moves).
    - Security risk (Pod can access node files).

## Putting It All Together
Through a combination of persistent volumes, readiness and liveness probes, and resource restrictions, Kubernetes provides everything needed to run stateful applications reliably.