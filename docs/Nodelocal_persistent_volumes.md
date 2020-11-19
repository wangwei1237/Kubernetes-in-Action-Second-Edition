# Node-local persistent volumes
In the previous sections of this chapter, you’ve used persistent volumes and claims to provide network-attached storage volumes to your pods, but this type of storage is too slow for some applications. To run a production-grade database, you should probably use an SSD connected directly to the node where the database is running.

In the previous chapter, you learned that you can use a `hostPath` volume in a pod if you want the pod to access part of the host’s filesystem. Now you’ll learn how to do the same with persistent volumes. You might wonder why I need to teach you another way to do the same thing, but it’s really not the same.

You might remember that when you add a `hostPath` volume to a pod, the data that the pod sees depends on which node the pod is scheduled to. In other words, if the pod is deleted and recreated, it might end up on another node and no longer have access to the same data.

If you use a local persistent volume instead, this problem is resolved. The Kubernetes scheduler ensures that the pod is always scheduled on the node to which the local volume is attached.

{% hint style='info' %}
NOTE

Local persistent volumes are also better than `hostPath` volumes because they offer much better security. As explained in the previous chapter, you don’t want to allow regular users to use `hostPath` volumes at all. Because persistent volumes are managed by the cluster administrator, regular users can’t use them to access arbitrary paths on the host node.
{% endhint %}

## Creating local persistent volumes
Imagine you are a cluster administrator and you have just connected a fast SSD directly to one of the worker nodes. Because this is a new class of storage in the cluster, it makes sense to create a new StorageClass object that represents it.

### Creating a storage class to represent local storage
Create a new storage class manifest as shown in the following listing.

```YAML
Listing 8.13 Defining the local storage class
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: local                                 #A
provisioner: kubernetes.io/no-provisioner     #B
volumeBindingMode: WaitForFirstConsumer       #C
```

\#A Let’s call this storage class local

\#B Persistent volumes of this class are provisioned manually

\#C The persistent volume claim should be bound only when the first pod that uses the claim is deployed.

As I write this, locally attached persistent volumes need to be provisioned manually, so you need to set the provisioner as shown in the listing. Because this storage class represents locally attached volumes that can only be accessed within the nodes to which they are physically connected, the `volumeBindingMode` is set to `WaitForFirstConsumer`, so the binding of the claim is delayed until the pod is scheduled.

### Attaching a disk to a cluster node
I assume that you’re using a Kubernetes cluster created with the kind tool to run this exercise. Let’s emulate the installation of the SSD in the node called `kind-worker`. Run the following command to create an empty directory at the location `/mnt/ssd1` in the node’s filesystem:

```shell
$ docker exec kind-worker mkdir /mnt/ssd1
```

### Creating a PersistentVolume object for the new disk
After attaching the disk to one of the nodes, you must tell Kubernetes that this node now provides a local persistent volume by creating a PersistentVolume object. The manifest for the persistent volume is shown in the following listing.

```YAML
Listing 8.14 Defining a local persistent volume
kind: PersistentVolume
apiVersion: v1
metadata:
  name: local-ssd-on-kind-worker              #A
spec:
  accessModes:
  - ReadWriteOnce
  storageClassName: local                     #B
  capacity:
    storage: 10Gi
  local:                                      #C
    path: /mnt/ssd1                           #C
  nodeAffinity:                               #D
    required:                                 #D
      nodeSelectorTerms:                      #D
      - matchExpressions:                     #D
        - key: kubernetes.io/hostname         #D
          operator: In                        #D
          values:                             #D
          - kind-worker                       #D
```

\#A This persistent volume represents the local SSD installed in the kind-worker node, hence the name.

\#B This volume belongs to the local storage class.

\#C This volume is mounted in the node’s filesystem at the specified path.

\#D This section tells Kubernetes which nodes can access this volume. Since the SSD is attached only to the node kind-worker, it is only accessible on this node.

Because this persistent volume represents a local disk attached to the `kind-worker` node, you give it a name that conveys this information. It refers to the `local` storage class that you created previously. Unlike previous persistent volumes, this volume represents storage space that is directly attached to the node. You therefore specify that it is a `local` volume. Within the `local` volume configuration, you also specify the path where the SSD is mounted (`/mnt/ssd1`).

At the bottom of the manifest, you’ll find several lines that indicate the volume’s node affinity. A volume’s node affinity defines which nodes can access this volume.

{% hint style='info' %}
NOTE

You’ll learn more about node affinity and selectors in later chapters. Although it looks complicated, the node affinity definition in the listing simply defines that the volume is accessible from nodes whose `hostname` is `kind-worker`. This is obviously exactly one node.
{% endhint %}

Okay, as a cluster administrator, you’ve now done everything you needed to do to enable cluster users to deploy applications that use locally attached persistent volumes. Now it’s time to put your application developer hat back on again.

## Claiming and using local persistent volumes
As an application developer, you can now deploy your pod and its associated persistent volume claim.

### Creating the pod
The pod definition is shown in the following listing.

```YAML
Listing 8.15 Pod using a locally attached persistent volume
apiVersion: v1
kind: Pod
metadata:
  name: mongodb-local
spec:
  volumes:
  - name: mongodb-data
    persistentVolumeClaim:
      claimName: mongodb-pvc-local            #A
  containers:
  - image: mongo
    name: mongodb
    volumeMounts:
    - name: mongodb-data
      mountPath: /data/db
```

\#A The pod uses the mongodb-pvc-local claim

There should be no surprises in the pod manifest. You already know all this.

### Creating the persistent volume claim for a local volume
As with the pod, creating the claim for a local persistent volume is no different than creating any other persistent volume claim. The manifest is shown in the next listing.

```YAML
Listing 8.16 Persistent volume claim using the local storage class
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mongodb-pvc-local
spec:
  storageClassName: local                  #A
  resources:
    requests:
      storage: 1Gi
  accessModes:
    - ReadWriteOnce
```

\#A The claim requests a persistent volume from the local storage class

No surprises here either. Now on to creating these two objects.

### Creating the pod and the claim
After you write the pod and claim manifests, you can create the two objects by applying the manifests in any order you want. If you create the pod first, since the pod requires the claim to exist, it simply remains in the `Pending` state until you create the claim.

After both the pod and the claim are created, the following events take place:

1. The claim is bound to the persistent volume.
2. The scheduler determines that the volume bound to the claim that is used in the pod can only be accessed from the kind-worker node, so it schedules the pod to this node.
3. The pod’s container is started on this node, and the volume is mounted in it.

You can now use the MongoDB shell again to add documents to it. Then check the `/mnt/ssd1` directory on the kind-worker node to see if the files are stored there.

### Recreating the pod
If you delete and recreate the pod, you’ll see that it’s always scheduled on the kind-worker node. The same happens if multiple nodes can provide a local persistent volume when you deploy the pod for the first time. At this point, the scheduler selects one of them to run your MongoDB pod. When the pod runs, the claim is bound to the persistent volume on that particular node. If you then delete and recreate the pod, it is always scheduled on the same node, since that is where the volume that is bound to the claim referenced in the pod is located.