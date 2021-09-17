This chapter covers
Setting the command and arguments for the container's main process
Setting environment variables
Storing configuration in config maps
Storing sensitive information in secrets
Using the Downward API to expose pod metadata to the application
Using configMap, secret, downwardAPI and projected volumes
You’ve now learned how to use Kubernetes to run an application process and attach file volumes to it. In this chapter, you’ll learn how to configure the application - either in the pod manifest itself, or by referencing other API objects within it. You’ll also learn how to inject information about the pod itself into the application running inside it.

NOTE
You’ll find the code files for this chapter at https://github.com/luksa/kubernetes-in-action-2nd-edition/tree/master/Chapter09

9.1 Setting the command, arguments, and environment variables
Like regular applications, containerized applications can be configured using command-line arguments, environment variables, and files.

You learned that the command that is executed when a container starts is typically defined in the container image. The command is configured in the container’s Dockerfile using the ENTRYPOINT directive, while the arguments are typically specified using the CMD directive. Environment variables can also be specified using the the ENV directive in the Dockerfile. If the application is configured using configuration files, these can be added to the container image using the COPY directive. You’ve seen several examples of this in the previous chapters.

Let’s take the kiada application and make it configurable via command-line arguments and environment variables. The previous versions of the application all listen on port 8080. This will now be configurable via the --listen-port command line argument. Also, the application will read the initial status message from the environment variable INITIAL_STATUS_MESSAGE. Instead of just returning the hostname, the application now also returns the pod name and IP address, as well as the name of the cluster node on which it is running. The application obtains this information through environment variables. You can find the updated code in the book’s code repository. The container image for this new version is available at docker.io/luksa/kiada:0.4.

The updated Dockerfile, which you can also find in the code repository, is shown in the following listing.

Listing 9.1 A sample Dockerfile using several application configuration methods
FROM node:12
COPY app.js /app.js
COPY html/ /html
 
ENV INITIAL_STATUS_MESSAGE="This is the default status message"
 
ENTRYPOINT ["node", "app.js"]
CMD ["--listen-port", "8080"]

Hardcoding the configuration into the container image is the same as hardcoding it into the application source code. This is not ideal because you must rebuild the image every time you change the configuration. Also, you should never include sensitive configuration data such as security credentials or encryption keys in the container image because anyone who has access to it can easily extract them.

Instead, it’s much safer to store these files in a volume that you mount in the container. As you learned in the previous chapter, one way to do this is to store the files in a persistent volume. Another way is to use an emptyDir volume and an init container that fetches the files from secure storage and writes them to the volume. You should know how to do this if you’ve read the previous chapters, but there’s a better way. In this chapter, you’ll learn how to use special volume types to achieve the same result without using init containers. But first, let’s learn how to change the command, arguments, and environment variables without recreating the container image.

9.1.1 Setting the command and arguments
When creating a container image, the command and its arguments are specified using the ENTRYPOINT and CMD directives in the Dockerfile. Since both directives accept array values, you can specify both the command and its arguments with one of these directives or split them between the two. When the container is executed, the two arrays are concatenated to produce the full command.

Kubernetes provides two fields that are analogous to Docker’s ENTRYPOINT and CMD directives. The two fields are called command and args, respectively. You specify these fields in the container definition in your pod manifest. As with Docker, the two fields accept array values, and the resulting command executed in the container is derived by concatenating the two arrays.

Figure 9.1 Overriding the command and arguments in the pod manifest



Normally, you use the ENTRYPOINT directive to specify the bare command, and the CMD directive to specify the arguments. This allows you to override the arguments in the pod manifest without having to specify the command again. If you want to override the command, you can still do so. And you can do it without overriding the arguments.

The following table shows the equivalent pod manifest field for each of the two Dockerfile directives.

Table 9.1 Specifying the command and arguments in the Dockerfile vs the pod manifest
Dockerfile

Pod manifest

Description

ENTRYPOINT

command

The executable file that runs in the container. This may contain arguments in addition to the executable.

CMD

args

Additional arguments passed to the command specified with the ENTRYPOINT directive or the command field.

Let’s look at two examples of setting the command and args fields.

