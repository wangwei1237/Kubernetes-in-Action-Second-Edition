# Creating pods from YAML or JSON files
With the information you learned in the previous sections, you can now start creating pods. In chapter 3, you created them using the imperative command `kubectl create`, but pods and other Kubernetes objects are usually created by creating a JSON or YAML manifest file and posting it to the Kubernetes API, as you’ve already learned in the previous chapter.

{% hint style='info' %}
NOTE

The decision whether to use YAML or JSON to define your objects is yours. Most people prefer to use YAML because it’s slightly more human-friendly and allows you to add comments to the object definition.
{% endhint %}


By using YAML files to define the structure of your application, you don’t need shell scripts to make the process of deploying your applications repeatable, and you can keep a history of all changes by storing these files in a VCS (Version Control System). Just like you store code.

In fact, the application manifests of the exercises in this book are all stored in a VCS. You can find them on GitHub at github.com/luksa/kubernetes-in-action-2ed.

## Creating a YAML manifest for a pod
In the previous chapter you learned how to retrieve and examine the YAML manifests of existing API objects. Now you’ll create an object manifest from scratch.

You’ll start by creating a file called `kubia.yaml` on your computer, in a file directory of your choice. You can also find the file in the book’s code archive, available on GitHub. The file is in the `Chapter04/` directory. The following listing shows its contents.

```YAML
Listing 5.1 A basic pod manifest: kubia.yaml
apiVersion: v1
kind: Pod
metadata:    
  name: kubia
spec:    
  containers:     
  - name: kubia
    image: luksa/kubia:1.0
    ports:        
    - containerPort: 8080
```

I’m sure you’ll agree that this pod manifest is much easier to understand than the mammoth of a manifest representing the Node object, which you saw in the previous chapter. But once you post this pod object manifest to the API and then read it back, it won’t be much different.

The manifest in listing 5.1 is short only because it does not yet contain all the fields that a pod object gets after it is created through the API. For example, you’ll notice that the `metadata` section contains only a single field and that the `status` section is completely missing. Once you create the object from this manifest, this will no longer be the case. But we’ll get to that later.

Before you create the object, let’s examine the manifest in detail. It uses version `v1` of the Kubernetes API to describe the object. The object is a `Pod` called `kubia`. The pod consists of a single container called kubia, based on the `luksa/kubia:1.0` image. The pod definition also specifies that the application in the container listens on port `8080`.

{% hint style='info' %}
TIP

Whenever you want to create a pod manifest from scratch, you can also use the following command to create the file and then edit it to add more fields: `kubectl run kubia --image=luksa/kubia:1.0 --dry-run=client -o yaml > mypod.yaml`. The `--dry-run=client` flag tells kubectl to output the definition instead of actually creating the object via the API.
{% endhint %}


The fields in the YAML file are self-explanatory, but if you want more information about each field or want to know what additional fields you can add, remember to use the `kubectl explain pods` command.

## Creating the Pod object from the YAML file
After you’ve prepared the manifest file for your pod, you can now create the object by posting the file to the Kubernetes API.

### Creating objects by applying the manifest file to the cluster
When you post the manifest to the API, you are directing Kubernetes to apply the manifest to the cluster. That’s why the `kubectl` sub-command that does this is called `apply`. Let’s use it to create the pod:

```shell
$ kubectl apply -f kubia.yaml
pod “kubia” created
```

### Updating objects by modifying the manifest file and re-applying it
The `kubectl` `apply` command is used for creating objects as well as for making changes to existing objects. If you later decide to make changes to your pod object, you can simply edit the `kubia.yaml` file and run the apply command again. Some of the pod’s fields aren’t mutable, so the update may fail, but you can always delete the pod and then create it again. You’ll learn how to delete pods and other objects at the end of this chapter.

### Retrieving the full manifest of a running pod
The pod object is now part of the cluster configuration. You can now read it back from the API to see the full object manifest with the following command:

```shell
$ kubectl get po kubia -o yaml
```

If you run this command, you’ll notice that the manifest has grown considerably compared to the one in the `kubia.yaml` file. You’ll see that the `metadata` section is now much bigger, and the object now has a `status` section. The `spec` section has also grown by several fields. You can use `kubectl explain` to learn more about these new fields, but most of them will be explained in this and the following chapters.

## Checking the newly created pod
Let’s use the basic `kubectl` commands to see how the pod is doing before we start interacting with the application running inside it.

### Quickly checking the status of a pod
Your Pod object has been created, but how do you know if the container in the pod is actually running? You can use the `kubectl get` command to see a summary of the pod:

```shell
$ kubectl get pod kubia
NAME     READY   STATUS    RESTARTS   AGE
kubia    1/1     Running   0          32s
```

You can see that the pod is running, but not much else. To see more, you can try the `kubectl get pod -o` wide or the `kubectl describe` command that you learned in the previous chapter.

### Using kubectl describe to see pod details
To display a more detailed view of the pod, use the `kubectl describe` command:

```shell
Listing 5.2 Using kubectl describe pod to inspect a pod
$ kubectl describe pod kubia
Name:         kubia
Namespace:    default
Priority:     0
Node:         worker2/172.18.0.4
Start Time:   Mon, 27 Jan 2020 12:53:28 +0100
...
```

The listing doesn’t show the entire output, but if you run the command yourself, you’ll see virtually all information that you’d see if you print the complete object manifest using the `kubectl get -o yaml` command.

### Inspecting events to see what happens beneath the surface
As in the previous chapter where you used the `describe` `node` command to inspect a Node object, the `describe pod` command should display several events related to the pod at the bottom of the output.

If you remember, these events aren’t part of the object itself, but are separate objects. Let’s print them to learn more about what happens when you create the pod object. The following listing shows all the events that were logged after creating the pod.

```shell
Listing 5.3 Events recorded after deploying a Pod object
$ kubectl get events
LAST SEEN   TYPE     REASON      OBJECT      MESSAGE
<unknown>   Normal   Scheduled   pod/kubia   Successfully assigned default/
                                             kubia to worker2
5m          Normal   Pulling     pod/kubia   Pulling image luksa/kubia:1.0
5m          Normal   Pulled      pod/kubia   Successfully pulled image
5m          Normal   Created     pod/kubia   Created container kubia
5m          Normal   Started     pod/kubia   Started container kubia
```

These events are printed in chronological order. The most recent event is at the bottom. You see that the pod was first assigned to one of the worker nodes, then the container image was pulled, then the container was created and finally started.

No warning events are displayed, so everything seems to be fine. If this is not the case in your cluster, you should read section 5.4 to learn how to troubleshoot pod failures.