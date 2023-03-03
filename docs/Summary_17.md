# 17.3 Summary
In this chapter, you learned about Jobs and CronJobs. You learned that:

* A Job object is used to run workloads that execute a task to completion instead of running indefinitely.
* Running a task with the Job object ensures that the Pod running the task is rescheduled in the event of a node failure.
* A Job can be configured to repeat the same task several times if you set the completions field. You can specify the number of tasks that are executed in parallel using the parallelism field.
* When a container running a task fails, the failure is handled either at the Pod level by the Kubelet or at the Job level by the Job controller.
* By default, the Pods created by a Job are identical unless you set the Job's completionMode to Indexed. In that case, each Pod gets its own completion index. This index allows each Pod to process only a certain portion of the data.
* You can use a work queue in a Job, but you must provide your own queue and implement work item retrieval in your container.
* Pods running in a Job can communicate with each other, but you need to define a headless Service so they can find each other via DNS.
* If you want to run a Job at a specific time or at regular intervals, you wrap it in a CronJob. In the CronJob you define the schedule in the well-known crontab format.

This brings us to the end of the second part of this book. You now know how to run all kinds of workloads in Kubernetes. In the next part, you’ll learn more about the Kubernetes Control Plane and how it works.
