# 7.5 Summary
This chapter has explained the basics of adding volumes to pods, but this was only the beginning. You’ll learn more about this topic in the next chapter. So far you’ve learned this:

* Pods consist of containers and volumes. Each volume can be mounted at the desired location in the container’s filesystem.
* Volumes are used to persist data across container restarts, share data between the containers in the pod, and even share data between the pods.
* Many volume types exist. Some are generic and can be used in any cluster regardless of the cluster environment, while others, such as the gcePersistentDisk, can only be used if the cluster runs on a particular cloud provider infrastructure.
* An emptyDir volume is used to store data for the duration of the pod. It starts as an empty directory just before the pod’s containers are started and is deleted when the pod terminates.
* The gitRepo volume is a deprecated volume type that is initialized by cloning a Git repository. Alternatively, an emptyDir volume can be used in combination with an init container that initializes the volume from Git or any other source.
* Network volumes are typically mounted by the host node and then exposed to the pod(s) on that node.
* Depending on the underlying storage technology, you may or may not be able to mount a network storage volume in read/write mode on multiple nodes simultaneously.
* By using a proprietary volume type in a pod manifest, the pod manifest is tied to a specific Kubernetes cluster. The manifest must be modified before it can be used in another cluster. Chapter 8 explains how to avoid this issue.
* The hostPath volume allows a pod to access any path in filesystem of the worker node. This volume type is dangerous because it allows users to make changes to the configuration of the worker node and run any process they want on the node.

In the next chapter, you’ll learn how to abstract the underlying storage technology away from the pod manifest and make the manifest portable to any other Kubernetes cluster.