Setting the command
Imagine you want to run the Kiada application with CPU and heap profiling enabled. With Node.JS, you can enable profiling by passing the --cpu-prof and --heap-prof arguments to the node command. Instead of modifying the Dockerfile and rebuilding the image, you can do this by modifying the pod manifest, as shown in the following listing.

Listing 9.2 A container definition with the command specified
kind: Pod
spec:
  containers:
  - name: kiada
    image: luksa/kiada:0.4
    command: ["node", "--cpu-prof", "--heap-prof", "app.js"]


When you deploy the pod in the listing, the node --cpu-prof --heap-prof app.js command is run instead of the default command specified in the Dockerfile, which is node app.js.

As you can see in the listing, the command field, just like its Dockerfile counterpart, accepts an array of strings representing the command to be executed. The array notation used in the listing is great when the array contains only a few elements, but becomes difficult to read as the number of elements increases. In this case, you’re better off using the following notation:

command:
    - node
    - --cpu-prof
    - --heap-prof
    - app.js

TIP
Values that the YAML parser might interpret as something other than a string must be enclosed in quotes. This includes numeric values such as 1234, and Boolean values such as true and false. Some other special strings must also be quoted, otherwise they would also be interpreted as Boolean or other types. These include the values true, false, yes, no, on, off, y, n, t, f, null, and others.

Setting command arguments
Command line arguments can be overridden with the args field, as shown in the following listing.

Listing 9.3 A container definition with the args fields set
1
2
3
4
5
6
kind: Pod
spec:
  containers:
  - name: kiada
    image: luksa/kiada:0.4
    args: ["--listen-port", "9090"]

The pod manifest in the listing overrides the default --listen-port 8080 arguments set in the Dockerfile with --listen-port 9090. When you deploy this pod, the full command that runs in the container is node app.js --listen-port 9090. The command is a concatenation of the ENTRYPOINT in the Dockerfile and the args field in the pod manifest.

9.1.2 Setting environment variables in a container
Containerized applications are often configured using environment variables. Just like the command and arguments, you can set environment variables for each of the pod’s containers, as shown in figure 9.2.

Figure 9.2 Environment variables are set per container.

NOTE
As I write this, environment variables can only be set for each container individually. It isn’t possible to set a global set of environment variables for the entire pod and have them inherited by all its containers.

You can set an environment variable to a literal value, have it reference another environment variable, or obtain the value from an external source. Let’s see how.

Setting a literal value to an environment variable
Version 0.4 of the Kiada application displays the name of the pod, which it reads from the environment variable POD_NAME. It also allows you to set the status message using the environment variable INITIAL_STATUS_MESSAGE. Let’s set these two variables in the pod manifest.

To set the environment variable, you could add the ENV directive to the Dockerfile and rebuild the image, but the faster way is to add the env field to the container definition in the pod manifest, as I’ve done in the file pod.kiada.env-value.yaml shown in the following listing.

Listing 9.4 Setting environment variables in the pod manifest
1
2
3
4
5
6
7
8
9
10
11
12
13
kind: Pod
metadata:
  name: kiada
spec:
  containers:
  - name: kiada
    image: luksa/kiada:0.4
    env:
    - name: POD_NAME
      value: kiada
    - name: INITIAL_STATUS_MESSAGE
      value: This status message is set in the pod spec.
    ...


As you can see in the listing, the env field takes an array of values. Each entry in the array specifies the name of the environment variable and its value.

NOTE
Since environment variables values must be strings, you must enclose values that aren’t strings in quotes to prevent the YAML parser from treating them as anything other than a string. As explained in section 9.1.1, this also applies to strings such as yes, no, true, false, and so on.

When you deploy the pod in the listing and send an HTTP request to the application, you should see the pod name and status message that you specified using environment variables. You can also run the following command to examine the environment variables in the container. You’ll find the two environment variables in the following output:

$ kubectl exec kiada -- env
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=kiada
NODE_VERSION=12.19.1
YARN_VERSION=1.22.5
POD_NAME=kiada
INITIAL_STATUS_MESSAGE=This status message is set in the pod spec.
KUBERNETES_SERVICE_HOST=10.96.0.1
...
KUBERNETES_SERVICE_PORT=443

