# Using volumes
The simplest volume type is emptyDir. This is the volume type you normally use when you need to persist files across container restarts, as explained in section 7.1.1.

## Using an emptyDir volume to persist files across container restarts
Remember the fortune pod from the previous chapter? It uses a post-start hook to write a fortune quote to a file that is then served by the Nginx web server. This file is stored in the container’s filesystem, which means that it’s lost whenever the container is restarted.

You can confirm this by deploying the pod in the fortune-no-volume.yaml file, retrieving the quote, and then restarting the container by telling Nginx to stop. The following listing shows the commands you need to run.

```shell
Listing 7.1 Testing the behavior of the fortune-no-volume pod
$ kubectl apply -f fortune-no-volume.yaml                                  #A
pod/fortune-no-volume created                                              #A
 
$ kubectl exec fortune-no-volume -- cat /usr/share/nginx/html/quote        #B
Quick!! Act as if nothing has happened!                                    #B
 
$ kubectl exec fortune-no-volume -- nginx -s stop                          #C
[notice] 71#71: signal process started                                     #C
 
$ kubectl exec fortune-no-volume -- cat /usr/share/nginx/html/quote        #D
Hindsight is an exact science.                                             #D
```

When you retrieve the quote after restarting the container, you’ll see that it has changed. If you want the pod to serve the same quote no matter how often its container is restarted, you must ensure the file in stored in a volume. An emptyDir volume is perfect for this.

### Introducing the emptyDir volume type
The emptyDir volume is the simplest type of volume. As the name suggests, it starts as an empty directory. When this type of volume is mounted in a container, files written by the application to the path where the volume is mounted are preserved for the duration of the pod’s existence.

This volume type is used in single-container pods when data must be preserved even if the container is restarted. It’s also used when the container’s filesystem is marked read-only and you want to make only part of it writable. In pods with two or more containers, an emptyDir volume is used to exchange data between them.

### Adding an emptyDir volume to the fortune pod
Let’s change the definition of the fortune pod so that the post-start hook writes the file to the volume instead of to the ephemeral filesystem of the container. Two changes to the pod manifest are required to achieve this:

1. An emptyDir volume must be added to the pod.
2. The volume must be mounted into the container.

However, since the post-start hook is executed every time the container is started, you also need to change the command that the post-start hook executes if you want to prevent it from overwriting the file if it already exists. The operation of the new command is explained in the sidebar.

{% hint style='info' %}
UNDERSTANDING THE NEW COMMAND IN THE FORTUNE POST-START HOOK

The post-start hook executes the following shell command:

```shell
ls /usr/share/nginx/html/quote || (apk add fortune &&
[CA] fortune > /usr/share/nginx/html/quote)
```

If you’re not familiar with the Linux shell, it may not be obvious what this command does. Let me explain.

First, the ls command, which is typically used to list directory contents, is used in an untypical way to check whether the quote file exists. What you may not know is that the ls command succeeds (returns exit code zero) if the specified file exists, and fails (returns a non-zero exit code) if it doesn’t.

Since in our case the entire command is an or expression, as denoted by the double pipe character, the right side of the expression is never evaluated if the left side of the expression evaluates to true (which is when the ls command succeeds). When this happens, the apk and fortune commands are never executed. Therefore, if the quote file already exists, it’s not overwritten.

If the file doesn’t exist, the ls command fails, causing the left side of the expression to be false, which means that the right side must be evaluated to resolve the expression. The apk and fortune commands are therefore executed and they generate the quote file.

This new command ensures that the quote file is only written once - at the first start of the container, not at subsequent restarts
{% endhint %}

The following listing shows the new pod manifest that incorporates all the changes.

```YAML
Listing 7.2 A pod with an emptyDir volume: fortune-emptydir.yaml
kind: Pod
metadata:
name: fortune-emptydir
spec:
volumes:                                                    #A
- name: content                                             #A
emptyDir: {}                                                #A
containers:
- name: nginx
image: nginx:alpine
volumeMounts:                                               #B
- name: content                                             #B
mountPath: /usr/share/nginx/html                            #B
lifecycle:
postStart:
exec:
command:
- sh
- -c
- "ls /usr/share/nginx/html/quote || (apk add fortune &&" \ #C
"[CA] fortune > /usr/share/nginx/html/quote)"               #C
ports:
- name: http
containerPort: 80
```

The listing shows that an emptyDir volume named content is defined in the spec.volumes array of the pod manifest.

