# 13.4 Deleting a ReplicaSet
A ReplicaSet represents a group of Pod replicas that you manage as a unit. By creating a ReplicaSet object, you indicate that you want a specific number of Pod replicas based on a specific Pod template in your cluster. By deleting the ReplicaSet object, you indicate that you no longer want those Pods. So when you delete a ReplicaSet, all the Pods that belong to it are also deleted. This is done by the garbage collector, as explained earlier in this chapter.

## 13.4.1 Deleting a ReplicaSet and all associated Pods
To delete a ReplicaSet and all Pods it controls, run the following command:

```shell
$ kubectl delete rs kiada
replicaset.apps "kiada" deleted
```

As expected, this also deletes the Pods:

```shell
$ kubectl get pods -l app=kiada
NAME          READY   STATUS        RESTARTS   AGE
kiada-2dq4f   0/2     Terminating   0          7m29s
kiada-f5nff   0/2     Terminating   0          7m29s
kiada-khmj5   0/2     Terminating   0          7m29s
```

But in some cases, you don’t want that. So how can you prevent the garbage collector from removing the Pods? Before we get to that, recreate the ReplicaSet by reapplying the `rs.kiada.versionLabel.yaml` file.

## 13.4.2 Deleting a ReplicaSet while preserving the Pods
At the beginning of this chapter you learned that the label selector in a ReplicaSet is immutable. If you want to change the label selector, you have to delete the ReplicaSet object and create a new one. In doing so, however, you may not want the Pods to be deleted, because that would cause your service to become unavailable. Fortunately, you can tell Kubernetes to orphan the Pods instead of deleting them.

To preserve the Pods when you delete the ReplicaSet object, use the following command:

```shell
$ kubectl delete rs kiada --cascade=orphan
replicaset.apps "kiada" deleted
```

Now, if you list the Pods, you’ll find that they’ve been preserved. If you look at their manifests, you’ll notice that the ReplicaSet object has been removed from `ownerReferences`. These Pods are now orphaned, but if you create a new ReplicaSet with the same label selector, it’ll take these Pods under its wing. Apply the `rs.kiada.versionLabel.yaml` file again to see this for yourself.