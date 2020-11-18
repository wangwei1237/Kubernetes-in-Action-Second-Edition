# Dynamic provisioning of persistent volumes
So far in this chapter you’ve seen how developers can claim pre-provisioned persistent volumes as a place for their pods to store data persistently without having to deal with the details of the underlying storage technology. However, a cluster administrator must pre-provision the physical volumes and create a PersistentVolume object for each of these volumes. Then each time the volume is bound and released, the administrator must manually delete the data on the volume and recreate the object.

To keep the cluster running smoothly, the administrator may need to pre-provision dozens, if not hundreds, of persistent volumes, and constantly keep track of the number of available volumes to ensure the cluster never runs out. All this manual work contradicts the basic idea of Kubernetes, which is to automate the management of large clusters. As one might expect, a better way to manage volumes exists. It’s called *dynamic provisioning of persistent volumes*.

With dynamic provisioning, instead of provisioning persistent volumes in advance (and manually), the cluster admin deploys a persistent volume provisioner to automate the just-in-time provisioning process, as shown in the following figure.

Figure 8.8 Dynamic provisioning of persistent volumes

![](../images/8.8.png)

In contrast to static provisioning, the order in which the claim and the volume arise is reversed. When a user creates a persistent volume claim, the dynamic provisioner provisions the underlying storage and creates the PersistentVolume object for that particular claim. The two objects are then bound.

If your Kubernetes cluster is managed by a cloud provider, it probably already has a persistent volume provisioner configured. If you are running Kubernetes on-premises, you’ll need to deploy a custom provisioner, but this is outside the scope of this chapter. Clusters that are provisioned with Minikube or kind usually also come with a provisioner out of the box.

## Introducing the StorageClass object
The persistent volume claim definition you created in the previous section specifies the minimum size and the required access modes of the volume, but it also contains a field named `storageClassName`, which wasn’t discussed yet.

A Kubernetes cluster can run multiple persistent volume provisioners, and a single provisioner may support several different types of storage volumes. When creating a claim, you use the `storageClassName` field to specify which storage class you want.

### Listing storage classes
The storage classes available in the cluster are represented by StorageClass API objects. You can list them with the `kubectl get` command:

```shell
$ kubectl get sc
NAME                 PROVISIONER             RECLAIMPOLICY   ...
standard (default)   rancher.io/local-path   Delete          ...
```

{% hint style='info' %}
NOTE 

The shorthand for `storageclass` is `sc`.
{% endhint %}

In many clusters, as in the example above, only one storage class called `standard` is configured. It’s also marked as the default, which means that this is the class that is used to provision the persistent volume when you omit the `storageClassName` field in your persistent volume claim definition.

{% hint style='info' %}
NOTE

Remember that omitting the `storageClassName` field causes the default storage class to be used, whereas explicitly setting the field to `""` disables dynamic provisioning and causes an existing volume to be selected and bound to the claim.
{% endhint %}

### Inspecting the default storage class
Let’s get to know the StorageClass object kind by looking at the YAML definition of the `standard` storage class using the `kubectl` `get` command. The output of the command is shown in the following listing.

```YAML
Listing 8.7 The definition of the standard storage class in a kind-provisioned cluster
$ kubectl get sc standard -o yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  annotations:
    storageclass.kubernetes.io/is-default-class: "true"  #A
  name: standard                                         #B
  ...
provisioner: rancher.io/local-path                       #C
reclaimPolicy: Delete                                    #D
volumeBindingMode: WaitForFirstConsumer                  #E
```

\#A This marks the storage class as default.

\#B The name of this storage class

\#C The name of the provisioner that gets called to provision persistent volumes of this class.

\#D The reclaim policy for persistent volumes of this class.

\#E How volumes of this class are provisioned and bound.

{% hint style='info' %}
NOTE

You’ll notice that StorageClass objects have no `spec` or `status` sections. This is because the object only contains static information. Since the object’s fields aren’t organized in the two sections, the YAML manifest may be more difficult to read. This is also confounded by the fact that fields in YAML are typically sorted in alphabetical order, which means that some fields may appear above the `apiVersion`, `kind` or `metadata` fields. Be careful not to overlook these.
{% endhint %}

If you look closely at the top of the listing, you’ll see that the storage class definition includes an annotation, which marks this particular storage class as the default.

{% hint style='info' %}
NOTE

