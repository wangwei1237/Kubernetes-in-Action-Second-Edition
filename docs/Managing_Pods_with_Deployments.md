# 14 Managing Pods with Deployments
 
This chapter covers
* Deploying stateless workloads with the Deployment object
* Horizontally scaling Deployments
* Updating workloads declaratively
* Preventing rollouts of faulty workloads
* Implementing various deployment strategies

In the previous chapter, you learned how to deploy Pods via ReplicaSets. However, workloads are rarely deployed this way because ReplicaSets don’t provide the functionality necessary to easily update these Pods. This functionality is provided by the Deployment object type. By the end of this chapter, each of the three services in the Kiada suite will have its own Deployment object.

Before you begin, make sure that the Pods, Services, and other objects of the Kiada suite are present in your cluster. If you followed the exercises in the previous chapter, they should already be there. If not, you can create them by creating the `kiada` namespace and applying all the manifests in the `Chapter14/SETUP/` directory with the following command:

```shell
$ kubectl apply -f SETUP -R
```

{% hint style='info' %}
NOTE

You can find the code files for this chapter at https://github.com/luksa/kubernetes-in-action-2nd-edition/tree/master/Chapter14.
{% endhint %}