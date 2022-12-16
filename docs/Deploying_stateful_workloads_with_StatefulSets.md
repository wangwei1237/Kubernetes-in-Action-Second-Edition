# 15 Deploying stateful workloads with StatefulSets

This chapter covers
* Managing stateful workloads via StatefulSet objects
Exposing individual Pods via headless Services
* Understanding the difference between Deployments and StatefulSets
* Automating stateful workload management with Kubernetes Operators

Each of the three services in your Kiada suite is now deployed via a Deployment object. The Kiada and Quote services each have three replicas, while the Quiz service has only one because its data doesn’t allow it to scale easily. In this chapter, you’ll learn how to properly deploy and scale stateful workloads like the Quiz service with a `StatefulSet`.

Before you begin, create the `kiada` Namespace, change to the `Chapter15/` directory and apply all manifests in the `SETUP/` directory with the following command:

```shell
$ kubectl apply -n kiada -f SETUP -R
```
{% hint style='info' %}
IMPORTANT

The examples in this chapter assume that the objects are created in the `kiada` Namespace. If you create them in a different location, you must update the DNS domain names in several places.
{% endhint %}

{% hint style='info' %}
NOTE

You can find the code files for this chapter at https://github.com/luksa/kubernetes-in-action-2nd-edition/tree/master/Chapter15.
{% endhint %}