You’ll learn what an object annotation is in chapter 10.
{% endhint %}

As specified in the storage class definition, each time you create a persistent volume claim that references this class, the provisioner `rancher.io/local-path` is called to provision the persistent volume.

The provisioner sets the volume’s reclaim policy to the one set in the storage class, which in the above example is `Delete`. As you have already learned, this means that the volume is deleted when you release it by deleting the claim.

The last field in the storage class definition is `volumeBindingMode`. It determines whether the persistent volume is created immediately or only when the first consumer appears. You’ll find out what this means later.

`StorageClass` objects also support some other fields that are not shown in the above listing. You can use `kubectl explain` to see what these are. You’ll learn about some of them in the following sections.

In summary, a StorageClass object represents a class of storage that can be dynamically provisioned. As shown in the following figure, each storage class specifies what provisioner to use and the parameters that should be passed to it when provisioning the volume. The user decides which storage class to use for each of their persistent volume claims.

Figure 8.9 The relationship between storage classes, persistent volume claims and dynamic volume provisioners

![](../images/8.9.png)

## Dynamic provisioning using the default storage class
You’ve previously used a statically provisioned persistent volume for your MongoDB pod. Now you’ll use dynamic provisioning to achieve the same result, but with much less manual work.

### Creating a claim with dynamic provisioning
To dynamically provision a persistent volume using the storage class from the previous section, you can create a PersistentVolumeClaim object with the `storageClassName` field set to `standard` or with the field omitted altogether.

Let’s use the latter approach, as this makes the manifest as minimal as possible. You can find the manifest in the mongodb-pvc-dynamic-default.yaml file in the book’s GitHub repository. It’s also shown in the following listing.

```YAML
Listing 8.8 A minimal PVC definition using the default storage class
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mongodb-pvc-default
spec:                      #A
  resources:               #A
    requests:              #A
      storage: 1Gi         #A
  accessModes:             #A
    - ReadWriteOnce        #A
```

\#A Only the minimum size and access modes are specified. The storageClassName field is not set.

This PersistentVolumeClaim manifest contains only the storage size request and the desired access mode, but no `storageClassName` field, so the default storage class is used.

After you have created the claim with `kubectl` `apply`, you can see which storage class it’s using by inspecting the claim with `kubectl get`:

```shell
$ kubectl get pvc mongodb-pvc-default
NAME                 STATUS   VOLUME  CAPACITY  ACCESS MODES   STORAGECLASS
mongodb-pvc-default  Pending                                   standard
```

As expected and as indicated in the `STORAGECLASS` column, the claim you just created uses the `standard` storage class.

### Understanding why a claim’s status is Pending
Depending on where you run this example, you may find that the persistent volume claim you just created is `Pending`. In one of the previous sections, you learned that this happens when no persistent volume matches the claim, either because it doesn’t exist or because it’s not available for binding.

However, you are now using dynamic provisioning, where the volume is to be created after you create the claim, and specifically for this claim. Is your claim pending because it takes more than a few seconds to provision the volume?

No, the reason for the pending status lies elsewhere. Your claim will remain in the `Pending` state until you create a pod that uses this claim. I’ll explain why later. For now, let’s just create the pod.

### Using the persistent volume claim in a pod
Create a new pod manifest file from the `mongodb` pod manifest that you used earlier. Change the name of the pod to `mongodb-default` and the value of the `claimName` field to make the pod use the `mongodb-pvc-default` claim. You can find the resulting manifest in the mongodb-pod-pvc-default.yaml file in the book’s GitHub repository. Use it to create the pod.

Moments after you create the pod, the status of the persistent volume claim should change to `Bound`, as shown here:

```shell
$ kubectl get pvc mongodb-pvc-default
NAME                  STATUS   VOLUME             CAPACITY   ACCESS   ...
mongodb-pvc-default   Bound    pvc-c71fb2c2-...   1Gi        RWO      ...
```

This implies that the persistent volume has been created. List persistent volumes to confirm (the following output has been reformatted to make it easier to read):

```shell
$ kubectl get pv
NAME              CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   ...
pvc-c71fb2c2...   1Gi        RWO            Delete           Bound    ...
 
...   STATUS   CLAIM                         STORAGECLASS   REASON   AGE
...   Bound    default/mongodb-pvc-default   standard                3s
```

