# 13.5 Summary
In this chapter, you learned that:

* A ReplicaSet represents a group of identical Pods that you manage as a unit. In the ReplicaSet, you specify a Pod template, the desired number of replicas, and a label selector.
* Almost all Kubernetes API object types have an associated controller that processes objects of that type. In each controller, a reconciliation control loop runs that constantly reconciles the actual state with the desired state.
* The ReplicaSet controller ensures that the actual number of Pods always matches the desired number specified in the ReplicaSet. When these two numbers diverge, the controller immediately reconciles them by creating or deleting Pod objects.
* You can change the number of replicas you want at any time and the controller will take the necessary steps to honor your request. However, when you update the Pod template, the controller won’t update the existing Pods.
* Pods created by a ReplicaSet are owned by that ReplicaSet. If you delete the owner, the dependents are deleted by the garbage collector, but you can tell kubectl to orphan them instead.

In the next chapter, you’ll replace the ReplicaSet with a Deployment object.