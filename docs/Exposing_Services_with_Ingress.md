# 12 Exposing Services with Ingress

This chapter covers
* Creating Ingress objects
* Deploying and understanding Ingress controllers
* Securing ingresses with TLS
* Adding additional configuration to an Ingress
* Using IngressClasses when multiple controllers are installed
* Using Ingresses with non-service backends

In the previous chapter, you learned how to use the Service object to expose a group of pods at a stable IP address. If you use the LoadBalancer service type, the service is made available to clients outside the cluster through a load balancer. This approach is fine if you only need to expose a single service externally, but it becomes problematic with large numbers of services, since each service needs its own public IP address.

Fortunately, by exposing these services through an Ingress object instead, you only need a single IP address. Additionally, the Ingress provides other features such as HTTP authentication, cookie-based session affinity, URL rewriting, and others that Service objects can’t.

{% hint style='info' %}
NOTE

You’ll find the code files for this chapter at https://github.com/luksa/kubernetes-in-action-2nd-edition/tree/master/Chapter12.
{% endhint %}