# 8. Persisting application data with PersistentVolumes
This chapter covers

* Using PersistentVolume objects to represent persistent storage
* Claiming persistent volumes with PersistentVolumeClaim objects
* Dynamic provisioning of persistent volumes
* Using node-local persistent storage

The previous chapter taught you how to mount a network storage volume into your pods. However, the experience was not ideal because you needed to understand the environment your cluster was running in to know what type of volume to add to your pod. For example, if your cluster runs on Google’s infrastructure, you must define a `gcePersistentDisk` volume in your pod manifest. You can’t use the same manifest to run your application on Amazon because GCE Persistent Disks aren’t supported in their environment. To make the manifest compatible with Amazon, one must modify the volume definition in the manifest before deploying the pod.

You may remember from chapter 1 how Kubernetes is supposed to standardize application deployment between cloud providers, but using proprietary storage volume types in pod manifests goes against this.

Fortunately, there is a better way to add persistent storage to your pods. One where you don’t refer to a specific storage technology within the pod. This chapter explains this improved approach.