As you can see, there are a few other variables set in the container. They come from different sources - some are defined in the container image, some are added by Kubernetes, and the rest come from elsewhere. While there is no way to know where each of the variables comes from, you’ll learn to recognize some of them. For example, the ones added by Kubernetes relate to the Service object, which is covered in chapter 11. To determine where the rest come from, you can inspect the pod manifest and the Dockerfile of the container image.

Using variable references in environment variable values
In the previous example, you set a fixed value for the environment variable INITIAL_STATUS_MESSAGE, but you can also reference other environment variables in the value by using the syntax $(VAR_NAME).

For example, you can reference the variable POD_NAME within the status message variable as shown in the following listing, which shows part of the file pod.kiada.env-value-ref.yaml.

Listing 9.5 Referring to an environment variable in another variable
1
2
3
4
5
env:
- name: POD_NAME
  value: kiada
- name: INITIAL_STATUS_MESSAGE
  value: My name is $(POD_NAME). I run NodeJS version $(NODE_VERSION).

Notice that one of the references points to the environment variable POD_NAME defined above, whereas the other points to the variable NODE_VERSION set in the container image. You saw this variable when you ran the env command in the container earlier. When you deploy the pod, the status message it returns is the following:

1
My name is kiada. I run NodeJS version $(NODE_VERSION).


As you can see, the reference to NODE_VERSION isn’t resolved. This is because you can only use the $(VAR_NAME) syntax to refer to variables defined in the same manifest. The referenced variable must be defined before the variable that references it. Since NODE_VERSION is defined in the NodeJS image’s Dockerfile and not in the pod manifest, it can’t be resolved.

NOTE
If a variable reference can’t be resolved, the reference string remains unchanged.

NOTE
When you want a variable to contain the literal string $(VAR_NAME) and don’t want Kubernetes to resolve it, use a double dollar sign as in $${VAR_NAME). Kubernetes will remove one of the dollar signs and skip resolving the variable.

Using variable references in the command and arguments
You can refer to environment variables defined in the manifest not only in other variables, but also in the command and args fields you learned about in the previous section. For example, the file pod.kiada.env-value-ref-in-args.yaml defines an environment variable named LISTEN_PORT and references it in the args field. The following listing shows the relevant part of this file.

Listing 9.6 Referring to an environment variable in the args field
1
2
3
4
5
6
7
8
9
10
spec:
  containers:
  - name: kiada
    image: luksa/kiada:0.4
    args:
    - --listen-port
    - $(LISTEN_PORT)
    env:
    - name: LISTEN_PORT
      value: "8080"

This isn’t the best example, since there’s no good reason to use a variable reference instead of just specifying the port number directly. But later you’ll learn how to get the environment variable value from an external source. You can then use a reference as shown in the listing to inject that value into the container’s command or arguments.

Referring to environment variables that aren’t in the manifest
Just like using references in environment variables, you can only use the $(VAR_NAME) syntax in the command and args fields to reference variables that are defined in the pod manifest. You can’t reference environment variables defined in the container image, for example.

However, you can use a different approach. If you run the command through a shell, you can have the shell resolve the variable. If you are using the bash shell, you can do this by referring to the variable using the syntax $VAR_NAME or ${VAR_NAME} instead of $(VAR_NAME).

For example, the command in the following listing correctly prints the value of the HOSTNAME environment variable even though it’s not defined in the pod manifest but is initialized by the operating system. You can find this example in the file pod.env-var-references-in-shell.yaml.

Listing 9.7 Referring to environment variables in a shell command
1
2
3
4
5
6
7
containers:
- name: main
  image: alpine
  command:
  - sh
  - -c
  - 'echo "Hostname is $HOSTNAME."; sleep infinity'


SETTING THE POD’S FULLY QUALIFIED DOMAIN NAME
While we’re on the subject of the pod’s hostname, this is a good time to explain that the pod’s hostname and subdomain are configurable in the pod manifest. By default, the hostname is the same as the pod’s name, but you can override it using the hostname field in the pod’s spec. You can also set the subdomain field so that the fully qualified domain name (FQDN) of the pod is as follows: <hostname>.<subdomain>.<pod namespace>.svc.<cluster domain>

