# Deleting pods and other objects
If you’ve tried the exercises in this chapter and in chapter 2, several pods and other objects now exist in your cluster. To close this chapter, you’ll learn various ways to delete them. Deleting a pod will terminate its containers and remove them from the node. Deleting a Deployment object causes the deletion of its pods, whereas deleting a LoadBalancer-typed Service deprovisions the load balancer if one was provisioned.

## Deleting a pod by name
The easiest way to delete an object is to delete it by name.

### Deleting a single pod
Use the following command to remove the `kubia` pod from your cluster:

```shell
$ kubectl delete po kubia
pod "kubia" deleted
```

By deleting a pod, you state that you no longer want the pod or its containers to exist. The Kubelet shuts down the pod’s containers, removes all associated resources, such as log files, and notifies the API server after this process is complete. The Pod object is then removed.

{% hint style='info' %}
TIP

By default, the `kubectl delete` command waits until the object no longer exists. To skip the wait, run the command with the `--wait=false` option.
{% endhint %}

While the pod is in the process of shutting down, its status changes to `Terminating`:

```shell
$ kubectl get po kubia
NAME    READY   STATUS        RESTARTS   AGE
kubia   1/1     Terminating   0          35m
```

Knowing exactly how containers are shut down is important if you want your application to provide a good experience for its clients. This is explained in the next chapter, where we dive deeper into the life cycle of the pod and its containers.

{% hint style='info' %}
NOTE

If you’re familiar with Docker, you may wonder if you can stop a pod and start it again later, as you can with Docker containers. The answer is no. With Kubernetes, you can only remove a pod completely and create it again later.
{% endhint %}

### Deleting multiple pods with a single command
You can also delete multiple pods with a single command. If you ran the `kubia-init` and the `kubia-init-slow` pods, you can delete them both by specifying their names separated by a space, as follows:

```shell
$ kubectl delete po kubia-init kubia-init-slow
pod "kubia-init" deleted
pod "kubia-init-slow" deleted
```

## Deleting objects defined in manifest files
Whenever you create objects from a file, you can also delete them by passing the file to the `delete` command instead of specifying the name of the pod.

### Deleting objects by specifying the manifest file
You can delete the `kubia-ssl` pod, which you created from the `kubia-ssl.yaml` file, with the following command:

```shell
$ kubectl delete -f kubia-ssl.yaml
pod "kubia-ssl" deleted
```

In your case, the file contains only a single pod object, but you’ll typically come across files that contain several objects of different types that represent a complete application. This makes deploying and removing the application as easy as executing `kubectl apply -f app.yaml` and `kubectl delete -f app.yaml`, respectively.

### Deleting objects from multiple manifest files
Sometimes, an application is defined in several manifest files. You can specify multiple files by separating them with a comma. For example:

```shell
$ kubectl delete -f kubia.yaml,kubia-ssl.yaml
```

{% hint style='info' %}
NOTE

You can also apply several files at the same time using this syntax (for example: `kubectl apply -f kubia.yaml,kubia-ssl.yaml`).
{% endhint %}

I’ve never actually used this approach in the many years I’ve been using Kubernetes, but I often deploy all the manifest files from a file directory by specifying the directory name instead of the names of individual files. For example, you can deploy all the pods you created in this chapter again by running the following command in the base directory of this book’s code archive:

```shell
$ kubectl apply -f Chapter05/
```

This applies all files in the directory that have the correct file extension (`.yaml`, `.json`, and others). You can then delete the pods using the same method:

```shell
$ kubectl delete -f Chapter05/
```

{% hint style='info' %}
NOTE

Use the `--recursive` flag to also scan subdirectories.
{% endhint %}

## Deleting all pods
You’ve now removed all pods except `kubia-stdin` and the pods you created in chapter 3 using the `kubectl` `create deployment` command. Depending on how you’ve scaled the deployment, some of these pods should still be running:

```shell
$ kubectl get pods
NAME                    READY   STATUS    RESTARTS   AGE
kubia-stdin             1/1     Running   0          10m
kubia-9d785b578-58vhc   1/1     Running   0          1d
kubia-9d785b578-jmnj8   1/1     Running   0          1d
```

Instead of deleting these pods by name, we can delete them all using the `--all` option:

```shell
$ kubectl delete po --all
pod "kubia-stdin" deleted
pod "kubia-9d785b578-58vhc" deleted
pod "kubia-9d785b578-jmnj8" deleted
```

Now confirm that no pods exist by executing the `kubectl get pods` command again:

```shell
$ kubectl get po
NAME                    READY   STATUS    RESTARTS   AGE
kubia-9d785b578-cc6tk   1/1     Running   0          13s
kubia-9d785b578-h4gml   1/1     Running   0          13s
```

That was unexpected! Two pods are still running. If you look closely at their names, you’ll see that these aren’t the two you’ve just deleted. The `AGE` column also indicates that these are new pods. You can try to delete them as well, but you’ll see that no matter how often you delete them, new pods are created to replace them.

The reason why these pods keep popping up is because of the Deployment object. The controller responsible for bringing Deployment objects to life must ensure that the number of pods always matches the desired number of replicas specified in the object. When you delete a pod associated with the Deployment, the controller immediately creates a replacement pod.

To delete these pods, you must either scale the Deployment to zero or delete the object altogether. This would indicate that you no longer want this deployment or its pods to exist in your cluster.

## Deleting objects of most kinds
You can delete everything you’ve created so far - including the deployment, its pods, and the service - with the command shown in the next listing.

```shell
Listing 5.13 Deleting most objects regardless of type
$ kubectl delete all --all
pod "kubia-9d785b578-cc6tk" deleted
pod "kubia-9d785b578-h4gml" deleted
service "kubernetes" deleted
service "kubia" deleted
deployment.apps "kubia" deleted
replicaset.apps "kubia-9d785b578" deleted
```

The first `all` in the command indicates that you want to delete objects of all types. The `--all` option indicates that you want to delete all instances of each object type. You used this option in the previous section when you tried to delete all pods.

When deleting objects, `kubectl` prints the type and name of each deleted object. In the previous listing, you should see that it deleted the pods, the deployment, and the service, but also a so-called replica set object. You’ll learn what this is in chapter 11, where we take a closer look at deployments.

You’ll notice that the delete command also deletes the built-in `kubernetes` service. Don’t worry about this, as the service is automatically recreated after a few moments.

Certain objects aren’t deleted when using this method, because the keyword `all` does not include all object kinds. This is a precaution to prevent you from accidentally deleting objects that contain important information. The Event object kind is one example of this.

{% hint style='info' %}
NOTE

You can specify multiple object types in the `delete` command. For example, you can use `kubectl delete events,all --all` to delete events along with all object kinds included in `all`.
{% endhint %}
