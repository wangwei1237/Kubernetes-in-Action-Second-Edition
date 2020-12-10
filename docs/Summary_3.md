# 3.4 Summary

In this hands-on chapter, you’ve learned:

* Virtually all cloud providers offer a managed Kubernetes option. They take on the burden of maintaining your Kubernetes cluster, while you just use its API to deploy your applications.
* You can also install Kubernetes in the cloud yourself, but this has often proven not to be the best idea until you master all aspects of managing Kubernetes.
* You can install Kubernetes locally, even on your laptop, using tools such as Docker Desktop or Minikube, which run Kubernetes in a Linux VM, or kind, which runs the master and worker nodes as Docker containers and the application containers inside those containers.
* Kubectl, the command-line tool, is the usual way you interact with Kubernetes. A web-based dashboard also exists but is not as stable and up to date as the CLI tool.
* To work faster with kubectl, it is useful to define a short alias for it and enable shell completion.
* An application can be deployed using `kubectl create deployment`. It can then be exposed to clients by running `kubectl expose deployment`. Horizontally scaling the application is trivial: `kubectl scale deployment` instructs Kubernetes to add new replicas or removes existing ones to reach the number of replicas you specify.
* The basic unit of deployment is not a container, but a pod, which can contain one or more related containers.
* Deployments, Services, Pods and Nodes are Kubernetes objects/resources. You can list them with `kubectl get` and inspect them with `kubectl describe`.
* The Deployment object deploys the desired number of Pods, while the Service object makes them accessible under a single, stable IP address.
* Each service provides internal load balancing in the cluster, but if you set the type of service to `LoadBalancer`, Kubernetes will ask the cloud infrastructure it runs in for an additional load balancer to make your application available at a publicly accessible address.
  
You’ve now completed your first guided tour around the bay. Now it’s time to start learning the ropes, so that you’ll be able to sail independently. The next part of the book focuses on the different Kubernetes objects and how/when to use them. You’ll start with the most important one – the Pod.