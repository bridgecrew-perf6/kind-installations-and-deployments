# kind-installations-and-deployments

## About kind: 

kind is a tool for running local Kubernetes clusters using Docker container "nodes". kind was primarily designed for testing Kubernetes itself, but may be used for local development or CI.

## Install kubectl

Install kubectl with Homebrew on MacOS:

```
brew install kubectl
```

Make sure version of installation is up-to-date:

```
kubectl version --client
```

Link for the installation: 
https://kubernetes.io/docs/tasks/tools/install-kubectl-macos/#install-with-homebrew-on-macos


## Install kind

After installing kubectl, install kind. 

Instal kind with homebrew: 

```
brew install kind
```

## Creating Cluster and Node Images:

Use kind version to find a complete listing of images created for a kind release.

```
kind version
```

Usually there is already a default node image provided. <br> 
If you want to build your own node image, it can be built by the following command:

```
kind build node-image
```

Create a cluster: 
this will bootstrap a Kubernetes cluster using a pre-built node image,

```
kind create cluster
```

optional flags: <br>

--name flag to assign the cluster a different context name. <br>
--wait 30s or –wait 5m flag and specify a timeout to block create cluster command until the control plane reaches a ready status <br>

You should be able to see something like this: <br> 
<img width="468" alt="image" src="https://user-images.githubusercontent.com/61640858/173143338-63e353eb-543d-477f-a6c7-5e860a285ad1.png">

------------------------------------------------------------------------------------------------------------------------- <br>
Side Notes: <br>
What is a node image? <br>
It is a Docker image for running nested containers, systemd, and Kubernetes components. The image is built on top of a “base” image <br>(https://kind.sigs.k8s.io/docs/design/base-image)
More information about node image is on https://kind.sigs.k8s.io/docs/design/node-image <br>
------------------------------------------------------------------------------------------------------------------------- <br>


If you see the following error, that means you have not installed or opened docker: <br>
```
ERROR: failed to create cluster: failed to get docker info: command "docker info --format '{{json .}}'" failed with error: exec: "docker": executable file not found in $PATH
```

If you did not install docker, use the following link to install: 
https://docs.docker.com/get-docker/

## Interacting With Cluster

By default, the cluster access configuration is stored in ${HOME}/.kube/config if $KUBECONFIG environment variable is not set. <br>
Get a list of clusters that you have set up: <br>
```
$ kind get clusters
kind
```
you can also check via docker ps command: 
```
$ docker ps
CONTAINER ID   IMAGE                  COMMAND                  CREATED        STATUS          PORTS                       NAMES
b6fafca5fe17   kindest/node:v1.24.0   "/usr/local/bin/entr…"   15 hours ago   Up 31 minutes   127.0.0.1:59161->6443/tcp   kind-control-plane
```

Another way to confirm it is through kubectl get nodes
```
$ kubectl get nodes
NAME                 STATUS   ROLES           AGE   VERSION
kind-control-plane   Ready    control-plane   15h   v1.24.0
```

delete a cluster: 
```
kind delete cluster
```

Get information about your cluster using this template: 
kubectl cluster-info --context kind-<cluster name, eg: kind> <br>

You should be able to see the following result: 

```
$ kubectl cluster-info --context kind-kind
Kubernetes control plane is running at https://127.0.0.1:59161
CoreDNS is running at https://127.0.0.1:59161/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
```

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.

## Creating Clusters using yaml file 
You can configure yaml files to create clusters with different settings.

### Multi-node clusters
Build a configure yaml file according to the following: 

```
# three node (two workers) cluster config
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
- role: worker
- role: worker
```

This creates ion control plane nodes and two worker nodes or data plane nodes. <br>
Run the following command: 
```
kind create cluster --config kind-multi-node-clusters-config.yaml --name kind-2
```

Note: <br>
1. need to use –name <cluster name> to create cluster with different name from exisitng clusters. If not, the folowing error will occur: 
```
$ kind create cluster --config kind-multi-node-clusters-config.yaml
ERROR: failed to create cluster: node(s) already exist for a cluster with the name "kind"
```
  
2. capital letter is not allowed in naming the clusters. If not, the folling error will be shown: 
```
ERROR: failed to create cluster: 'kind-HA' is not a valid cluster name, cluster names must match `^[a-z0-9.-]+$`
```

### Control-plane HA

You can also build multiple control planes to form high availiabiilty mode in order to ensure more control-planes are available: 
```
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
- role: control-plane
- role: control-plane
- role: worker
- role: worker
- role: worker
```

Execute the following command to create the cluster
```
kind create cluster --config kind-multi-node-clusters-config.yaml --name kind-ha
```

### Mapping ports to the host machine

```
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
  extraPortMappings:
  - containerPort: 80
    hostPort: 80
    listenAddress: "0.0.0.0" # Optional, defaults to "0.0.0.0"
    protocol: udp # Optional, defaults to tcp
  
```
  
Notes: "hostPort" is the port on the local computer that kind is residing on, and "containerPort" is port on the docker container, which encapsulates the microservice. "listenAddress" is the IP address of the interface on the "containerPort". So "containerPort," the server in this case, is listening for any request that is sent from "hostPort" on client.
