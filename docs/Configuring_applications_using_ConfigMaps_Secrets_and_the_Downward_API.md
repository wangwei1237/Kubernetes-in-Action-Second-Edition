This chapter covers

* Setting the command and arguments for the container's main process
* Setting environment variables
* Storing configuration in config maps
* Storing sensitive information in secrets
* Using the Downward API to expose pod metadata to the application
* Using configMap, secret, downwardAPI and projected volumes

You’ve now learned how to use Kubernetes to run an application process and attach file volumes to it. In this chapter, you’ll learn how to configure the application - either in the pod manifest itself, or by referencing other API objects within it. You’ll also learn how to inject information about the pod itself into the application running inside it.

{% hint style='info' %}
NOTE

You’ll find the code files for this chapter at https://github.com/luksa/kubernetes-in-action-2nd-edition/tree/master/Chapter09
{% endhint %}