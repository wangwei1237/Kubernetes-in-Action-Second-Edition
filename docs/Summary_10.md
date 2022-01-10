# 10.5 Summary

The Kubernetes features described in this chapter will help you keep your cluster organized and make your system easier to understand. In this chapter, you learned that:

* Objects in a Kubernetes cluster are typically divided into many namespaces. Within a namespace, object names must be unique, but you can give two objects the same name if you create them in different namespaces.
* Namespaces allow different users and teams to use the same cluster as if they were using separate Kubernetes clusters.
* Each object can have several labels. Labels are key-value pairs that help identify the object. By adding labels to objects, you can effectively organize objects into groups.
* Label selectors allow you to filter objects based on their labels. You can easily filter pods that belong to a specific application, or by any other criteria if you’ve previously added the appropriate labels to those pods.
* Field selectors are like label selectors, but they allow you to filter objects based on specific fields in the object manifest. A field selector is used to list pods that run on a particular node.
* Instead of performing an operation on each pod individually, you can use a label selector to perform the same operation on a set of objects that match the label selector.
* Labels and selectors are also used internally by some object types. You can add labels to Node objects and define a node selector in a pod to schedule that pod only to those nodes that meet the specified criteria.
* In addition to labels, you can also add annotations to objects. An annotation can contain a much larger amount of data and can include whitespace and other special characters that aren’t allowed in labels. Annotations are typically used to add additional information used by tools and cluster users. They are also used to defer changes to the Kubernetes API.


In the next chapter, you’ll learn how to forward traffic to a set of pods using the Service object.