## 15.5 Summary
In this chapter, you learned how to run stateful applications in Kubernetes. You learned that:

* Stateful workloads are harder to manage than their stateless counterparts because managing state is difficult. However, with StatefulSets, managing stateful workloads becomes much easier because the StatefulSet controller automates most of the work.
* With StatefulSets you can manage a group of Pods as pets, whereas Deployments treat the Pods like cattle. The Pods in a StatefulSet use ordinal numbers instead of having random names.
* A StatefulSet ensures that each replica gets its own stable identity and its own PersistentVolumeClaim(s). These claims are always associated with the same Pods.
* In combination with a StatefulSet, a headless Service ensures that each Pod receives a DNS record that always resolves to the Pod’s IP address, even if the Pod is moved to another node and receives a new IP address.
* StatefulSet Pods are created in the order of ascending ordinal numbers, and deleted in reverse order.
The Pod management policy configured in the StatefulSet determines whether Pods are created and deleted sequentially or simultaneously.
* The PersistentVolumeClaim retention policy determines whether claims are deleted or retained when you scale down or delete a StatefulSet.
* When you update the Pod template in a StatefulSet, the controller updates the underlying Pods. This happens on a rolling basis, from highest to lowest ordinal number. Alternatively, you can use a semi-automatic update strategy, where you delete a Pod and the controller then replaces it.
* Since StatefulSets don’t provide everything needed to fully manage a stateful workload, these types of workloads are typically managed via custom API object types and Kubernetes Operators. You create an instance of the custom object, and the Operator then creates the StatefulSet and supporting objects.


In this chapter, you also created the quiz-data-importer Pod, which, unlike all the other Pods you’ve created so far, performs a single task and then exits. In the next chapter, you’ll learn how to run these types of workloads using the Job and CronJob object types. You'll also learn how to use a DaemonSet to run a system Pod on each node.