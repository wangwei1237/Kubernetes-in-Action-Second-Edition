# 5.5 Running additional containers at pod startup
When a pod contains more than one container, all the containers are started in parallel. Kubernetes doesn’t yet provide a mechanism to specify whether a container depends on another container, which would allow you to ensure that one is started before the other. However, Kubernetes allows you to run a sequence of containers to initialize the pod before its main containers start. This special type of container is explained in this section.

## Introducing init containers
A pod manifest can specify a list of containers to run when the pod starts and before the pod’s normal containers are started. These containers are intended to initialize the pod and are appropriately called init containers. As the following figure shows, they run one after the other and must all finish successfully before the main containers of the pod are started.

Figure 5.11 Time sequence showing how a pod’s init and regular containers are started

![](../images/5.11.png)

Init containers are like the pod’s regular containers, but they don’t run in parallel - only one init container runs at a time.

### Understanding what init containers can do
Init containers are typically added to pods to achieve the following:

* Initialize files in the volumes used by the pod’s main containers. This includes retrieving certificates and private keys used by the main container from secure certificate stores, generating config files, downloading data, and so on.
* Initialize the pod’s networking system. Because all containers of the pod share the same network namespaces, and thus the network interfaces and configuration, any changes made to it by an init container also affect the main container.
* Delay the start of the pod’s main containers until a precondition is met. For example, if the main container relies on another service being available before the container is started, an init container can block until this service is ready.
* Notify an external service that the pod is about to start running. In special cases where an external system must be notified when a new instance of the application is started, an init container can be used to deliver this notification.

You could perform these operations in the main container itself but using an init container is sometimes a better option and can have other advantages. Let’s see why.

### Understanding when moving initialization code to init containers makes sense
Using an init container to perform initialization tasks doesn’t require the main container image to be rebuilt and allows a single init container image to be reused with many different applications. This is especially useful if you want to inject the same infrastructure-specific initialization code into all your pods. Using an init container also ensures that this initialization is complete before any of the (possibly multiple) main containers start.

Another important reason is security. By moving tools or data that could be used by an attacker to compromise your cluster from the main container to an init container, you reduce the pod’s attack surface.

For example, imagine that the pod must be registered with an external system. The pod needs some sort of secret token to authenticate against this system. If the registration procedure is performed by the main container, this secret token must be present in its filesystem. If the application running in the main container has a vulnerability that allows an attacker to read arbitrary files on the filesystem, the attacker may be able to obtain this token. By performing the registration from an init container, the token must be available only in the filesystem of the init container, which an attacker can’t easily compromise.

## Adding init containers to a pod
In a pod manifest, init containers are defined in the `initContainers` field in the spec section, just as regular containers are defined in its `containers` field.

### Adding two containers to the kubia-ssl pod
Let’s look at an example of adding two init containers to the kubia pod. The first init container emulates an initialization procedure. It runs for 5 seconds, while printing a few lines of text to standard output.

The second init container performs a network connectivity test by using the `ping` command to check if a specific IP address is reachable from within the pod. If the IP address is not specified, the address 1.1.1.1 is used. You’ll find the `Dockerfiles` and other artifacts for both images in the book’s code archive, if you want to build them yourself. Alternatively, you can use the pre-build images specified in the following listing.

A pod manifest containing these two init containers is in the `kubia-init.yaml` file. The following listing shows how the init containers are defined.

```YAML
Listing 5.9 Defining init containers in a pod manifest: kubia-init.yaml
apiVersion: v1
kind: Pod
metadata:
  name: kubia-init
spec:
  initContainers:
  - name: init-demo
    image: luksa/init-demo:1.0
  - name: network-check
    image: luksa/network-connectivity-checker:1.0
  containers:
  - name: kubia
    image: luksa/kubia:1.0
    ports:
    - name: http
      containerPort: 8080
  - name: envoy
    image: luksa/kubia-ssl-proxy:1.0
    ports:
    - name: https
      containerPort: 8443
    - name: admin
      containerPort: 9901
```