As you can see, because the volume was created on demand, its properties perfectly match the requirements specified in the claim and the storage class it references.

### Understanding when a dynamically provisioned volume is actually provisioned
Why was the volume created and bound to the claim only after you had deployed the pod? In an earlier example, the pre-provisioned volume was bound to the claim as soon as you created the claim. Is this a difference between static and dynamic provisioning? The answer is no.

The system behaves this way because of how the storage class you’re using is configured. You may remember that its YAML definition contained a field called `volumeBindingMode` that was set to `WaitForFirstConsumer`. This mode causes the system to wait until the first pod (the consumer of the claim) exists before binding the claim. The volume is only provisioned then and not earlier.

Some types of volumes require such behaviour, as the system needs to know where the pod is scheduled before it can provision the volume. This is the case with provisioners that create node-local volumes, such as the one you find in clusters created with the kind tool (you may remember that the provisioner referenced in the storage class was `rancher.io/local-path`).

{% hint style='info' %}
NOTE

Refer to the documentation of your chosen provisioner to determine whether it requires the volume binding mode to be set to `WaitForFirstConsumer`.
{% endhint %}

The alternative to `WaitForFirstConsumer` is the `Immediate` volume binding mode. The two modes are explained in the following table.

| Volume binding mode | Description |
| Immediate | The provision and binding of the persistent volume takes place immediately after the claim is created. Because the consumer of the claim is unknown at this point, this mode is only applicable to volumes that are can be accessed from any cluster node. |
| WaitForFirstConsumer | The volume is provisioned and bound to the claim when the first pod that uses this claim is created. This mode is used for topology-constrained volume types.|

Table 8.4 Supported volume binding modes

## Creating storage classes
As you saw in the previous sections, a Kubernetes cluster deployed with the kind tool contains a single storage class. As I write this, the same is true for GKE clusters, but GKE uses a different provisioner that creates GCE Persistent Disks instead of local volumes.

### Inspecting the standard storage class in GKE
The following listing shows the definition of the `standard` storage class in GKE. I’ve rearranged the fields since the default alphabetical ordering makes it difficult to understand.

```YAML
Listing 8.9 The standard storage class in GKE
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: standard
  annotations:
    storageclass.kubernetes.io/is-default-class: "true"
    ...
provisioner: kubernetes.io/gce-pd       #A
parameters:                             #B
  type: pd-standard                     #B
volumeBindingMode: Immediate
allowVolumeExpansion: true
reclaimPolicy: Delete
```

\#A The provisioner used to provision volumes of this storage class

\#B This type parameter is passed to the provisioner

As you can see, the annotation marks this storage class as default. If you create a persistent volume claim in GKE that references either this class or none at all, the provisioner `kubernetes.io/gce-pd` is called to create the volume. The parameter `type: pd-standard` is passed to the provisioner when the volume is to be provisioned. This tells the provisioner what type of GCE Persistent Disk to create.

{% hint style='info' %}
NOTE

Google Compute Engine supports several other disk types. Their availability depends on where you operate your cluster. To view the list of types for each availability zone, run `gcloud compute disk-types list`.
{% endhint %}

### Using an SSD persistent disk in GKE
One of the disk types supported in most GCE zones is the `pd-ssd` type, which provisions a network-attached SSD. Let’s create a storage class called `fast` and configure it so that the provisioner creates a disk of type `pd-ssd` when you request this storage class in your claim.

```YAML
Listing 8.10 A custom storage class definition: storageclass-fast-gcepd.yaml
apiVersion: storage.k8s.io/v1           #A
kind: StorageClass                      #A
metadata:
  name: fast                            #B
provisioner: kubernetes.io/gce-pd       #C
parameters:
  type: pd-ssd                          #D
```

\#A This manifest defines a StorageClass object

\#B The name of this storage class

\#C The provisioner to use

\#D Tells the provisioner to provision an SSD disk

{% hint style='info' %}
NOTE 

If you’re using another cloud provider, check their documentation to find the name of the provisioner and the parameters you need to pass in. If you’re using Minikube or kind, and you’d like to run this example, set the provisioner and parameters to the same values as in the default storage class. For this exercise, it doesn’t matter if the provisioned volume doesn’t actually use an SSD.
{% endhint %}

Create the StorageClass object by applying this manifest to your cluster and list the available storage classes to confirm that more than one is now available.

## Requesting the storage class in a persistent volume claim