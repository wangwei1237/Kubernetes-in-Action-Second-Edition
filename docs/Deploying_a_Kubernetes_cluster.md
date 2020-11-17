# 3.1      Deploying a Kubernetes cluster
Setting up a full-fledged, multi-node Kubernetes cluster isn’t a simple task, especially if you’re not familiar with Linux and network administration. A proper Kubernetes installation spans multiple physical or virtual machines and requires proper network setup to allow all containers in the cluster to communicate with each other.

You can install Kubernetes on your laptop computer, on your organization’s infrastructure, or on virtual machines provided by cloud providers (Google Compute Engine, Amazon EC2, Microsoft Azure, and so on). Alternatively, most cloud providers now offer managed Kubernetes services, saving you from the hassle of installation and management. Here’s a short overview of what the largest cloud providers offer:

* Google offers GKE - Google Kubernetes Engine,
* Amazon has EKS - Amazon Elastic Kubernetes Service,
* Microsoft has AKS – Azure Kubernetes Service,
* IBM has IBM Cloud Kubernetes Service,
* Alibaba provides the Alibaba Cloud Container Service.

Installing and managing Kubernetes is much more difficult than just using it, especially until you’re intimately familiar with its architecture and operation. For this reason, we’ll start with the easiest ways to obtain a working Kubernetes cluster. You’ll learn several ways to run a single-node Kubernetes cluster on your local computer and how to use a hosted cluster running on Google Kubernetes Engine (GKE).

A third option, which involves installing a cluster using the kubeadm tool, is explained in Appendix B. The tutorial there will show you how to set up a three-node Kubernetes cluster using virtual machines. But you may want to try that only after you’ve become familiar with using Kubernetes. Many other options also exist, but they are beyond the scope of this book. Refer to the kubernetes.io website to learn more.

If you’ve been granted access to an existing cluster deployed by someone else, you can skip this section and go on to section 3.2 where you’ll learn how to interact with Kubernetes clusters.