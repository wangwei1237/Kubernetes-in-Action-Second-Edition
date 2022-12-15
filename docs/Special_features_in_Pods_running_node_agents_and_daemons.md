# 16.2  Special features in Pods running node agents and daemons

Unlike regular workloads, which are usually isolated from the node they run on, node agents and daemons typically require greater access to the node. As you know, the containers running in a Pod can’t access the devices and files of the node, or all the system calls to the node’s kernel because they live in their own Linux namespaces (see chapter 2). If you want a daemon, agent, or other workload running in a Pod to be exempt from this restriction, you must specify this in the Pod manifest.

To explain how you can do this, look at the DaemonSets in the `kube-system` namespace. If you run Kubernetes via kind, your cluster should contain the two DaemonSets as follows:

```shell
$ kubectl get ds -n kube-system
NAME         DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR      AGE
kindnet      3         3         3       3            3           <none>             23h
kube-proxy   3         3         3       3            3           kubernetes.io...   23h
```

If you don’t use kind, the list of DaemonSets in `kube-system` may look different, but you should find the `kube-proxy` DaemonSet in most clusters, so I’ll focus on this one.

## 16.2.1  Giving containers access to the OS kernel

The operating system kernel provides system calls that programs can use to interact with the operating system and hardware. Some of these calls are harmless, while others could negatively affect the operation of the node or the other containers running on it. For this reason, containers are not allowed to execute these calls unless explicitly allowed to do so. This can be achieved in two ways. You can give the container full access to the kernel or to groups of system calls by specifying the capabilities to be given to the container.

#### Running a privileged container

If you want to give a process running in a container full access to the operating system kernel, you can mark the container as privileged. You can see how to do this by inspecting the Pod template in the `kube-proxy` DaemonSet as follows:

```shell
$ kubectl -n kube-system get ds kube-proxy -o yaml
apiVersion: apps/v1
kind: DaemonSet
spec:
  template:
    spec:
      containers:
      - name: kube-proxy
        securityContext:
          privileged: true
    ...
```

The `kube-proxy` DaemonSet runs Pods with a single container, also called kube-proxy. In the `securityContext` section of this container’s definition, the `privileged` flag is set to `true`. This gives the process running in the `kube-proxy` container root access to the host’s kernel and allows it to modify the node’s network packet filtering rules. As you’ll learn in chapter 19, Kubernetes Services are implemented this way.

#### Giving a container access to specific capabilities

A privileged container bypasses all kernel permission checks and thus has full access to the kernel, whereas a node agent or daemon typically only needs access to a subset of the system calls provided by the kernel. From a security perspective, running such workloads as privileged is far from ideal. Instead, you should grant the workload access to only the minimum set of system calls it needs to do its job. You achieve this by specifying the capabilities that it needs in the container definition.

The `kube-proxy` DaemonSet doesn’t use capabilities, but other DaemonSets in the `kube-system` namespace may do so. An example is the `kindnet` DaemonSet, which sets up the pod network in a kind-provisioned cluster. The capabilities listed in the Pod template are as follows:

```shell
$ kubectl -n kube-system get ds kindnet -o yaml
apiVersion: apps/v1
kind: DaemonSet
spec:
  template:
    spec:
      containers:
      - name: kindnet-cni
        securityContext:
          capabilities:
            add:
            - NET_RAW
            - NET_ADMIN
          privileged: false
```

Instead of being fully privileged, the capabilities `NET_RAW` and `NET_ADMIN` are added to the container. According to the capabilities man pages, which you can display with the `man capabilities` command on a Linux system, the `NET_RAW` capability allows the container to use special socket types and bind to any address, while the `NET_ADMIN` capability allows various privileged network-related operations such as interface configuration, firewall management, changing routing tables, and so on. Things you’d expect from a container that sets up the networking for all other Pods on a Node.

## 16.2.2  Accessing the node’s filesystem

A node agent or daemon may need to access the host node’s file system. For example, a node agent deployed through a DaemonSet could be used to install software packages on all cluster nodes.

You already learned in chapter 7 how to give a Pod’s container access to the host node’s file system via the `hostPath` volume, so I won’t go into it again, but it’s interesting to see how this volume type is used in the context of a daemon pod.

Let’s take another look at the `kube-proxy` DaemonSet. In the Pod template, you’ll find two `hostPath` volumes, as shown here:

```shell
$ kubectl -n kube-system get ds kube-proxy -o yaml
apiVersion: apps/v1
kind: DaemonSet
spec:
  template:
    spec:
      volumes:
      - hostPath:
          path: /run/xtables.lock
          type: FileOrCreate
        name: xtables-lock
      - hostPath:
          path: /lib/modules
          type: ""
        name: lib-modules
```

The first volume allows the process in the kube-proxy daemon Pod to access the node’s `xtables.lock` file, which is used by the `iptables` or `nftables` tools that the process uses to manipulate the node’s IP packet filtering. The other `hostPath` volume allows the process to access the kernel modules that are installed on the node.

