# Common kubectl Commands

### Viewing Cubernetes API Objects
- List resources
    - `kubectl get <resource_name>` [resource_name= pods, services, nodes, depployments, namespaces etc]
    - `kubectl get <resource_name_1>,<resource_name_2>` [View multiple objects of different types]
       Ex: `kubectl get pods, services`
- Get specific object
    - `kubectl get services <service_name>`
    - `kubectl get services <service_name> -o json` [Get slight more details in JSON format]
    - `kubectl get services <service_name> -o yaml` [Get slight more details in YAML format]
- Get more detailed information about a particular object
`kubectl describe services <service_name>`
- To continually observe the state of a particular Kubernetes resource:  
`kubectl get services kubernetes --watch` [Observing a service named kubernetes]
- See a list of supported fields for each supported type of Kubernetes object
`kubectl explain services`

### Creating, Updating, and Destroying Kubernetes Objects
Let’s assume that you have a simple object stored in obj.yaml.
- To create this object in Kubernetes: `kubectl apply -f obj.yaml`
- Update the object (after making changes to the object): `kubectl apply -f myobj.yaml view-last-applied`  
    **Note:**  The apply tool will only modify objects that are different from the current objects in the cluster. If the objects you are creating already exist in the cluster, it will simply exit successfully without making any changes.
- View the last state that was applied to the object: `kubectl apply -f myobj.yaml view-last-applied`
- Destroying the object:`kubectl delete -f obj.yaml`  
 or [Using resource type and name]  
`kubectl delete <resource-name> <obj-name>`

### Labeling and Annotating Objects
- Check description of an object [A service named kubernetes] and check the ***Labels*** : `kubectl describe services kubernetes`
- To add the ***color=red*** label to the service named ***kubernetes*** : `kubectl label services kubernetes color=red`
- Againg check description to verify
- Remove the label : `kubectl label services kubernetes color-`


### Debugging Commands
- Check the logs of a pod: `kubectl logs <pod-name>`
    - To choose among multiple containers in a pod: `kubectl logs <pod-name> -c` [c=choose]
    - To continually stream the logs: `kubectl logs <pod-name> -f` [f(follow)]

- To get interactive shell inside a running container: `kubectl exec -it <pod-name> -- bash` or `kubectl exec -it <pod-name> -- bin/sh`
    - Access container from a specific namespace: `kubectl exec -it <pod-name> --namespace=<namespace_name> -- bin/sh`
    - If bash or some other terminal not available within the container: `kubectl attach -it <pod-name>`  
    **Note:** The attach command is similar to kubectl logs but will allow you to send input to the running process, assuming that process is set up to read from standard input.

- Copying files to and from the container to local machine: `kubectl cp <pod-name>:<path_to_remote_file> <path_to_local_file>`

- To forward network traffic from the local machine to the Pod: `kubectl port-forward <pod-name> 8080:80`  
 **Note:** opens up a connection that forwards traffic from the local machine on port 8080 to the remote container on port 80  

- To see a list of the latest 10 events on all objects in a given namespace: `kubectl get events`
    - To see events in all namespaces: `kubectl get events -A`
    - To stream events as they happen: `kubectl get events --watch`

-  To see resources usage by either nodes or Pods:
`kubectl top nodes` or `kubectl top pods`
    - Check in all namespaces: `kubectl top pods --all-namespaces`

### Cluster Management
- For repair or upgrades of a machine, you need to safely remove the machine from the cluster. Use `kubectl cordon` followed by `kubectl drain`   
**Note**: When you cordon a node, you prevent future Pods from being scheduled onto that machine. When you drain a node, you remove any Pods that are currently running on that machine.
- Once the machine is repaired, you can use `kubectl uncordon` to re-enable Pods scheduling onto the node

**Note:** it’s only necessary if the machine will be out of service long enough that you want the Pods to move to a different machine.

### Help
`kubectl help`  
`kubectl help <command-name>`
