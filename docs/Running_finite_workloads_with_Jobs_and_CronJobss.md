# 17 Running finite workloads with Jobs and CronJobs

This chapter covers
* Running finite tasks with Jobs
* Handling Job failures
* Parameterizing Pods created through a Job
* Processing items in a work queue
* Enabling communication between a Job’s Pods
* Using CronJobs to run Jobs at a specific time or at regular intervals

As you learned in the previous chapters, a Pod created via a Deployment, StatefulSet, or DaemonSet, runs continuously. When the process running in one of the Pod’s containers terminates, the Kubelet restarts the container. The Pod never stops on its own, but only when you delete the Pod object. Although this is ideal for running web servers, databases, system services, and similar workloads, it’s not suitable for finite workloads that only need to perform a single task.

A finite workload doesn’t run continuously, but lets a task run to completion. In Kubernetes, you run this type of workload using the Job resource. However, a Job always runs its Pods immediately, so you can’t use it for scheduling tasks. For that, you need to wrap the Job in a CronJob object. This allows you to schedule the task to run at a specific time in the future or at regular intervals.

In this chapter you’ll learn everything about Jobs and CronJobs. Before you begin, create the kiada Namespace, change to the Chapter17/ directory, and apply all the manifests in the SETUP/ directory by running the following commands:

```shell
$ kubectl create ns kiada
$ kubectl config set-context --current --namespace kiada
$ kubectl apply -f SETUP -R
```
