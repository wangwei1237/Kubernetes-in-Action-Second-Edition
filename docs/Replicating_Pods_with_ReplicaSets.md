# 13 Replicating Pods with ReplicaSets
This chapter covers
* Replicating Pods with the ReplicaSet object
* Keeping Pods running when cluster nodes fail
* The reconciliation control loop in Kubernetes controllers
* API Object ownership and garbage collection

So far in this book, you’ve deployed workloads by creating Pod objects directly. In a production cluster, you might need to deploy dozens or even hundreds of copies of the same Pod, so creating and managing those Pods would be difficult. Fortunately, in Kubernetes, you can automate the creation and management of Pod replicas with the ReplicaSet object.

{% hint style='info' %}
NOTE

Before ReplicaSets were introduced, similar functionality was provided by the ReplicationController object type, which is now deprecated. A ReplicationController behaves exactly like a ReplicaSet, so everything that’s explained in this chapter also applies to ReplicationControllers.
{% endhint %}


Before you begin, make sure that the Pods, Services, and other objects of the Kiada suite are present in your cluster. If you followed the exercises in the previous chapter, they should already be there. If not, you can create them by creating the kiada namespace and applying all the manifests in the `Chapter13/SETUP/` directory with the following command:

```shell
$ kubectl apply -f SETUP -R
```

{% hint style='info' %}
NOTE

You can find the code files for this chapter at https://github.com/luksa/kubernetes-in-action-2nd-edition/tree/master/Chapter13.
{% endhint %}