This is only the internal FQDN of the pod. It isn’t resolvable via DNS without additional steps, which are explained in chapter 11. You can find a sample pod that specifies a custom hostname for the pod in the file pod.kiada.hostname.yaml.

9.2 Using a config map to decouple configuration from the pod
In the previous section, you learned how to hardcode configuration directly into your pod manifests. While this is much better than hard-coding in the container image, it’s still not ideal because it means you might need a separate version of the pod manifest for each environment you deploy the pod to, such as your development, staging, or production cluster.

To reuse the same pod definition in multiple environments, it’s better to decouple the configuration from the pod manifest. One way to do this is to move the configuration into a ConfigMap object, which you then reference in the pod manifest. This is what you’ll do next.

9.2.1 Introducing ConfigMaps
A ConfigMap is a Kubernetes API object that simply contains a list of key/value pairs. The values can range from short strings to large blocks of structured text that you typically find in an application configuration file. Pods can reference one or more of these key/value entries in the config map. A pod can refer to multiple config maps, and multiple pods can use the same config map.

To keep applications Kubernetes-agnostic, they typically don’t read the ConfigMap object via the Kubernetes REST API. Instead, the key/value pairs in the config map are passed to containers as environment variables or mounted as files in the container’s filesystem via a configMap volume, as shown in the following figure.

Figure 9.3 Pods use config maps through environment variables and configMap volumes.

In the previous section you learned how to reference environment variables in command-line arguments. You can use this technique to pass a config map entry that you’ve exposed as an environment variable into a command-line argument.

Regardless of how an application consumes config maps, storing the configuration in a separate object instead of the pod allows you to keep the configuration separate for different environments by simply keeping separate config map manifests and applying each to the environment for which it is intended. Because pods reference the config map by name, you can deploy the same pod manifest across all your environments and still have a different configuration for each environment by using the same config map name, as shown in the following figure.

Figure 9.4 Deploying the same pod manifest and different config map manifests in different environments

9.2.2 Creating a ConfigMap object
Let’s create a config map and use it in a pod. The following is a simple example where the config map contains a single entry used to initialize the environment variable INITIAL_STATUS_MESSAGE for the kiada pod.

Creating a config map with the kubectl create configmap command
As with pods, you can create the ConfigMap object from a YAML manifest, but a faster way is to use the kubectl create configmap command as follows:

1
2
$ kubectl create configmap kiada-config --from-literal status-message="This status message is set in the kiada-config config map"
configmap "kiada-config" created


NOTE
Keys in a config map may only consist of alphanumeric characters, dashes, underscores, or dots. Other characters are not allowed.

Running this command creates a config map called kiada-config with a single entry. The key and value are specified with the --from-literal argument.

In addition to --from-literal, the kubectl create configmap command also supports sourcing the key/value pairs from files. The following table explains the available methods.

Table 9.2 Options for creating config map entries using kubectl create configmap
Option

Description

--from-literal

Inserts a key and a literal value into the config map. Example: --from-literal mykey=myvalue.

--from-file

Inserts the contents of a file into the config map. The behavior depends on the argument that comes after --from-file:

If only the filename is specified (example: --from-file myfile.txt), the base name of the file is used as the key and the entire contents of the file are used as the value.

If key=file is specified (example: --from-file mykey=myfile.txt), the contents of the file are stored under the specified key.

If the filename represents a directory, each file contained in the directory is included as a separate entry. The base name of the file is used as the key, and the contents of the file are used as the value. Subdirectories, symbolic links, devices, pipes, and files whose base name isn’t a valid config map key are ignored.

--from-env-file

Inserts each line of the specified file as a separate entry (example: --from-env-file myfile.env). The file must contain lines with the following format: key=value

Config maps usually contain more than one entry. To create a config map with multiple entries, you can use multiple arguments --from-literal, --from-file, and --from-env-file, or a combination thereof.

Creating a config map from a YAML manifest
Alternatively, you can create the config map from a YAML manifest file. The following listing shows the contents of an equivalent manifest file named cm.kiada-config.yaml, which is available in the code repository You can create the config map by applying this file using kubectl apply.

