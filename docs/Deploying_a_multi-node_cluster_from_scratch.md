# 3.1.6   Deploying a multi-node cluster from scratch
Until you get a deeper understanding of Kubernetes, I strongly recommend that you don’t try to install a multi-node cluster from scratch. If you are an experienced systems administrator, you may be able to do it without much pain and suffering, but most people may want to try one of the methods described in the previous sections first. Proper management of Kubernetes clusters is incredibly difficult. The installation alone is a task not to be underestimated.

If you still feel adventurous, you can start with the instructions in Appendix B, which explain how to create VMs with VirtualBox and install Kubernetes using the kubeadm tool. You can also use those instructions to install Kubernetes on your bare-metal machines or in VMs running in the cloud.

Once you’ve successfully deployed one or two clusters using kubeadm, you can then try to deploy it completely manually, by following Kelsey Hightower’s Kubernetes the Hard Way tutorial at github.com/kelseyhightower/Kubernetes-the-hard-way. Though you may run into several problems, figuring out how to solve them can be a great learning experience.

