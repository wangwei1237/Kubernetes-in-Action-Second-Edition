# 2.4 Summary

If you were new to containers before reading this chapter, you should now understand what they are, why we use them, and what features of the Linux kernel make them possible. If you have previously used containers, I hope this chapter has helped to clarify your uncertainties about how containers work, and you now understand that they’re nothing more than regular OS processes that the Linux kernel isolates from other processes.

After reading this chapter, you should know that:

* Containers are regular processes, but isolated from each other and the other processes running in the host OS.
* Containers are much lighter than virtual machines, but because they use the same Linux kernel, they aren’t as isolated as VMs.
* Docker was the first container platform to make containers popular and the first container runtime supported by Kubernetes. Now, others are supported through the Container Runtime Interface (CRI).
* A container image contains the user application and all its dependencies. It is distributed through a container registry and used to create running containers.
* Containers can be downloaded and executed with a single docker run command.
* Docker builds an image from a Dockerfile that contains commands that Docker should execute during the build process. Images consist of layers that can be shared between multiple images. Each layer only needs to be transmitted and stored once.
* Containers are isolated by Linux kernel features called Namespaces, Control groups, Capabilities, seccomp, AppArmor and/or SELinux. Namespaces ensure that a container sees only a part of the resources available on the host, Control groups limit the amount of a resource it can use, while other features strengthen the isolation between containers.

After inspecting the containers on this ship, you’re now ready to raise the anchor and sail into the next chapter, where you’ll learn about running containers with Kubernetes.