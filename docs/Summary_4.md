# Summary
In this chapter, you’ve learned:

* Kubernetes provides a RESTful API for interaction with a cluster. API Objects map to actual components that make up the cluster, including applications, load balancers, nodes, storage volumes, and many others.
* An object instance can be represented by many resources. A single object type can be exposed through several resources that are just different representations of the same thing.
* Kubernetes API objects are described in YAML or JSON manifests. Objects are created by posting a manifest to the API. The status of the object is stored in the object itself and can be retrieved by requesting the object from the API with a GET request.
* All Kubernetes API objects contain Type and Object Metadata, and most have a spec and status sections. A few object types don’t have these two sections, because they only contain static data.
* Controllers bring objects to life by constantly watching for changes in their spec, updating the cluster state and reporting the current state via the object’s status field.
* As controllers manage Kubernetes API objects, they emit events to reveal what actions they have performed. Like everything else, events are represented by Event objects and can be retrieved through the API. Events signal what is happening to a Node or other object. They show what has recently happened to the object and can provide clues as to why it is broken.
* The kubectl explain command provides a quick way to look up documentation on a specific object kind and its fields from the command line.
* The status in a Node object contains information about the node’s IP address and hostname, its resource capacity, conditions, cached container images and other information about the node. Which pods are running on the node is not part of the node’s status, but the kubectl describe node commands gets this information from the pods resource.
* Many object types use status conditions to signal the state of the component that the object represents. For nodes, these conditions are MemoryPressure, DiskPressure and PIDPressure. Each condition is either True, False, or Unknown and has an associated reason and message that explain why the condition is in the specified state.

You should now be familiar with the general structure of the Kubernetes API objects. In the next chapter, you’ll learn about the Pod object, the fundamental building block which represents one running instance of your application.