## 16.2.3  Using the node’s network and other namespaces

As you know, each Pod gets its own network interface. However, you may want some of your Pods, especially those deployed through a DaemonSet, to use the node’s network interface(s) instead of having their own. The Pods deployed through the `kube-proxy` DaemonSet use this approach. You can see this by examining the Pod template as follows:

```shell
$ kubectl -n kube-system get ds kube-proxy -o yaml
apiVersion: apps/v1
kind: DaemonSet
spec:
  template:
    spec:
      dnsPolicy: ClusterFirst
      hostNetwork: true
```

In the Pod’s `spec`, the `hostNetwork` field is set to `true`. This causes the Pod to use the host Node’s network environment (devices, stacks, and ports) instead of having its own, just like all other processes that run directly on the node and not in a container. This means that the Pod won’t even get its own IP address but will use the Node’s address(es). If you list the Pods in the `kube-system` Namespace with the `-o wide` option as follows, you’ll see that the IPs of the `kube-proxy` Pods match the IPs of their respective host Nodes.

```shell
$ kubectl -n kube-system get po -o wide
NAME               READY   STATUS    RESTARTS   AGE   IP           ...                
kube-proxy-gj9pd   1/1     Running   0          90m   172.18.0.4   ...
kube-proxy-rhjqr   1/1     Running   0          90m   172.18.0.2   ...
kube-proxy-vq5g8   1/1     Running   0          90m   172.18.0.3   ...
```

Configuring daemon Pods to use the host node’s network is useful when the process running in the Pod needs to be accessible through a network port at the node’s IP address.

{% hint style='info' %}
NOTE

Another option is for the Pod to use its own network, but forward one or more host ports to the container by using the `hostPort` field in the container’s port list. You’ll learn how to do this later.
{% endhint %}

Containers in a Pod configured with `hostNetwork: true` continue to use the other namespace types, so they remain isolated from the node in other respects. For example, they use their own IPC and PID namespaces, so they can’t see the other processes or communicate with them via inter-process communication. If you want a daemon Pod to use the node’s IPC and PID namespaces, you can configure this using the `hostIPC` and `hostPID` properties in the Pod’s `spec`.

## 16.2.4  Marking daemon Pods as critical

A node can run a few system Pods and many Pods with regular workloads. You don’t want Kubernetes to treat these two groups of Pods the same, as the system Pods are probably more important than the non-system Pods. For example, if a system Pod can’t be scheduled to a Node because the Node is already full, Kubernetes should evict some of the non-system Pods to make room for the system Pod.

#### Introducing Priority Classes

By default, Pods deployed via a DaemonSet are no more important than Pods deployed via Deployments or StatefulSets. To mark your daemon Pods as more or less important, you use Pod priority classes. These are represented by the PriorityClass object. You can list them as follows:

```shell
$ kubectl get priorityclasses
NAME                      VALUE        GLOBAL-DEFAULT   AGE
system-cluster-critical   2000000000   false            9h
system-node-critical      2000001000   false            9h
```

Each cluster usually comes with two priority classes: `system-cluster-critical` and `system-node-critical`, but you can also create your own. As the name implies, Pods in the `system-cluster-critical` class are critical to the operation of the cluster. Pods in the `system-node-critical` class are critical to the operation of individual nodes, meaning they can’t be moved to a different node.

You can learn more about the priority classes defined in your cluster by using the `kubectl describe priorityclasses` command as follows:

```shell
$ kubectl describe priorityclasses
Name:           system-cluster-critical
Value:          2000000000
GlobalDefault:  false
Description:    Used for system critical pods that must run in the cluster, but can be moved to another node if necessary.
Annotations:    <none>
Events:         <none>
 
Name:           system-node-critical
Value:          2000001000
GlobalDefault:  false
Description:    Used for system critical pods that must not be moved from their current node.
Annotations:    <none>
Events:         <none>
```

As you can see, each priority class has a value. The higher the value, the higher the priority. The preemption policy in each class determines whether or not Pods with lower priority should be evicted when a Pod with that class is scheduled to an overbooked Node.

You specify which priority class a Pod belongs to by specifying the class name in the `priorityClassName` field of the Pod’s `spec` section. For example, the `kube-proxy` DaemonSet sets the priority class of its Pods to `system-node-critical`. You can see this as follows:

```shell
$ kubectl -n kube-system get ds kube-proxy -o yaml
apiVersion: apps/v1
kind: DaemonSet
spec:
  template:
    spec:
      priorityClassName: system-node-critical
```

The priority class of the `kube-proxy` Pods ensures that the kube-proxy Pods have a higher priority than the other Pods, since a node can’t function properly without a `kube-proxy` Pod (Pods on the Node can’t use Kubernetes Services).

When you create your own DaemonSets to run other node agents that are critical to the operation of a node, remember to set the `priorityClassName` appropriately.