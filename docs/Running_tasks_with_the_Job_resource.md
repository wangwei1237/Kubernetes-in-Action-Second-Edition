# 17.1 Running tasks with the Job resource
Before you create your first Pod via the Job resource, let’s think about the Pods in the kiada Namespace. They’re all meant to run continuously. When a container in one of these pods terminates, it’s automatically restarted. When the Pod is deleted, it’s recreated by the controller that created the original Pod. For example, if you delete one of the kiada pods, it’s quickly recreated by the Deployment controller because the replicas field in the kiada Deployment specifies that three Pods should always exist.

Now consider a Pod whose job is to initialize the MongoDB database. You don’t want it to run continuously; you want it to perform one task and then exit. Although you want the Pod’s containers to restart if they fail, you don’t want them to restart when they finish successfully. You also don’t want a new Pod to be created after you delete the Pod that completed its task.

You may recall that you already created such a Pod in chapter 15, namely the quiz-data-importer Pod. It was configured with the OnFailure restart policy to ensure that the container would restart only if it failed. When the container terminated successfully, the Pod was finished, and you could delete it. Since you created this Pod directly and not through a Deployment, StatefulSet or DaemonSet, it wasn’t recreated. So, what’s wrong with this approach and why would you create the Pod via a Job instead?

To answer this question, consider what happens if someone accidentally deletes the Pod prematurely or if the Node running the Pod fails. In these cases, Kubernetes wouldn’t automatically recreate the Pod. You’d have to do that yourself. And you’d have to watch that Pod from creation to completion. That might be fine for a Pod that completes its task in seconds, but you probably don’t want to be stuck watching a Pod for hours. So, it’s better to create a Job object and let Kubernetes do the rest.

## 17.1.1 Introducing the Job resource

The Job resource resembles a Deployment in that it creates one or more Pods, but instead of ensuring that those Pods run indefinitely, it only ensures that a certain number of them complete successfully.

As you can see in the following figure, the simplest Job runs a single Pod to completion, whereas more complex Jobs run multiple Pods, either sequentially or concurrently. When all containers in a Pod terminate with success, the Pod is considered completed. When all the Pods have completed, the Job itself is also completed.

Figure 17.1 Three different Job examples. Each Job is completed once its Pods have completed successfully.

![](../images/17.1.png)

As you might expect, a Job resource defines a Pod template and the number of Pods that must be successfully completed. It also defines the number of Pods that may run in parallel.

> NOTE  
>   
> Unlike Deployments and other resources that contain a Pod template, you can’t modify the template in a Job object after creating the object.

Let’s look at what the simplest Job object looks like.

**Defining a Job resource**

In this section, you take the *quiz-data-importer* Pod from chapter 15 and turn it into a Job. This Pod imports the data into the Quiz MongoDB database. You may recall that before running this Pod, you had to initiate the MongoDB replica set by issuing a command in one of the *quiz* Pods. You can do that in this Job as well, using an init container. The Job and the Pod it creates are visualized in the following figure.

Figure 17.2 An overview of the quiz-init Job

![](../images/17.2.png)

The following listing shows the Job manifest, which you can find in the file *job.quiz-init.yaml*.

> NOTE  
>   
> The manifest file also contains a ConfigMap in which the quiz questions are stored but this ConfigMap is not shown in the listing.

Listing 17.1 A Job manifest for running a single task
```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: quiz-init
  labels:
    app: quiz
    task: init
spec:
  template:
    metadata:
      labels:
        app: quiz
        task: init
    spec:
      restartPolicy: OnFailure
      initContainers:
      - name: init
        image: mongo:5
        command:
        - sh
        - -c
        - |
          mongosh mongodb://quiz-0.quiz-pods.kiada.svc.cluster.local \
    --quiet --file /dev/stdin <<EOF

          # MongoDB code that initializes the replica set
          # Refer to the job.quiz-init.yaml file to see the actual code

          EOF
      containers:
      - name: import
        image: mongo:5
        command:
        - mongoimport
        - mongodb+srv://quiz-pods.kiada.svc.cluster.local/kiada?tls=false
        - --collection
        - questions
        - --file
        - /questions.json
        - --drop
        volumeMounts:
        - name: quiz-data
          mountPath: /questions.json
          subPath: questions.json
          readOnly: true
      volumes:
      - name: quiz-data
        configMap:
          name: quiz-data
```

The manifest in the listing defines a Job object that runs a single Pod to completion. Jobs belong to the batch API group, and you’re using API version *v1* to define the object. The Pod that this Job creates consists of two containers that execute in sequence, as one is an init and the other a normal container. The init container makes sure that the MongoDB replica set is initialized, then the main container imports the quiz questions from the *quiz-data* ConfigMap that’s mounted into the container through a volume.

