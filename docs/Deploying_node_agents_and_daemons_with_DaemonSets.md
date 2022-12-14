# 16 Deploying node agents and daemons with DaemonSets

This chapter covers
* Running an agent Pod on each cluster node
* Running agent Pods on a subset of nodes
* Allowing Pods to access the host node’s resources
* Assigning a priority class to a Pod
* Communicating with the local agent Pod

In the previous chapters, you learned how to use Deployments or StatefulSets to distribute multiple replicas of a workload across the nodes of your cluster. But what if you want to run exactly one replica on each node? For example, you might want each node to run an agent or daemon that provides a system service such as metrics collection or log aggregation for that node. To deploy these types of workloads in Kubernetes, you use a DaemonSet.

Before you begin, create the `kiada` Namespace, change to the `Chapter16/` directory, and apply all manifests in the `SETUP/`directory by running the following commands:

```shell
$ kubectl create ns kiada
$ kubectl config set-context --current --namespace kiada
$ kubectl apply -f SETUP -R
```

{% hint style='info' %}
NOTE

You can find the code files for this chapter at https://github.com/luksa/kubernetes-in-action-2nd-edition/tree/master/Chapter16.
{% endhint %}