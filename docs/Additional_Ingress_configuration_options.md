# 12.4 Additional Ingress configuration options
I hope you haven’t forgotten that you can use the `kubectl explain` command to learn more about a particular API object type, and that you use it regularly. If not, now is a good time to use it to see what else you can configure in an Ingress object’s `spec` field. Inspect the output of the following command:

```shell
$ kubectl explain ingress.spec
```

Look at the list of fields displayed by this command. You may be surprised to see that in addition to the `defaultBackend`, `rules`, and `tls` fields explained in the previous sections, only one other field is supported, namely `ingressClassName`. This field is used to specify which ingress controller should process the Ingress object. You’ll learn more about it later. For now, I want to focus on the lack of additional configuration options that HTTP proxies normally provide.

The reason you don’t see any other fields for specifying these options is that it would be nearly impossible to include all possible configuration options for every possible ingress implementation in the Ingress object’s schema. Instead, these custom options are configured via annotations or in separate custom Kubernetes API objects.

Each ingress controller implementation supports its own set of annotations or objects. I mentioned earlier that the Nginx ingress controller uses annotations to configure TLS passthrough. Annotations are also used to configure HTTP authentication, session affinity, URL rewriting, redirects, Cross-Origin Resource Sharing (CORS), and more. The list of supported annotations can be found at https://kubernetes.github.io/ingress-nginx/user-guide/nginx-configuration/annotations/.

I don’t want to go into each of these annotations, since they’re implementation specific, but I do want to show you an example of how you can use them.

## 12.4.1 Configuring the Ingress using annotations
You learned in the previous chapter that Kubernetes services only support client IP-based session affinity. Cookie-based session affinity isn’t supported because services operate at Layer 4 of the OSI network model, whereas cookies are part of Layer 7 (HTTP). However, because Ingresses operate at L7, they can support cookie-based session affinity. This is the case with the Nginx ingress controller that I use in the following example.

#### Using annotations to enable cookie-based session affinity in Nginx ingresses
The following listing shows an example of using Nginx-ingress-specific annotations to enable cookie-based session affinity and configure the session cookie name. The manifest shown in the listing can be found in the `ing.kiada.nginx-affinity.yaml` file.

Listing 12.9 Using annotations to configure session affinity in an Nginx ingress

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: kiada
  annotations:
    nginx.ingress.kubernetes.io/affinity: cookie
    nginx.ingress.kubernetes.io/session-cookie-name: SESSION_COOKIE
spec:
  ...
```

In the listing, you can see the annotations `nginx.ingress.kubernetes.io/affinity` and `nginx.ingress.kubernetes.io/session-cookie-name`. The first annotation enables cookie-based session affinity, and the second sets the cookie name. The annotation key prefix indicates that these annotations are specific to the Nginx ingress controller and are ignored by other implementations.

#### Testing the cookie-based session affinity
If you want to see session affinity in action, first apply the manifest file, wait until the Nginx configuration is updated, and then retrieve the cookie as follows:

```shell
$ curl -I http://kiada.example.com --resolve kiada.example.com:80:11.22.33.44
HTTP/1.1 200 OK
Date: Mon, 06 Dec 2021 08:58:10 GMT
Content-Type: text/plain
Connection: keep-alive
Set-Cookie: SESSION_COOKIE=1638781091; Path=/; HttpOnly
```

You can now include this cookie in your request by specifying the `Cookie` header:

```shell
$ curl -H "Cookie: SESSION_COOKIE=1638781091" http://kiada.example.com \
  --resolve kiada.example.com:80:11.22.33.44
```

If you run this command several times, you’ll notice that the HTTP request is always forwarded to the same pod, which indicates that the session affinity is using the cookie.

## 12.4.2 Configuring the Ingress using additional API objects
Some ingress implementations don’t use annotations for additional ingress configuration, but instead provide their own object kinds. In the previous section, you saw how to use annotations to configure session affinity when using the Nginx ingress controller. In the current section, you’ll learn how to do the same in Google Kubernetes Engine.

{% hint style='info' %}
NOTE

You’ll learn how to create your own custom object kinds via the CustomResourceDefinition object in chapter 29.
{% endhint %}

#### Using the BackendConfig object type to enable cookie-based session affinity in GKE
In clusters running on GKE, a custom object of type BackendConfig can be found in the Kubernetes API. You create an instance of this object and reference it by name in the Service object to which you want to apply the object. You reference the object using the `cloud.google.com/backend-config` annotations, as shown in the following listing.

Listing 12.10 Referring to a BackendConfig in a Service object in GKE

```yaml
apiVersion: v1
kind: Service
metadata:
  name: kiada
  annotations:
    cloud.google.com/backend-config: '{"default": "kiada-backend-config"}'
spec:
```

You can use the BackendConfig object to configure many things. Since this object is beyond the scope of this book, use `kubectl explain backendconfig.spec` to learn more about it, or see the GKE documentation.

As a quick example of how custom objects are used to configure ingresses, I’ll show you how to configure cookie-based session affinity using the BackendConfig object. You can see the object manifest in the following listing.

Listing 12.11 Using GKE-specific BackendConfig object to configure session affinity

```yaml
apiVersion: cloud.google.com/v1
kind: BackendConfig
metadata:
  name: kiada-backend-config
spec:
  sessionAffinity:
    affinityType: GENERATED_COOKIE
```

In the listing, the session affinity type is set to `GENERATED_COOKIE`. Since this object is referenced in the `kiada` service, whenever a client accesses the service through the ingress, the request is always routed to the same backend pod.

In this and the previous section, you saw two ways to add custom configuration to an Ingress object. Since the method depends on which ingress controller you’re using, see its documentation for more information.