The Pod’s *restartPolicy* is set to *OnFailure*. A Pod defined within a Job can’t use the default policy of *Always*, as that would prevent the Pod from completing.

> NOTE  
> 
> In a Job’s pod *template*, you must explicitly set the restart policy to either *OnFailure* or *Never*.

You’ll notice that unlike Deployments, the Job manifest in the listing doesn’t define a *selector*. While you can specify it, you don’t have to, as Kubernetes sets it automatically. The Pod template in the listing does contain two labels, but they’re there only for your convenience.

**Running a Job**

The Job controller creates the Pods immediately after you create the Job object. To run the *quiz-init* Job, apply the *job.quiz-init.yaml* manifest with *kubectl apply*.

**Displaying a brief Job status**

To get a brief overview of the Job’s status, list the Jobs in the current Namespace as follows:

```shell
$ kubectl get jobs
NAME        COMPLETIONS   DURATION   AGE
quiz-init   0/1           3s         3s
```

The *COMPLETIONS* column indicates how many times the Job has run and how many times it’s configured to complete. The DURATION column shows how long the Job has been running. Since the task the *quiz-init* Job performs is relatively short, its status should change within a few seconds. List the Jobs again to confirm this:

```shell
$ kubectl get jobs
NAME        COMPLETIONS   DURATION   AGE
quiz-init   1/1           6s         42s
```

The output shows that the Job is now complete, which took 6 seconds.


**Displaying the detailed Job status**

To see more details about the Job, use the kubectl describe command as follows:

```shell
$ kubectl describe job quiz-init
Name:             quiz-init
Namespace:        kiada
Selector:         controller-uid=98f0fe52-12ec-4c76-a185-4ccee9bae1ef
Labels:           app=quiz
                  task=init
Annotations:      batch.kubernetes.io/job-tracking:
Parallelism:      1
Completions:      1
Completion Mode:  NonIndexed
Start Time:       Sun, 02 Oct 2022 12:17:59 +0200
Completed At:     Sun, 02 Oct 2022 12:18:05 +0200
Duration:         6s
Pods Statuses:    0 Active / 1 Succeeded / 0 Failed
Pod Template:
  Labels:  app=quiz
           controller-uid=98f0fe52-12ec-4c76-a185-4ccee9bae1ef
           job-name=quiz-init
           task=init
  Init Containers:
   init: ...
  Containers:
   import: ...
  Volumes:
   quiz-data: ...
Events:
  Type    Reason            Age    From            Message
  ----    ------            ----   ----            -------
  Normal  SuccessfulCreate  7m33s  job-controller  Created pod: quiz-init-xpl8d
  Normal  Completed         7m27s  job-controller  Job completed
```

In addition to the Job *name*, *namespace*, *labels*, *annotations*, and other properties, the output of the *kubectl describe* command also shows the selector that was automatically assigned. The *controller-uid* label used in the selector was also automatically added to the Job’s Pod template. The *job-name* label was also added to the template. As you’ll see in the next section, you can easily use this label to list the Pods that belong to a particular Job.

At the end of the *kubectl describe* output, you see the *Events* associated with this Job object. Only two events were generated for this Job: the creation of the Pod and the successful completion of the Job.

**Examining the Pods that belong to a Job**

To list the Pods created for a particular Job, you can use the job-name label that’s automatically added to those Pods. To list the Pods of the quiz-init job, run the following command:
```shell
$ kubectl get pods -l job-name=quiz-init
NAME              READY   STATUS      RESTARTS   AGE
quiz-init-xpl8d   0/1     Completed   0          25m
```

The pod shown in the output has finished its task. The Job controller doesn’t delete the Pod, so you can see its status and view its logs.

**Examining the logs of a Job Pod**

The fastest way to see the logs of a Job is to pass the Job name instead of the Pod name to the kubectl logs command. To see the logs of the *quiz-init* Job, you could do something like the following:
```shell
$ kubectl logs job/quiz-init --all-containers --prefix
[pod/quiz-init-xpl8d/init] Replica set initialized successfully!
[pod/quiz-init-xpl8d/import] 2022-10-02T10:51:01.967+0000  connected to: ...
[pod/quiz-init-xpl8d/import] 2022-10-02T10:51:01.969+0000  dropping: kiada.questions
[pod/quiz-init-xpl8d/import] 2022-10-02T10:51:03.811+0000  6 document(s) imported...
```

The *--all-containers* option tells *kubectl* to print the logs of all the Pod’s containers, and the *--prefix* option ensures that each line is prefixed with the source, that is, the pod and container names.

The output contains both the init and the *import* container logs. These logs indicate that the MongoDB replica set has been successfully initialized and that the question database has been populated with data.