### Configuring the volume
In general, each volume definition must include a name and a type, which is indicated by the name of the nested field (for example: emptyDir, gcePersistentDisk, nfs, and so on). This field is typically an object with additional sub-fields that allow you to configure the volume and are specific to each volume type.

For example, the emptyDir volume type supports two fields for configuring the volume. They are explained in the following table.

| Field | Description |
| --- | --- | 
| medium | The type of storage medium to use for the directory. If left empty, the default medium of the host node is used (the directory is created on one of the node’s disks). The only other supported option is Memory, which causes the volume to use tmpfs, a virtual memory filesystem where the files are kept in memory instead of on the hard disk. |
| sizeLimit | The total amount of local storage required for the directory, whether on disk or in memory. For example, to set the maximum size to ten mebibytes, you set this field to 10Mi. |

Table 7.1 Configuration options for an emptyDir volume

{% hint style='info' %}
NOTE

The emptyDir field in the volume definition defines neither of these properties. The curly braces {} have been added to indicate this explicitly, but they can be omitted.
{% endhint %}

### Mounting the volume in the container
Defining a volume in the pod is only half of what you need to do to make it available in a container. The volume must also be mounted in the container. This is done by referencing the volume by name in the volumeMounts array within the container definition.

In addition to the name, a volume mount definition must also include the mountPath - the path within the container where the volume should be mounted. In the example, the volume is mounted at /usr/share/nginx/html because that’s where the post-start hook writes the quote file.

Since the quote file is now written to the volume, which exists for the duration of the pod, the pod should always serve the same quote, even when the container is restarted.

### Observing the emptyDir volume in action
If you deploy the pod in the listing, you’ll notice that the quote will remain the same throughout the life of the pod, regardless of how often the container is restarted, because the /usr/share/nginx/html directory where the file is stored is mounted from somewhere else. You can tell this by listing the mount points in the container. Run the mount command to see that the directory is mounted into the container from elsewhere:

```shell
$ kubectl exec fortune-emptydir -- mount --list | grep nginx/html
/dev/mapper/fedora_t580-home on /usr/share/nginx/html type ext4 (rw)
```

## Using an emptyDir volume to share files between containers
An emptyDir volume is also useful for sharing files between containers of the same pod (it can’t be used to share files between containers of different pods). Let’s see how this is done.

The fortune pod currently serves the same quote throughout the lifetime of the pod. This isn’t so interesting. Let’s build a new version of the pod, where the quote changes every 30 seconds.

You’ll retain Nginx as the web server, but will replace the post-start hook with a container that periodically runs the fortune command to update the quote stored in the file. The container is available at docker.io/luksa/fortune:1.0, but you can also build it yourself by following the instructions in the sidebar.

{% hint style='info' %}
BUILDING THE FORTUNE CONTAINER IMAGE

You need two files to build the image. First is the docker_entrypoint.sh script that has the following contents:

```shell
#!/bin/sh
trap "exit" INT
 
INTERVAL=${INTERVAL:-30}
OUTPUT_FILE=${1:-/var/local/output/quote}
 
echo "Fortune Writer 1.0"
echo "Configured to write fortune to $OUTPUT_FILE every $INTERVAL seconds"
 
while :
do
echo "$(date) Writing fortune..."
fortune > $OUTPUT_FILE
sleep $INTERVAL
done
```

This is the script that is executed when you run this container. To build the container image, you’ll also need a Dockerfile with the following contents:

```shell
FROM alpine
RUN apk add fortune
COPY docker_entrypoint.sh /docker_entrypoint.sh
VOLUME /var/local/output
ENTRYPOINT ["/docker_entrypoint.sh"]
```

You can build the image and push it to Docker Hub by running the following two commands (replace luksa with your own Docker Hub user ID):

```shell
$ docker build -t luksa/fortune-writer:1.0 .
$ docker push luksa/fortune-writer:1.0
```

If you’ve downloaded the code from the book’s code repository at GitHub, you’ll find both files in the Chapter07/fortune-writer-image directory. To build and push the image you can also run the following command in the Chapter07/ directory:

```shell
$ PREFIX=luksa/ PUSH=true ./build-fortune-writer-image.sh
```

Again, replace luksa/ with your Docker Hub user ID.
{% endhint %}

### Creating the pod
Now that you have both images required to run the pod, create the pod manifest. Its contents are shown in the following listing.

```YAML
Listing 7.3 Two containers sharing the same volume: fortune.yaml
apiVersion: v1
kind: Pod
metadata:
name: fortune
spec:
volumes:
- name: content
emptyDir: {}
containers:
- name: fortune
image: luksa/fortune-writer:1.0
volumeMounts:
- name: content
mountPath: /var/local/output #C
- name: nginx
image: nginx:alpine
volumeMounts:
- name: content
mountPath: /usr/share/nginx/html
readOnly: true
ports:
- name: http
containerPort: 80
```

