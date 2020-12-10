# 8.5 Summary
This chapter explained the details of adding persistent storage for your applications. You’ve learned that:

* Infrastructure-specific information about storage volumes doesn’t belong in pod manifests. Instead, it should be specified in the PersistentVolume object.
* A PersistentVolume object represents a portion of the disk space that is available to applications within the cluster.
* Before an application can use a PersistentVolume, the user who deploys the application must claim the PersistentVolume by creating a PersistentVolumeClaim object.
* A PersistentVolumeClaim object specifies the minimum size and other requirements that the PersistentVolume must meet.
* When using statically provisioned volumes, Kubernetes finds an existing persistent volume that meets the requirements set forth in the claim and binds it to the claim.
* When the cluster provides dynamic provisioning, a new persistent volume is created for each claim. The volume is created based on the requirements specified in the claim.
* A cluster administrator creates StorageClass objects to specify the storage classes that users can request in their claims.
* A user can change the size of the persistent volume used by their application by modifying the minimum volume size requested in the claim.
* Local persistent volumes are used when applications need to access disks that are directly attached to nodes. This affects the scheduling of the pods, since the pod must be scheduled to one of the nodes that can provide a local persistent volume. If the pod is subsequently deleted and recreated, it will always be scheduled to the same node.

In the next chapter, you’ll learn how to pass configuration data to your applications using command-line arguments, environment variables, and files. You’ll learn how to specify this data directly in the pod manifest and other Kubernetes API objects.