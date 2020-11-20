# Accessing files on the worker node’s filesystem
Most pods shouldn’t care which host node they are running on, and they shouldn’t access any files on the node’s filesystem. System-level pods are the exception. They may need to read the node’s files or use the node’s filesystem to access the node’s devices or other components via the filesystem. Kubernetes makes this possible through the hostPath volume type. I already mentioned it in the previous section, but this is where you’ll learn when to actually use it.

## Introducing the hostPath volume
A hostPath volume points to a specific file or directory in the filesystem of the host node, as shown in the next figure. Pods running on the same node and using the same path in their hostPath volume see the same files.

Figure 7.10 A hostPath volume mounts a file or directory from the worker node’s filesystem into the container.

![](../images/7.10.png)

A hostPath volume is not a good place to store the data of a database. Because the contents of the volume are stored on the filesystem of a specific node, the database pod will not be able to see the data if it gets rescheduled to another node.

Typically, a hostPath volume is used in cases where the pod actually needs to read or write files written or read by the node itself, such as system-level logs.

The hostPath volume type is one of the most dangerous volume types in Kubernetes and is usually reserved for use in privileged pods only. If you allow unrestricted use of the hostPath volume, users of the cluster can do anything they want on the node. For example, they can use it to mount the Docker socket file (typically /var/run/docker.sock) in their container and then run the Docker client within the container to run any command on the host node as the root user. You’ll learn how to prevent this in chapter 24.

## Using a hostPath volume
To demonstrate how dangerous hostPath volumes are, let’s deploy a pod that allows you to explore the entire filesystem of the host node from within the pod. The pod manifest is shown in the following listing.

```YAML
Listing 7.11 Using a hostPath volume to gain access to the host node’s filesystem
apiVersion: v1
kind: Pod
metadata:
name: node-explorer
spec:
volumes:
- name: host-root
hostPath:
path: /
containers:
- name: node-explorer
image: alpine
command: ["sleep", "9999999999"]
volumeMounts:
- name: host-root
mountPath: /host
```

As you can see in the listing, a hostPath volume must specify the path on the host that it wants to mount. The volume in the listing will point to the root directory on the node’s filesystem, providing access to the entire filesystem of the node the pod is scheduled to.

After creating the pod from this manifest using kubectl apply, run a shell in the pod with the following command:

```shell
$ kubectl exec -it node-explorer -- sh
```

You can now navigate to the root directory of the node’s filesystem by running the following command:

```shell
/ # cd /host
```

From here, you can explore the files on the host node. Since the container and the shell command are running as root, you can modify any file on the worker node. Be careful not to break anything.

Now imagine you’re an attacker that has gained access to the Kubernetes API and are able to deploy this type of pod in a production cluster. Unfortunately, at the time of writing, Kubernetes doesn’t prevent regular users from using hostPath volumes in their pods and is therefore totally unsecure. As already mentioned, you’ll learn how to secure the cluster from this type of attack in chapter 24.

### Specifying the type for a hostPath volume
In the previous example, you only specified the path for the hostPath volume, but you can also specify the type to ensure that the path represents what that the process in the container expects (a file, a directory, or something else).

The following table explains the supported hostPath types:

| Type | Description |
| <empty> | Kubernetes performs no checks before it mounts the volume. |
| Directory | Kubernetes checks if a directory exists at the specified path. You use this type if you want to mount a pre-existing directory into the pod and want to prevent the pod from running if the directory doesn’t exist. |
| DirectoryOrCreate | Same as Directory, but if nothing exists at the specified path, an empty directory is created. |
| File | The specified path must be a file. |
| FileOrCreate | Same as File, but if nothing exists at the specified path, an empty file is created. |
| BlockDevice | The specified path must be a block device. |
| CharDevice | The specified path must be a character device. |
| Socket | The specified path must be a UNIX socket. |

Table 7.3 Supported hostPath volume types

If the specified path doesn’t match the type, the pod’s containers don’t run. The pod’s events explain why the hostPath type check failed.

{% hint style='info' %}
NOTE

When the type is FileOrCreate or DirectoryOrCreate and Kubernetes needs to create the file/directory, its file permissions are set to 644 (rw-r--r--) and 755 (rwxr-xr-x), respectively. In either case, the file/directory is owned by the user and group used to run the Kubelet.
{% endhint %}