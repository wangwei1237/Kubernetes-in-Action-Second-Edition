# 12.6 Using custom resources instead of services as backends
In this chapter, the backends referenced in the Ingress have always been Service objects. However, some ingress controllers allow you to use other resources as backends.

Theoretically, an ingress controller could allow using an Ingress object to expose the contents of a ConfigMap or PersistentVolume, but it’s more typical for controllers to use resource backends to provide an option for configuring advanced Ingress routing rules through a custom resource.

## 12.6.1 Using a custom object to configure Ingress routing
The Citrix ingress controller provides the HTTPRoute custom object type, which allows you to configure where the ingress should route HTTP requests. As you can see in the following manifest, you don’t specify a Service object as the backend, but you instead specify the `kind`, `apiGroup`, and `name` of the HTTPRoute object that contains the routing rules.

Listing 12.15 Example Ingress object using a resource backend

```YAML
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-ingress
spec:
  ingressClassName: citrix
  rules:
  - host: example.com
    http:
      paths:
      - pathType: ImplementationSpecific
        backend:
          resource:
            apiGroup: citrix.com
            kind: HTTPRoute
            name: my-example-route
```

The Ingress object in the listing specifies a single rule. It states that the ingress controller should forward traffic destined for the host `example.com` according to the configuration specified in the object of the kind `HTTPRoute` (from the API group `citrix.com`) named `my-example-route`. Since the HTTPRoute object isn’t part of the Kubernetes API, its contents are beyond the scope of this book, but you can probably guess that it contains rules like those in the Ingress object but specified differently and with additional configuration options.

At the time of writing, ingress controllers that support custom resource backends are rare, but maybe you might want to implement one yourself. By the time you finish reading this book, you’ll know how.