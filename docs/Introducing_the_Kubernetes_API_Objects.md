# Introducing the Kubernetes API objects

{% hint style='info' %}
This chapter covers

* Managing a Kubernetes cluster and the applications it hosts via its API
* Understanding the structure of Kubernetes API objects
* Retrieving and understanding an object’s YAML or JSON manifest
* Inspecting the status of cluster nodes via Node objects
* Inspecting cluster events through Event objects
{% endhint %}

The previous chapter introduced three fundamental objects that make up a deployed application. You created a Deployment object that spawned multiple Pod objects representing individual instances of your application and exposed them to the world by creating a Service object that deployed a load balancer in front of them.

The chapters in the second part of this book explain these and other object types in detail. In this chapter, the common features of Kubernetes objects are presented using the example of Node and Event objects.