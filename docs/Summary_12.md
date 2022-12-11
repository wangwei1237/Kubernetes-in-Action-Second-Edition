# 12.7 Summary
In this chapter, you learned how to create Ingress objects to make one or more services accessible to external clients. You learned that:

* An Ingress controller configures an L7 load balancer or reverse proxy based on the configuration in the Ingress object.
* While a Service is an abstraction over a set of Pods, an Ingress is an abstraction over a set of Services.
* An Ingress requires a single public IP regardless of the number of services it exposes, whereas each LoadBalancer service requires its own public IP.
* External clients must resolve the hostnames specified in the Ingress object to the IP address of the ingress proxy. To accomplish this, you must add the necessary records to the DNS server responsible for the domain to which the host belongs. Alternatively, for development purposes, you can modify the /etc/hosts file on your local machine.
* An Ingress operates at Layer 7 of the OSI model and can therefore provide HTTP-related functionality that Services operating at Layer 4 cannot.
* An Ingress proxy usually forwards HTTP requests directly to the backend pod without going through the service IP, but this depends on the ingress implementation.
* The Ingress object contains rules that specify to which service the HTTP request received by the ingress proxy should be forwarded based on the host and path in the request. Each rule can specify an exact host or one with a wildcard and either an exact path or path prefix.
* The default backend is a catch-all rule that determines which service should handle requests that don’t match any rule.
* An Ingress can be configured to expose services over TLS. The Ingress proxy can terminate the TLS connection and forward the HTTP request to the backend pod unencrypted. Some ingress implementations support TLS passthrough.
* Ingress configuration options that are specific to a particular ingress implementation are set via annotations of the Ingress object or through custom Kubernetes object kinds that the controller provides.
* A Kubernetes cluster can run multiple ingress controller implementations simultaneously. When you create an Ingress object, you specify the IngressClass. The IngressClass object specifies which controller should process the Ingress object. Optionally, the IngressClass can also specify parameters for the controller.

You now understand how to expose groups of pods both internally and externally. In the next chapter, you’ll learn how to manage these pods as a unit and replicate them via a Deployment object.