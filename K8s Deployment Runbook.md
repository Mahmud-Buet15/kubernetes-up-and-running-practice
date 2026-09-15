---
title: K8s Service Deployment Runbook
tags: Kubernetes, Deployment, Backend
---

# K8s Service Deployment Runbook

This document can serve as a quick rundown on how to deploy services in kubernetes.

Feel free to reach out at shabab.noor@pathao.com for questions or suggestions.

**Happy deploying! :rocket:**

<br>

## Preparing and making image available

Crreate a Git tag
```bash=
git tag -a <TAG_NAME> -m "<TAG_MESSAGE>"
```

Push tag in remote repository
```bash=
git push origin <TAG_NAME>
```

Log in to Docker image repository if not already logged in
```bash=
docker login --help
docker login --username <USERNAME> --password <PASSWORD> <SERVER>
docker login -u <USERNAME> -p <PASSWORD> <SERVER>

# Example
docker login -u your.username -p your.passworrd harbor.pathaointernal.com
```
Pathao Docker image repository server: [`harbor.pathaointernal.com`](https://harbor.pathaointernal.com)

:::info
**&#9432;** You need an account in [`harbor.pathaointernal.com`](https://harbor.pathaointernal.com) before you can push images.

&nbsp; &nbsp; &nbsp;Contact Pillars team for access!
:::

Build Docker image
```bash=
docker build --help
docker build . --tag <IMAGE_NAME>:<TAG_NAME>
docker build . -t <IMAGE_NAME>:<TAG_NAME> 
```

Push docker image in image repository
```bash=
docker push <IMAGE_NAME>:[TAG_NAME]
docker image push <IMAGE_NAME>:[TAG_NAME]
```

<br>

## Deploying in Kubernetes

### *Prerequisite:* Getting Required Permissions

:::info
**&#9432;** Before being able to deploy, you need permission in the specific cluster and namespace!
:::

To request for permission, please create a **Merge Request** in the [`cluster-permission`](https://magic.pathao.com/platform/cluster-permission) repository.

Sample merge request: [!759](https://magic.pathao.com/platform/cluster-permission/-/merge_requests/759)

- Staging cluster: `clusters/gke-qa`


### Deploying in Kubernetes Cluster <> Pod

:::info
**&#9432;** Kubernetes in Pathao is hosted in Google Cloud Platform
:::

Log in to Google Cloud Platform using the CLI
```bash=
gcloud auth login
```

Check which Kubernetes clusters you have access to
```bash=
kubectl config get-contexts
```

You may need to run the following command to update local kubeconfig contexts
```bash=
gcloud container clusters get-credentials <CLUSTER_NAME> \
    --zone <ZONE_NAME> \
    --project <PROJECT_NAME>

# Example
gcloud container clusters get-credentials p-stageenv \
    --zone asia-east1-a \
    --project pathao-production-cloud
```
Example cluster name: `p-stageenv`

Select the Kubernetes cluster where you want to deploy
```bash=
kubectl config use-context <CONTEXT_NAME>

# Example
kubectl config use-context gke_pathao-production-cloud_asia-east1-a_p-stageenv
```
Example context_name: 
- For `p-stageenv` cluster `gke_pathao-production-cloud_asia-east1-a_p-stageenv`

Get Kubernetes pods
```bash=
kubectl get pods -n <NAMESPACE>
```
Example namespace: `hermes`

Edit deployment script
```bash=
kubectl edit -n <NAMESPACE> deployment/<POD_NAME>

# Example
kubectl edit -n hermes deployment/address_parse_pod
```
Example pod_name: `address_parse_pod`

:::success
**Done!** You should now have successfully deployed your service in kubernetes! :confetti_ball: 
:::

<br>

###  Additional Commands

#### Checking logs from pod
```bash=
kubectl logs -n <NAMESPACE> -f <POD_FULL_NAME>

# Example
kubectl logs -n hermes -f address-parse-pod-8587c7dcf4-pn882
```
Get `pod_full_name` from running `kubectl get pods -n <NAMESPACE>`

#### Scaling / Changing number of pods
```bash=
kubectl scale deploy -n <NAMESPACE> <POD_NAME> -n --replicas=<NO_OF_REPLICAS>

# Example
kubectl scale deploy -n hermes address-parse-pod --replicas=1
```

#### Checking existing services
Note: *both commands do the same*.
```bash=
kubectl get svc -n <NAMESPACE>
kubectl get services -n <NAMESPACE>

# Example
kubectl get svc -n hermes
kubectl get services -n hermes
```

#### Checking existing secrets
```bash=
kubectl get secret -n <NAMESPACE>

# Example
kubectl get secret -n hermes
```

#### Port forwarding from pod to local
```bash=
kubectl port-forward -n <NAMESPACE> svc/<SVC_NAME> <LOCAL_PORT>:<POD_PORT>

# Example
kubectl port-forward -n hermes svc/address-parse-svc 5000:5000
```

#### Patching a deployment
```bash=
# Expanded command
kc patch deployment/<SVC_NAME> \
    -p \
    "{
        \"spec\":{
            \"template\":{
                \"metadata\":{
                    \"annotations\":{
                        \"date\":\"`date +'%s'`\"
                    }
                }
            }
        }
    }" \
    -n <NAMESPACE>

# Collapsed command
kc patch deployment/<SVC_NAME> -p \ "{\"spec\":{\"template\":{\"metadata\":{\"annotations\":{\"date\":\"`date +'%s'`\"}}}}}" -n <NAMESPACE>

# Example
kc patch deployment/apt -p \ "{\"spec\":{\"template\":{\"metadata\":{\"annotations\":{\"date\":\"`date +'%s'`\"}}}}}" -n hermes
```

#### Deploying a service from scratch
Create secrets and deployment and service type `.yml` files

**`secrets.yml` File**
```yaml=
apiVersion: v1
kind: Secret
type: kubernetes.io/dockercfg
data:
  .dockercfg: {{DOCKER_CONFIG_SECRET}}
metadata:
  name: {{SECRET_NAME}}
```

**`deployment.yml` File**
```yaml=
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{NAME}}
  namespace: {{NAMESPACE}}
spec:
  replicas: 1
  selector:
    matchLabels:
      app: {{NAME}}
  template:
    metadata:
      name: {{NAME}}
      labels:
        app: {{NAME}}
    spec:
      containers:
        - name: {{NAME}}
          image: {{IMAGE_NAME}}:{{TAG}}
          imagePullPolicy: Always
          ports:
            - containerPort: {{PORT}}
              name: {{PORT_NAME}}
              protocol: TCP
          args:
            - run
      imagePullSecrets:
        - name: {{SECRET_NAME}}
```

**`service.yml` File**

```yaml=
apiVersion: v1
kind: Service
metadata:
  name: {{NAME}}
  namespace: {{NAMESPACE}}
spec:
  selector:
    app: {{NAME}}
  ports:
  - port: {{PORT}}
    targetPort: {{PORT_NAME}}
```

Then run the following commands in the correct cluster
```bash=
kubectl apply -f <path/to/secrets.yml> -n <NAMESPACE>

kubectl apply -f <path/to/deployment.yml> -n <NAMESPACE>

kubectl apply -f <path/to/service.yml> -n <NAMESPACE>
```


#### Deleting a pod
:::danger
:pushpin: **WARNING:** Be extremely careful when deleting a pod. 
- Make sure you are not in any `production` clusters. (Unless you know what you are doing or have a supervisor with you)
- Check if `kubectl scale` can be used instead
- Check if you are in the correct cluster using `kubectl config current-context`
:::

```bash=
kubectl delete -n <NAMESPACE> pod <POD_FULL_NAME>

# Example
kubectl delete -n hermes pod address-parse-pod-5b7596fff7-lc5xq
```
