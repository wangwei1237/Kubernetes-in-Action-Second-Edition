# 9.6 Summary
This wraps up this chapter on how to pass configuration data to containers. You’ve learned that:

* The default command and arguments defined in the container image can be overridden in the pod manifest.
* Environment variables for each container can also be set in the pod manifest. Their values can be hardcoded in the manifest or can come from other Kubernetes API objects.
* Config maps are Kubernetes API objects used to store configuration data in the form of key/value pairs. Secrets are another similar type of object used to store sensitive data such as credentials, certificates, and authentication keys.
* Entries of both config maps and secrets can be exposed within a container as environment variables or as files via the configMap and secret volumes.
* Config maps and other API objects can be edited in place using the kubectl edit command.
* The Downward API provides a way to expose the pod metadata to the application running within it. Like config maps and secrets, this data can be injected into environment variables or files.
* Projected volumes can be used to combine multiple volumes of possibly different types into a composite volume that is mounted into a single directory, rather than being forced to mount each individual volume into its own directory.

You’ve now seen that an application deployed in Kubernetes may require many additional objects. If you are deploying many applications in the same cluster, you need organize them so that everyone can see what fits where. In the next chapter, you’ll learn how to do just that.