Listing 9.8 A config map manifest file
1
2
3
4
5
6
apiVersion: v1
kind: ConfigMap
metadata:
  name: kiada-config
data:
  status-message: This status message is set in the kiada-config config map


Listing config maps and displaying their contents
Config maps are Kubernetes API objects that live alongside pods, nodes, persistent volumes, and the others you’ve learned about so far. You can use various kubectl commands to perform CRUD operations on them. For example, you can list config maps with:

1
$ kubectl get cm


NOTE
The shorthand for configmaps is cm.

You can display the entries in the config map by instructing kubectl to print its YAML manifest:

1
$ kubectl get cm kiada-config -o yaml


NOTE
Because YAML fields are output in alphabetical order, you’ll find the data field at the top of the output.

TIP
To display only the key/value pairs, combine kubectl with jq. For example: kubectl get cm kiada-config -o json | jq .data. Display the value of a given entry as follows: kubectl... | jq '.data["status-message"]'.

9.2.3 Injecting config map values into environment variables
In the previous section, you created the kiada-config config map. Let’s use it in the kiada pod.

Injecting a single config map entry
To inject the single config map entry into an environment variable, you just need to replace the value field in the environment variable definition with the valueFrom field and refer to the config map entry. The following listing shows the relevant part of the pod manifest. The full manifest can be found in the file pod.kiada.env-valueFrom.yaml.

Listing 9.9 Setting an environment variable from a config map entry
1
2
3
4
5
6
7
8
9
10
11
12
13
14
kind: Pod
...
spec:
  containers:
  - name: kiada
    env:
    - name: INITIAL_STATUS_MESSAGE
      valueFrom:
        configMapKeyRef:
          name: kiada-config
          key: status-message
          optional: true
    volumeMounts:
    - ...


Let me break down the definition of the environment variable that you see in the listing. Instead of specifying a fixed value for the variable, you declare that the value should be obtained from a config map. The name of the config map is specified using the name field, whereas the key field specifies the key within that map.

Create the pod from this manifest and inspect its environment variables using the following command:

1
2
3
4
$ kubectl exec kiada -- env
...
INITIAL_STATUS_MESSAGE=This status message is set in the kiada-config config map
...

The status message should also appear in the pod’s response when you access it via curl or your browser.

Marking a reference optional
In the previous listing, the reference to the config map key is marked as optional so that the container can be executed even if the config map or key is missing. If that’s the case, the environment variable isn’t set. You can mark the reference as optional because the Kiada application will run fine without it. You can delete the config map and deploy the pod again to confirm this.

NOTE
If a config map or key referenced in the container definition is missing and not marked as optional, the pod will still be scheduled normally. The other containers in the pod are started normally. The container that references the missing config map key is started as soon as you create the config map with the referenced key.

Injecting the entire config map
The env field in a container definition takes an array of values, so you can set as many environment variables as you need. However, if you want to set more than a few variables, it can become tedious and error prone to specify them one at a time. Fortunately, by using the envFrom instead of the env field, you can inject all the entries that are in the config map without having to specify each key individually.

The downside to this approach is that you lose the ability to transform the key to the environment variable name, so the keys must already have the proper form. The only transformation that you can do is to prepend a prefix to each key.

For example, the Kiada application reads the environment variable INITIAL_STATUS_MESSAGE, but the key you used in the config map is status-message. You must change the config map key to match the expected environment variable name if you want it to be read by the application when you use the envFrom field to inject the entire config map into the pod. I’ve already done this in the cm.kiada-config.envFrom.yaml file. In addition to the INITIAL_STATUS_MESSAGE key, it contains two other keys to demonstrate that they will all be injected into the container’s environment.

Replace the config map with the one in the file by running the following command:

1
$ kubectl replace -f cm.kiada-config.envFrom.yaml

The pod manifest in the pod.kiada.envFrom.yaml file uses the envFrom field to inject the entire config map into the pod. The following listing shows the relevant part of the manifest.

