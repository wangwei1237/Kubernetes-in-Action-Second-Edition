# Introducing Kubernetes into your organization
To close this chapter, let’s see what options are available to you if you decide to introduce Kubernetes in your own IT environment.

## Running Kubernetes on-premises and in the cloud
If you want to run your applications on Kubernetes, you have to decide whether you want to run them locally, in your organization’s own infrastructure (on-premises) or with one of the major cloud providers, or perhaps both - in a hybrid cloud solution.

### Running Kubernetes on-premises
Running Kubernetes on your own infrastructure may be your only option if regulations require you to run applications on site. This usually means that you’ll have to manage Kubernetes yourself, but we’ll come to that later.

Kubernetes can run directly on your bare-metal machines or in virtual machines running in your data center. In either case, you won’t be able to scale your cluster as easily as when you run it in virtual machines provided by a cloud provider.

### Deploying Kubernetes in the cloud
If you have no on-premises infrastructure, you have no choice but to run Kubernetes in the cloud. This has the advantage that you can scale your cluster at any time at short notice if required. As mentioned earlier, Kubernetes itself can ask the cloud provider to provision additional virtual machines when the current size of the cluster is no longer sufficient to run all the applications you want to deploy.

When the number of workloads decreases and some worker nodes are left without running workloads, Kubernetes can ask the cloud provider to destroy the virtual machines of these nodes to reduce your operational costs. This elasticity of the cluster is certainly one of the main benefits of running Kubernetes in the cloud.

### Using a hybrid cloud solution
A more complex option is to run Kubernetes on-premises, but also allow it to spill over into the cloud. It’s possible to configure Kubernetes to provision additional nodes in the cloud if you exceed the capacity of your own data center. This way, you get the best of both worlds. Most of the time, your applications run locally without the cost of virtual machine rental, but in short periods of peak load that may occur only a few times a year, your applications can handle the extra load by using the additional resources in the cloud.

If your use-case requires it, you can also run a Kubernetes cluster across multiple cloud providers or a combination of any of the options mentioned. This can be done using a single control plane or one control plane in each location.

## To manage or not to manage Kubernetes yourself
If you are considering introducing Kubernetes in your organization, the most important question you need to answer is whether you’ll manage Kubernetes yourself or use a Kubernetes-as-a-Service type offering where someone else manages it for you.

### Managing Kubernetes yourself
If you already run applications on-premises and have enough hardware to run a production-ready Kubernetes cluster, your first instinct is probably to deploy and manage it yourself. If you ask anyone in the Kubernetes community if this is a good idea, you’ll usually get a very definite “no”.

Figure 1.14 was a very simplified representation of what happens in a Kubernetes cluster when you deploy an application. Even that figure should have scared you. Kubernetes brings with it an enormous amount of additional complexity. Anyone who wants to run a Kubernetes cluster must be intimately familiar with its inner workings.

The management of production-ready Kubernetes clusters is a multi-billion-dollar industry. Before you decide to manage one yourself, it’s essential that you consult with engineers who have already done it to learn about the issues most teams run into. If you don’t, you may be setting yourself up for failure. On the other hand, trying out Kubernetes for non-production use-cases or using a managed Kubernetes cluster is much less problematic.

### Using a managed Kubernetes cluster in the cloud
Using Kubernetes is ten times easier than managing it. Most major cloud providers now offer Kubernetes-as-a-Service. They take care of managing Kubernetes and its components while you simply use the Kubernetes API like any of the other APIs the cloud provider offers.

The top managed Kubernetes offerings include the following:

* Google Kubernetes Engine (GKE)
* Azure Kubernetes Service (AKS)
* Amazon Elastic Kubernetes Service (EKS)
* IBM Cloud Kubernetes Service
* Red Hat OpenShift Online and Dedicated
* VMware Cloud PKS
* Alibaba Cloud Container Service for Kubernetes (ACK)

The first half of this book focuses on just using Kubernetes. You’ll run the exercises in a local development cluster and on a managed GKE cluster, as I find it’s the easiest to use and offers the best user experience. The second part of the book gives you a solid foundation for managing Kubernetes, but to truly master it, you’ll need to gain additional experience.

## Using vanilla or extended Kubernetes
The final question is whether to use a vanilla open-source version of Kubernetes or an extended, enterprise-quality Kubernetes product.

### Using a vanilla version of Kubernetes
The open-source version of Kubernetes is maintained by the community and represents the cutting edge of Kubernetes development. This also means that it may not be as stable as the other options. It may also lack good security defaults. Deploying the vanilla version requires a lot of fine tuning to set everything up for production use.

### Using enterprise-grade Kubernetes distributions
A better option for using Kubernetes in production is to use an enterprise-quality Kubernetes distribution such as OpenShift or Rancher. In addition to the increased security and performance provided by better defaults, they offer additional object types in addition to those provided in the upstream Kubernetes API. For example, vanilla Kubernetes does not contain object types that represent cluster users, whereas commercial distributions do. They also provide additional software tools for deploying and managing well-known third-party applications on Kubernetes.

Of course, extending and hardening Kubernetes takes time, so these commercial Kubernetes distributions usually lag one or two versions behind the upstream version of Kubernetes. It’s not as bad as it sounds. The benefits usually outweigh the disadvantages.

## Should you even use Kubernetes?
I hope this chapter has made you excited about Kubernetes and you can’t wait to squeeze it into your IT stack. But to close this chapter properly, we need to say a word or two about when introducing Kubernetes is not a good idea.

### Do your workloads require automated management?
The first thing you need to be honest about is whether you need to automate the management of your applications at all. If your application is a large monolith, you definitely don’t need Kubernetes.

Even if you deploy microservices, using Kubernetes may not be the best option, especially if the number of your microservices is very small. It’s difficult to provide an exact number when the scales tip over, since other factors also influence the decision. But if your system consists of less than five microservices, throwing Kubernetes into the mix is probably not a good idea. If your system has more than twenty microservices, you will most likely benefit from the integration of Kubernetes. If the number of your microservices falls somewhere in between, other factors, such as the ones described next, should be considered.

### Can you afford to invest your engineers’ time into learning Kubernetes?
Kubernetes is designed to allow applications to run without them knowing that they are running in Kubernetes. While the applications themselves don’t need to be modified to run in Kubernetes, development engineers will inevitably spend a lot of time learning how to use Kubernetes, even though the operators are the only ones that actually need that knowledge.

It would be hard to tell your teams that you’re switching to Kubernetes and expect only the operations team to start exploring it. Developers like shiny new things. At the time of writing, Kubernetes is still a very shiny thing.

### Are you prepared for increased costs in the interim?
While Kubernetes reduces long-term operational costs, introducing Kubernetes in your organization initially involves increased costs for training, hiring new engineers, building and purchasing new tools and possibly additional hardware. Kubernetes requires additional computing resources in addition to the resources that the applications use.

### Don’t believe the hype
Although Kubernetes has been around for several years at the time of writing this book, I can’t say that the hype phase is over. The initial excitement has just begun to calm down, but many engineers may still be unable to make rational decisions about whether the integration of Kubernetes is as necessary as it seems.