As you can see, the definition of an init container is almost trivial. It’s sufficient to specify only the `name` and `image` for each container.

{% hint style='info' %}
NOTE

Container names must be unique within the union of all init and regular containers.
{% endhint %}

### Deploying a pod with init containers
Before you create the pod from the manifest file, run the following command in a separate terminal so you can see how the pod’s status changes as the init and regular containers start:

```shell
$ kubectl get pods -w
```

You’ll also want to watch events in another terminal using the following command:

```shell
$ kubectl get events -w
```

When ready, create the pod by running the apply command:

```shell
$ kubectl apply -f kubia-init.yaml
```

### Inspecting the startup of a pod with init containers
As the pod starts up, inspect the events that the `kubectl get events -w` command prints. The following listing shows what you should see.

```shell
Listing 5.10 Pod events showing how the execution of init containers
TYPE     REASON      MESSAGE
Normal   Scheduled   Successfully assigned pod to worker2
Normal   Pulling     Pulling image "luksa/init-demo:1.0"
Normal   Pulled      Successfully pulled image
Normal   Created     Created container init-demo
Normal   Started     Started container init-demo
Normal   Pulling     Pulling image "luksa/network-connec...
Normal   Pulled      Successfully pulled image
Normal   Created     Created container network-check
Normal   Started     Started container network-check
Normal   Pulled      Container image "luksa/kubia:1.0"
                     already present on machine
Normal   Created     Created container kubia
Normal   Started     Started container kubia
Normal   Pulled      Container image "luksa/kubia-ssl-
                     proxy:1.0" already present on machine
Normal   Created     Created container envoy
Normal   Started     Started container envoy
```

The listing shows the order in which the containers are started. The `init-demo` container is started first. When it completes, the `network-check` container is started, and when it completes, the two main containers, `kubia` and `envoy`, are started.

Now inspect the transitions of the pod’s status in the other terminal. They are shown in the next listing.

```shell
Listing 5.11 Pod status changes during startup involving init containers
NAME         READY   STATUS            RESTARTS   AGE
kubia-init   0/2     Pending           0          0s
kubia-init   0/2     Pending           0          0s
kubia-init   0/2     Init:0/2          0          0s
kubia-init   0/2     Init:0/2          0          1s
kubia-init   0/2     Init:1/2          0          6s
kubia-init   0/2     PodInitializing   0          7s
kubia-init   2/2     Running           0          8s
```

As the listing shows, when the init containers run, the pod’s status shows the number of init containers that have completed and the total number. When all init containers are done, the pod’s status is displayed as `PodInitializing`. At this point, the images of the main containers are pulled. When the containers start, the status changes to `Running`.

## Inspecting init containers
While the init containers run and after they have finished, you can display their logs and enter the running container, just as you can with regular containers.

### Displaying the logs of an init container
The standard and error output, into which each init container can write, are captured exactly as they are for regular containers. The logs of an init container can be displayed using the `kubectl` `logs` command by specifying the name of the container with the `-c` option. To display the logs of the `network-check` container in the `kubia-init` pod, run the command shown in the following listing.

```shell
Listing 5.12 Displaying the logs of an init container
$ kubectl logs kubia-init -c network-check
Checking network connectivity to 1.1.1.1 ...
Host appears to be reachable
```

The logs show that the `network-check` init container ran without errors. In the next chapter, you’ll see what happens if an init container fails.

### Entering a running init container
You can use the `kubectl exec` command to run a shell or a different command inside an init container the same way you can with regular containers, but you can only do this before the init container terminates. If you’d like to try this yourself, create a pod from the `kubia-init-slow.yaml` file, which makes the `init-demo` container run for 60 seconds. When the pod starts, run a shell in the container with the following command:

```shell
$ kubectl exec -it kubia-init-slow -c init-demo -- sh
```

You can use the shell to explore the container from the inside, but not for long. When the container’s main process exits after 60 seconds, the shell process is also terminated.

You typically enter a running init container only when it fails to complete in time, and you want to find the cause. During normal operation, the init container terminates before you can run the `kubectl exec` command.