Listing 9.10 Using envFrom to inject the entire config map into environment variables
1
2
3
4
5
6
7
8
9
kind: Pod
...
spec:
  containers:
  - name: kiada
    envFrom:
    - configMapRef:
        name: kiada-config
        optional: true

Instead of specifying both the config map name and the key as in the previous example, only the config map name is specified. If you create the pod from this manifest and inspect its environment variables, you’ll see that it contains the environment variable INITIAL_STATUS_MESSAGE as well as the other two keys defined in the config map.

As before, you can mark the config map reference as optional, in which case the container will run even if the config map doesn’t exist. By default, this isn’t the case. Containers that reference config maps are prevented from starting until the referenced config maps exist.

Injecting multiple config maps
Listing 9.10 shows that the envFrom field takes an array of values, which means you can combine entries from multiple config maps. If two config maps contain the same key, the last one takes precedence. You can also combine the envFrom field with the env field if you wish to inject all entries of one config map and particular entries of another.

NOTE
When an environment variable is configured in the env field, it takes precedence over environment variables set in the envFrom field.

Prefixing keys
Regardless of whether you inject a single config map or multiple config maps, you can set an optional prefix for each config map. When their entries are injected into the container’s environment, the prefix is prepended to each key to yield the environment variable name.

9.2.4 Injecting config map entries into containers as files
Environment variables are typically used to pass small single-line values to the application, while multiline values are usually passed as files. Config map entries can also contain larger blocks of data that can be projected into the container using the special configMap volume type.

NOTE
The amount of information that can fit in a config map is dictated by etcd, the underlying data store used to store API objects. At this point, the maximum size is on the order of one megabyte.

A configMap volume makes the config map entries available as individual files. The process running in the container gets the entry’s value by reading the contents of the file. This mechanism is most often used to pass large config files to the container, but can also be used for smaller values, or combined with the env or envFrom fields to pass large entries as files and others as environment variables.

Creating config map entries from files
In chapter 4, you deployed the kiada pod with an Envoy sidecar that handles TLS traffic for the pod. Because volumes weren’t explained at that point, the configuration file, TLS certificate, and private key that Envoy uses were built into the container image. It would be more convenient if these files were stored in a config map and injected into the container. That way you could update them without having to rebuild the image. But since the security considerations of these files are different, we must handle them differently. Let’s focus on the config file first.

You’ve already learned how to create a config map from a literal value using the kubectl create configmap command. This time, instead of creating the config map directly in the cluster, you’ll create a YAML manifest for the config map so that you can store it in a version control system alongside your pod manifest.

Instead of writing the manifest file by hand, you can create it using the same kubectl create command that you used to create the object directly. The following command creates the YAML file for a config map named kiada-envoy-config:

$ kubectl create configmap kiada-envoy-config \
      --from-file=envoy.yaml \
      --from-file=dummy.bin \
      --dry-run=client -o yaml > cm.kiada-envoy-config.yaml

The config map will contain two entries that come from the files specified in the command. One is the envoy.yaml configuration file, while the other is just some random data to demonstrate that binary data can also be stored in a config map.

When using the --dry-run option, the command doesn’t create the object in the Kubernetes API server, but only generates the object definition. The -o yaml option prints the YAML definition of the object to standard output, which is then redirected to the cm.kiada-envoy-config.yaml file. The following listing shows the contents of this file.

Listing 9.11 A config map manifest containing a multi-line value
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
apiVersion: v1
binaryData:
  dummy.bin: n2VW39IEkyQ6Jxo+rdo5J06Vi7cz5...
data:
  envoy.yaml: |
    admin:
      access_log_path: /var/log/envoy.admin.log
      address:
        socket_address:
          protocol: TCP
          address: 0.0.0.0
    ...
kind: ConfigMap
metadata:
  creationTimestamp: null
  name: kiada-envoy-config


As you can see in the listing, the binary file ends up in the binaryData field, whereas the envoy config file is in the data field, which you know from the previous sections. If a config map entry contains non-UTF-8 byte sequences, it must be defined in the binaryData field. The kubectl create configmap command automatically determines where to put the entry. The values in this field are Base64 encoded, which is how binary values are represented in YAML.

