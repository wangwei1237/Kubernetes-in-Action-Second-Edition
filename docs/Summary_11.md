# 11.7 Summary
In this chapter, you finally connected the Kiada pods to the Quiz and Service pods. Now you can use the Kiada suite to test the knowledge you’ve acquired so far and refresh your memory with quotes from this book. In this chapter, you learned that:

* Pods communicate over a flat network that allows any pod to reach any other pod in the cluster, regardless of the actual network topology connecting the cluster nodes.
* A Kubernetes service makes a group of pods available under a single IP address. While the IPs of the pods may change, the IP of the service remains constant.
* The cluster IP of the service is reachable from inside the cluster, but NodePort and LoadBalancer services are also accessible from outside the cluster.
* Service endpoints are either determined by a label selector specified in the Service object or configured manually. These endpoints are stored in the Endpoints and EndpointSlice objects.
* Client pods can find services using the cluster DNS or environment variables. Depending on the type of Service, the following DNS records may be created: `A`, `AAAA`, `SRV`, and `CNAME`.
* Services can be configured to forward external traffic only to pods on the same node that received the external traffic, or to pods anywhere in the cluster. They can also be configured to route internal traffic only to pods on the same node as the pod from which the traffic originates from. Topology-aware routing ensures that traffic isn’t routed across availability zones when a local pod can provide the requested service.
* Pods don’t become service endpoints until they’re ready. By implementing a readiness probe handler in an application, you can define what readiness means in the context of that particular application.

In the next chapter, you’ll learn how to use Ingress objects to make multiple services accessible through a single external IP address.