**Suspending active Jobs and creating Jobs in a suspended state**

When you created the *quiz-init* Job, the Job controller created the Pod as soon as you created the Job object. However, you can also create Jobs in a suspended state. Let’s try this out by creating another Job. As you can see in the following listing, you suspend it by setting the suspend field to true. You can find this manifest in the file *job.demo-suspend.yaml*.

Listing 17.2 The manifest of a suspended Job
```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: demo-suspend
spec:
  suspend: true
  template:
    spec:
      restartPolicy: OnFailure
      containers:
      - name: demo
        image: busybox
        command:
        - sleep
        - "60"
```

Apply the manifest in the listing to create the Job. List the Pods as follows to confirm that none have been created yet:
```shell
$ kubectl get po -l job-name=demo-suspend
No resources found in kiada namespace.
```

The Job controller generates an Event indicating the suspension of the Job. You can see it when you run *kubectl get events* or when you describe the Job with *kubectl describe*:

```shell
$ kubectl describe job demo-suspend
...
Events:
  Type    Reason     Age    From            Message
  ----    ------     ----   ----            -------
  Normal  Suspended  3m37s  job-controller  Job suspended
```

When you’re ready to run the Job, you unsuspend it by patching the object as follows:
```shell
$ kubectl patch job demo-suspend -p '{"spec":{"suspend": false}}'
job.batch/demo-suspend patched
```

The Job controller creates the Pod and generates an Event indicating that the Job has resumed.

You can also suspend a running Job, whether you created it in a suspended state or not. To suspend a Job, set *suspend* to *true* with the following *kubectl patch* command:
```shell
$ kubectl patch job demo-suspend -p '{"spec":{"suspend": true}}'
job.batch/demo-suspend patched
```

The Job controller immediately deletes the Pod associated with the Job and generates an Event indicating that the Job has been suspended. The Pod’s containers are shut down gracefully, as they are every time you delete a Pod, regardless of how it was created. You can resume the Job at your discretion by resetting the *suspend* field to *false*.

**Deleting Jobs and their Pods**

You can delete a Job any time. Regardless of whether its Pods are still running or not, they’re deleted in the same way as when you delete a Deployment, StatefulSet, or DaemonSet.

You don’t need the *quiz-init* Job anymore, so delete it as follows:

```shell
$ kubectl delete job quiz-init
job.batch "quiz-init" deleted
```

Confirm that the Pod has also been deleted by listing the Pods as follows:
```shell
$ kubectl get po -l job-name=quiz-init
No resources found in kiada namespace.
```

You may recall that Pods are deleted by the garbage collector because they’re orphaned when their owner, in this case the Job object named *quiz-init*, is deleted. If you want to delete only the Job, but keep the Pods, you delete the Job with the *--cascade=orphan* option. You can try this method with the *demo-suspend* Job as follows:
```shell
$ kubectl delete job demo-suspend --cascade=orphan
 
job.batch "demo-suspend" deleted
```

If you now list Pods, you’ll see that the Pod still exists. Since it’s now a standalone Pod, it’s up to you to delete it when you no longer need it.

**Automatically deleting Jobs**

By default, you must delete Job objects manually. However, you can flag the Job for automatic deletion by setting the *ttlSecondsAfterFinished* field in the Job’s *spec*. As the name implies, this field specifies how long the Job and its Pods are kept after the Job is finished.

To see this setting in action, try creating the Job in the *job.demo-ttl.yaml* manifest. The Job will run a single Pod that will complete successfully after 20 seconds. Since ttlSecondsAfterFinished is set to *10*, the Job and its Pod are deleted ten seconds later.


> WARNING
> 
> If you set the ttlSecondsAfterFinished field in a Job, the Job and its pods are deleted whether the Job completes successfully or not. If this happens before you can check the logs of the failed Pods, it’s hard to determine what caused the Job to fail.

## 17.1.2 Running a task multiple times

In the previous section, you learned how to execute a task once. However, you can also configure the Job to execute the same task several times, either in parallel or sequentially. This may be necessary because the container running the task can only process a single item, so you need to run the container multiple times to process the entire input, or you may simply want to run the processing on multiple cluster nodes to improve performance.

You’ll now create a Job that inserts fake responses into the Quiz database, simulating a large number of users. Instead of having only one Pod that inserts data into the database, as in the previous example, you’ll configure the Job to create five such Pods. However, instead of running all five Pods simultaneously, you’ll configure the Job to run at most two Pods at a time. The following listing shows the Job manifest. You can find it in the file *job.generate-responses.yaml*.

Listing 17.3 A Job for running a task multiple times
```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: generate-responses
  labels:
    app: quiz
spec:
  completions: 5
  parallelism: 2
  template:
    metadata:
      labels:
        app: quiz
    spec:
      restartPolicy: OnFailure
      containers:
      - name: mongo
        image: mongo:5
        command:
        ...
```