In contrast, the contents of the envoy.yaml file are clearly visible in the data field. In YAML, you can specify multi-line values using a pipeline character and appropriate indentation. See the YAML specification on YAML.org for more ways to do this.

MIND YOUR WHITESPACE HYGIENE WHEN CREATING CONFIG MAPS
When creating config maps from files, make sure that none of the lines in the file contain trailing whitespace. If any line ends with whitespace, the value of the entry in the manifest is formatted as a quoted string with the newline character escaped. This makes the manifest incredibly hard to read and edit.

Compare the formatting of the two values in the following config map:

$ kubectl create configmap whitespace-demo \
    --from-file=envoy.yaml \
    --from-file=envoy-trailingspace.yaml \
    --dry-run=client -o yaml
apiVersion: v1
data:
  envoy-trailingspace.yaml: "admin: \n  access_log_path: /var/log/envoy.admin.log\n
    \ address:\n    socket_address:\n      protocol: TCP\n      address: 0.0.0.0\n
    \     port_value: 9901\nstatic_resources:\n  listeners:\n  - name: listener_0\n...
  envoy.yaml: |
    admin:
      access_log_path: /var/log/envoy.admin.log
      address:
        socket_address:...


Notice that the envoy-trailingspace.yaml file contains a space at the end of the first line. This causes the config map entry to be presented in a not very human-friendly format. In contrast, the envoy.yaml file contains no trailing whitespace and is presented as an unescaped multi-line string, which makes it easy to read and modify in place.

 

Don’t apply the config map manifest file to the Kubernetes cluster yet. You’ll first create the pod that refers to the config map. This way you can see what happens when a pod points to a config map that doesn’t exist.

Using a configMap volume in a pod
To make config map entries available as files in the container’s filesystem, you define a configMap volume in the pod and mount it in the container, as shown in the following listing.

Listing 9.12 Defining a configMap volume in a pod: pod.kiada-ssl.configmap-volume.yaml
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
apiVersion: v1
kind: Pod
metadata:
  name: kiada-ssl
spec:
  volumes:
  - name: envoy-config
    configMap:
      name: kiada-envoy-config
  ...
  containers:
  ...
  - name: envoy
    image: luksa/kiada-ssl-proxy:0.1
    volumeMounts:
    - name: envoy-config
      mountPath: /etc/envoy
  ...


If you’ve read the previous two chapters, the definitions of the volume and volumeMount in this listing should be clear. As you can see, the volume is a configMap volume that points to the kiada-envoy-config config map, and it’s mounted in the envoy container under /etc/envoy. The volume contains the envoy.yaml and dummy.bin files that match the keys in the config map.

Create the pod from the manifest file and check its status. Here’s what you’ll see:

1
2
3
$ kubectl get po
NAME        READY   STATUS              RESTARTS   AGE
Kiada-ssl   0/2     ContainerCreating   0          2m

Because the pod’s configMap volume references a config map that doesn’t exist, and the reference isn’t marked as optional, the container can’t run.

Marking a configMap volume as optional
Previously, you learned that if a container contains an environment variable definition that refers to a config map that doesn’t exist, the container is prevented from starting until you create that config map. You also learned that this doesn’t prevent the other containers from starting. What about the case at hand where the missing config map is referenced in a volume?

Because all of the pod’s volumes must be set up before the pod’s containers can be started, referencing a missing config map in a volume prevents all the containers in the pod from starting, not just the container in which the volume is mounted. An event is generated indicating the problem. You can display it with the kubectl describe pod or kubectl get events command, as explained in the previous chapters.

NOTE
A configMap volume can be marked as optional by adding the line optional: true to the volume definition. If a volume is optional and the config map doesn’t exist, the volume is not created, and the container is started without mounting the volume.

To enable the pod’s containers to start, create the config map by applying the cm.kiada-envoy-config.yaml file you created earlier. Use the kubectl apply command. After doing this, the pod should start, and you should be able to confirm that both config map entries are exposed as files in the container by listing the contents of the /etc/envoy directory as follows:

1
2
3
$ kubectl exec kiada-ssl -c envoy -- ls /etc/envoy
dummy.bin
envoy.yaml

Projecting only specific config map entries
》》》》》》》》》》》》》》》