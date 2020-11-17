# 3.1.3   Running a local cluster using kind (Kubernetes in Docker)
An alternative to Minikube, although not as mature, is kind (Kubernetes-in-Docker). Instead of running Kubernetes in a virtual machine or directly on the host, kind runs each Kubernetes cluster node inside a container. Unlike Minikube, this allows it to create multi-node clusters by starting several containers. The actual application containers that you deploy to Kubernetes then run within these node containers. The system is shown in the next figure.

Figure 3.4 Running a multi-node Kubernetes cluster using kind
![Figure 3.4 Running a multi-node Kubernetes cluster using kind](../images/3.4.png)

In the previous chapter I mentioned that a process that runs in a container actually runs in the host OS. This means that when you run Kubernetes using kind, all Kubernetes components run in your host OS. The applications you deploy to the Kubernetes cluster also run in your host OS.

This makes kind the perfect tool for development and testing, as everything runs locally and you can debug running processes as easily as when you run them outside of a container. I prefer to use this approach when I develop apps on Kubernetes, as it allows me to do magical things like run network traffic analysis tools such as Wireshark or even my web browser inside the containers that run my applications. I use a tool called nsenter that allows me to run these tools in the network or other namespaces of the container.

If you’re new to Kubernetes, the safest bet is to start with Minikube, but if you’re feeling adventurous, here’s how to get started with kind.

## Installing kind
Just like Minikube, kind consists of a single binary executable file. To install it, refer to the installation instructions at https://kind.sigs.k8s.io/docs/user/quick-start/. On macOS and Linux, the command to install it is as follows:

```
$ curl -Lo ./kind https://github.com/kubernetes-sigs/kind/releases/
[CA] download/v0.7.0/kind-$(uname)-amd64 && \
[CA] chmod +x ./kind && \
[CA] mv ./kind /some-dir-in-your-PATH/kind
```

Check the documentation to see what the latest version is and use it instead of v0.7.0 in the above example. Also, replace /some-dir-in-your-PATH/ with an actual directory in your path.

{% hint style='info' %}
NOTE
 Docker must be installed on your system to use kind.
{% endhint %}


## Starting a Kubernetes cluster with kind
Starting a new cluster is as easy as it is with Minikube. Execute the following command:

`$ kind create cluster`

Like Minikube, kind configures kubectl to use the cluster that it creates.

## Starting a multi-node cluster with kind
Kind runs a single-node cluster by default. If you want to run a cluster with multiple worker nodes, you must first create a configuration file named kind-multi-node.yaml with the following content (you can find the file in the book’s code archive, directory Chapter03/):

```
Listing 3.3 Config file for running a three-node cluster with kind
kind: Cluster
apiVersion: kind.sigs.k8s.io/v1alpha3
nodes:
- role: control-plane
- role: worker
- role: worker
```

With the file in place, create the cluster using the following command:

`$ kind create cluster --config kind-multi-node.yaml`

## Listing worker nodes
At the time of this writing, kind doesn’t provide a command to check the status of the cluster, but you can list cluster nodes using kind get nodes, as shown in the next listing.

```
Listing 3.4 Listing nodes using kind get nodes
$ kind get nodes
kind-worker2
kind-worker
kind-control-plane
```
{% hint style='info' %}
NOTE

Due to width restrictions, the node names control-plane, worker1, and worker2 are used instead of the actual node names throughout this book.
{% endhint %}

Since each node runs as a container, you can also see the nodes by listing the running containers using docker ps, as the next listing shows.

```
Listing 3.5 Displaying kind nodes running as containers
$ docker ps
CONTAINER ID    IMAGE                   ...    NAMES
45d0f712eac0    kindest/node:v1.18.2    ...    kind-worker2
d1e88e98e3ae    kindest/node:v1.18.2    ...    kind-worker
4b7751144ca4    kindest/node:v1.18.2    ...    kind-control-plane
```

## Logging into cluster nodes provisioned by kind
Unlike Minikube, where you use minikube ssh to log into the node if you want to explore the processes running inside it, with kind you use docker exec. For example, to enter the node called kind-control-plane, run:
`$ docker exec -it kind-control-plane bash`

Instead of using Docker to run containers, nodes created by kind use the CRI-O container runtime, which I mentioned in the previous chapter as a lightweight alternative to Docker. The crictl CLI tool is used to interact with CRI-O. Its use is very similar to that of the docker tool. After logging into the node, list the containers running in it by running crictl ps instead of docker ps. An example of the command’s output is shown in the next listing:

```
Listing 3.6 Listing containers inside a cluster node provisioned with kind
root@kind-control-plane:/# crictl ps
CONTAINER ID    IMAGE           CREATED      STATE     NAME
c7f44d171fb72   eb516548c180f   15 min ago   Running   coredns        ...
cce9c0261854c   eb516548c180f   15 min ago   Running   coredns        ...
e6522aae66fcc   d428039608992   16 min ago   Running   kube-proxy     ...
6b2dc4bbfee0c   ef97cccdfdb50   16 min ago   Running   kindnet-cni    ...
c3e66dfe44deb   be321f2ded3f3   16 min ago   Running   kube-apiserver ...
```