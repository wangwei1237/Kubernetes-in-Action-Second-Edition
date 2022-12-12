# 14.4 Summary
In this chapter, you created a Deployment for the Kiada service, now do the same for the Quote and Quiz services. If you need help, you can find the `deploy.quote.yaml` and `deploy.quiz.yaml` files in the book’s code repository.

Here’s a summary of what you learned in this chapter:
* A Deployment is an abstraction layer over ReplicaSets. In addition to all the functionality that a ReplicaSet provides, Deployments also allow you to update Pods declaratively. When you update the Pod template, the old Pods are replaced with new Pods created using the updated template.
* During an update, the Deployment controller replaces Pods based on the strategy configured in the Deployment. In the `Recreate` strategy, all Pods are replaced at once, while in the `RollingUpdate` strategy, they’re replaced gradually.
* Pods created by a ReplicaSet are owned by that ReplicaSet. The ReplicaSet is usually owned by a Deployment. If you delete the owner, the dependents are deleted by the garbage collector, but you can tell `kubectl` to orphan them instead.
* Other deployment strategies aren’t directly supported by Kubernetes, but can be implemented by appropriately configuring Deployments, Services, and the Ingress.

You also learned that Deployments are typically used to run stateless applications. In the next chapter, you’ll learn about StatefulSets, which are tailored to run stateful applications.

