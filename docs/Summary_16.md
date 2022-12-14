# 16.4  Summary
In this chapter, you learned how to run daemons and node agents. You learned that:

* A DaemonSet object represents a set of daemon Pods distributed across the cluster Nodes so that exactly one daemon Pod instance runs on each node.
* A DaemonSet is used to deploy daemons and agents that provide system-level services such as log collection, process monitoring, node configuration, and other services required by each cluster Node.
* When you add a node selector to a DaemonSet, the daemon Pods are deployed only on a subset of all cluster Nodes.
* A DaemonSet doesn't deploy Pods to control plane Nodes unless you configure the Pod to tolerate the Nodes' taints.
* The DaemonSet controller ensures that a new daemon Pod is created when a new Node is added to the cluster, and that it’s removed when a Node is removed.
* Daemon Pods are updated according to the update strategy specified in the DaemonSet. The RollingUpdate strategy updates Pods automatically and in a rolling fashion, whereas the OnDelete strategy requires you to manually delete each Pod for it to be updated.
* If Pods deployed through a DaemonSet require extended access to the Node's resources, such as the file system, network environment, or privileged system calls, you configure this in the Pod template in the DaemonSet.
* Daemon Pods should generally have a higher priority than Pods deployed via Deployments. This is achieved by setting a higher PriorityClass for the Pod.
* Client Pods can communicate with local daemon Pods through a Service with `internalTrafficPolicy` set to `Local`, or through the Node's IP address if the daemon Pod is configured to use the node's network environment (`hostNetwork`) or a host port is forwarded to the Pod (`hostPort`).
In the next chapter, you’ll learn how to run batch workloads with the Job and CronJob object types.