In addition to the Pod template, the Job manifest in the listing defines two new properties, *completions* and *parallelism*., which are explained next.

**Understanding Job completions and parallelism**

The *completions* field specifies the number of Pods that must be successfully completed for this Job to be complete. The *parallelism* field specifies how many of these Pods may run in parallel. There is no upper limit to these values, but your cluster may only be able to run so many Pods in parallel.

You can set neither of these fields, one or the other, or both. If you don’t set either field, both values are set to one by default. If you set only *completions*, this is the number of Pods that run one after the other. If you set only *parallelism*, this is the number of Pods that run, but only one must complete successfully for the Job to be complete.

If you set *parallelism* higher than completions, the Job controller creates only as many Pods as you specified in the *completions* field.

If *parallelism* is lower than *completions*, the Job controller runs at most *parallelism* Pods in parallel but creates additional Pods when those first Pods complete. It keeps creating new Pods until the number of successfully completed Pods matches *completions*. The following figure shows what happens when *completions* is 5 and *parallelism* is 2.

Figure 17.3 Running a parallel Job with completion=5 and parallelism=2

![](../images/17.3.png)

As shown in the figure, the Job controller first creates two Pods and waits until one of them completes. In the figure, Pod 2 is the first to finish. The controller immediately creates the next Pod (Pod 3), bringing the number of running Pods back to two. The controller repeats this process until five Pods complete successfully.

The following table explains the behavior for different examples of *completions* and *parallelism*.

Table 17.1 Completions and parallelism combinations
| Completions | Parallelism | Job behavior | 
| --- | --- | --- |
| Not set | Not set | A single Pod is created. Same as when completions and parallelism is 1.
| 1 | 1 | A single Pod is created. If the Pod completes successfully, the Job is complete. If the Pod is deleted before completing, it’s replaced by a new Pod. |
| 2 | 5 | Only three Pods are created. The same as if parallelism was 2. |
| 5 | 2 | Two Pods are created initially. When one of them completes, the third Pod is created. There are again two Pods running. When one of the two completes, the fourth Pod is created. There are again two Pods running. When another one completes, the fifth and last Pod is created. |
| 5 | 5 | Five Pods run simultaneously. If one of them is deleted before it completes, a replacement is created. The Job is complete when five Pods complete successfully. |
| 5 | Not set | Five Pods are created sequentially. A new Pod is created only when the previous Pod completes (or fails). |
| Not set | 5 | Five Pods are created simultaneously, but only one needs to complete successfully for the Job to complete. |

In the *generate-responses* Job that you’re about to create, the number of *completions* is set to 5 and *parallelism* is set to 2, so at most two Pods will run in parallel. The Job isn’t complete until five Pods complete successfully. The total number of Pods may end up being higher if some of the Pods fail. More on this in the next section.

**Running the Job**
Use *kubectl apply* to create the Job by applying the manifest file *job.generate-responses.yaml*. List the Pods while running the Job as follows:

```shell
$ kubectl get po -l job-name=generate-responses
NAME                       READY   STATUS      RESTARTS      AGE
generate-responses-7kqw4   1/1     Running     2 (20s ago)   27s
generate-responses-98mh8   0/1     Completed   0             27s
generate-responses-tbgns   1/1     Running     0             3s
```

List the Pods several times to observe the number Pods whose *STATUS* is shown as *Running* or *Completed*. As you can see, at any given time, at most two Pods run simultaneously. After some time, the Job completes. You can see this by displaying the Job status with the *kubectl get* command as follows:


```shell
$ kubectl get job generate-responses
NAME                 COMPLETIONS   DURATION  AGE
generate-responses   5/5           110s      115s
```

The *COMPLETIONS* column shows that this Job completed five out of the desired five times, which took 110 seconds. If you list the Pods again, you should see five completed Pods, as follows:

```shell
$ kubectl get po -l job-name=generate-responses
NAME                       READY   STATUS      RESTARTS   AGE
generate-responses-5xtlk   0/1     Completed   0          82s
generate-responses-7kqw4   0/1     Completed   3          2m46s
generate-responses-98mh8   0/1     Completed   0          2m46s
generate-responses-tbgns   0/1     Completed   1          2m22s
generate-responses-vbvq8   0/1     Completed   1          111s
```

As indicated in the Job status earlier, you should see five *Completed* Pods. However, if you look closely at the *RESTARTS* column, you’ll notice that some of these Pods had to be restarted. The reason for this is that I hard-coded a 25% failure rate into the code running in those Pods. I did this to show what happens when an error occurs.



```shell

```


```shell

```



![](../images/17.1.png)

```shell

```