The pod consists of two containers and a single volume, which is mounted in both containers, but at different locations within each container. The reason for this is that the fortune container writes the quote file into the /var/local/output directory, whereas the nginx container serves files from the /usr/share/nginx/html directory.

### Running the pod
When you create the pod from the manifest, the two containers start and continue running until the pod is deleted. The fortune container writes a new quote to the file every 30 seconds, and the nginx container serves this file. After you create the pod, use the kubectl port-forward command to open a communication tunnel to the pod and test if the server responds with a different quote every 30 seconds.

Alternatively, you can also display the contents of the file using either of the following two commands:

```shell
$ kubectl exec fortune -c fortune -- cat /var/local/output/quote
$ kubectl exec fortune -c nginx -- cat /usr/share/nginx/html/quote
```

As you can see, one of them prints the contents of the file from within the fortune container, whereas the other command prints the contents from within the nginx container. Because the two paths point to the same quote file on the shared volume, the output of the commands is identical.

### Specifying the storage medium to use in the emptyDir volume
The emptyDir volume in the previous example created a directory on the actual drive of the worker node hosting your pod, so its performance depends on the type of drive installed on the node. If you need to perform the I/O operations on the volume as quickly as possible, you can instruct Kubernetes to create the volume using the tmpfs filesystem, which keeps its files in memory. To do this, set the emptyDir’s medium field to Memory as in the following snippet:

```shell
volumes:
  - name: content
    emptyDir:
medium: Memory
```

Creating the emptyDir volume in memory is also a good idea whenever it’s used to store sensitive data. Because the data is not written to disk, there is less chance that the data will be compromised and persisted longer than desired. As you’ll learn in chapter 9, Kubernetes uses the same in-memory approach when data stored in the Secret API object type needs to be exposed to the application in the container.

### Specifying the size limit for the emptyDir volume
The size of an emptyDir volume can be limited by setting the sizeLimit field. Setting this field is especially important for in-memory volumes when the overall memory usage of the pod is limited by so-called resource limits. You’ll learn about this in chapter 20.

## Specifying how a volume is to be mounted in the container
In the previous section, the focus was mainly on the volume type. The volume was mounted in the container with the minimum configuration required, as shown in the following listing.

```shell
Listing 7.4 The minimal configuration of a volumeMount
containers:
- name: my-container
...
volumeMounts:
- name: my-volume
mountPath: /path/in/container
```

As you can see in the listing, the absolute minimum is to specify the name of the volume and the path where it should be mounted. But other configuration options for mounting volumes exist. The full list of volume mount options is shown in the following table.

| Field | Description |
| --- | --- |
| name | The name of the volume to mount. This must match one of the volumes defined in the pod. |
| mountPath | The path within the container at which to mount the volume. |
| readOnly | Whether to mount the volume as read-only. Defaults to false. |
| mountPropagation | Specifies what should happen if additional filesystem volumes are mounted inside the volume. <br /> <br /> Defaults to None, which means that the container won’t receive any mounts that are mounted by the host, and the host won’t receive any mounts that are mounted by the container. <br /> <br /> HostToContainer means that the container will receive all mounts that are mounted into this volume by the host, but not the other way around. <br /> <br /> Bidirectional means that the container will receive mounts added by the host, and the host will receive mounts added by the container. |
| subPath | Defaults to “” which means that the entire volume is mounted into the container. When set to a non-empty string, only the specified subPath within the volume is mounted into the container. |
| subPathExpr | Just like subPath but can have environment variable references using the syntax $(ENV_VAR_NAME). Only environment variables that are explicitly defined in the container definition are applicable. Implicit variables such as HOSTNAME will not be resolved. You’ll learn how to specify environment variables in chapter 9. |

Table 7.2 Configuration options for a volume mount

In most cases, you only specify the name, mountPath and whether the mount should be readOnly. The mountPropagation option comes into play for advanced use-cases where additional mounts are added to the volume’s file tree later, either from the host or from the container. The subPath and subPathExpr options are useful when you want to use a single volume with multiple directories that you want to mount to different containers instead of using multiple volumes.

The subPathExpr option is also used when a volume is shared by multiple pod replicas. In chapter 9, you’ll learn how to use the Downward API to inject the name of the pod into an environment variable. By referencing this variable in subPathExpr, you can configure each replica to use its own subdirectory based on its name.