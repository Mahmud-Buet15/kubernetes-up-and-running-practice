# Deploying a Kubernetes Cluster
## Installing Kubernetes with Google Kubernetes Engine (GKE)
- Resource link: https://cloud.google.com/kubernetes-engine/docs/deploy-app-cluster

Need to do the followings in the vm
### Set project ID
`gcloud config set project data-cloud-staging-210306`

### Install kubectl on Linux
Resource link: https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/

- Download the latest release with the command:  
`curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"`
- Install kubectl  
`sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl`
- Test to ensure the version you installed is up-to-date:  
`kubectl version --client`
- Install required plugins  
kubectl and other Kubernetes clients require an authentication plugin, `gke-gcloud-auth-plugin`, which uses the **Client-go Credential Plugins** framework to provide authentication tokens to communicate with GKE clusters.
    - Install the gke-gcloud-auth-plugin binary:  
    `gcloud components install gke-gcloud-auth-plugin` or `sudo apt-get install google-cloud-sdk-gke-gcloud-auth-plugin`
    - Check the gke-gcloud-auth-plugin binary version:   
    `gke-gcloud-auth-plugin --version`

### Install kubectl on Mac
Resource link: https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/

- Download the latest release with the command:  
    - Intel
    `curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/darwin/amd64/kubectl"`
    - Apple Silicon
    `curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/darwin/arm64/kubectl"`
- Make the kubectl binary executable: `chmod +x ./kubectl`
- Move the kubectl binary to a file location on your system `PATH`  
    - `sudo mv ./kubectl /usr/local/bin/kubectl`
    - `sudo chown root: /usr/local/bin/kubectl`
- Test to ensure the version you installed is up-to-date:  
`kubectl version --client`

- Install required plugins  
kubectl and other Kubernetes clients require an authentication plugin, `gke-gcloud-auth-plugin`, which uses the **Client-go Credential Plugins** framework to provide authentication tokens to communicate with GKE clusters.
    - Install the gke-gcloud-auth-plugin binary:  
    `gcloud components install gke-gcloud-auth-plugin`
    - Check the gke-gcloud-auth-plugin binary version:   
    `gke-gcloud-auth-plugin --version`


### Create a GKE cluster
- A cluster consists of at least one **cluster control plane** machine and multiple worker machines called **nodes**. Nodes are Compute Engine virtual machine (VM) instances that run the Kubernetes processes necessary to make them part of the cluster. 
- You deploy applications to clusters, and the applications run on the nodes.
- Create a cluster named kuar-cluster:  
`gcloud container clusters create kuar-cluster --location=asia-east1-a`  

    **Note:** May take few minutes


### Get authentication credentials for the cluster
After creating your cluster, you need to get authentication credentials to interact with the cluster:  
`gcloud container clusters get-credentials kuar-cluster --location asia-east1-a`


### Checking Cluster Status
- verify that your cluster is generally healthy  
`kubectl get componentstatuses`

### Cluster Components
#### Kubernetes Nodes
- list out all of the nodes in your cluster  
`kubectl get nodes`
- get more information about a specific node  
`kubectl describe nodes <node_name>`

#### Kubernetes Proxy
The Kubernetes proxy is responsible for routing network traffic to load-balanced services in the Kubernetes cluster. To do its job, the proxy must be present on every node in the cluster  
To see the proxies:  
`kubectl get daemonSets --namespace=kube-system kube-proxy`

#### Kubernetes DNS
Kubernetes also runs a DNS server, which provides naming and discovery for the services that are defined in the cluster. This DNS server also runs as a replicated service on the cluster
- The DNS service is run as a Kubernetes deployment, which manages these replicas. Check:  
`kubectl get deployments --namespace=kube-system kube-dns`
- There is also a Kubernetes service that performs load balancing for the DNS server  
`kubectl get services --namespace=kube-system kube-dns`

### Clean up
- Delete the application's Service by running:  
`kubectl delete service kuar-server`
- Delete your cluster by running:  
`gcloud container clusters delete kuar-cluster --location asia-east1-a`