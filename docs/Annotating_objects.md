# 10.4 Annotating objects

Adding labels to your objects makes them easier to manage. In some cases, objects must have labels because Kubernetes uses them to identify which objects belong to the same set. But as you learned in this chapter, you can’t just store anything you want in the label value. For example, the maximum length of a label value is only 63 characters, and the value can’t contain whitespace at all.

For this reason, Kubernetes provides a feature similar to labels–object annotations.

## 10.4.1 Introducing object annotations
Like labels, annotations are also key-value pairs, but they don’t store identifying information and can’t be used to filter objects. Unlike labels, an annotation value can be much longer (up to 256 KB at the time of this writing) and can contain any character.

## Understanding annotations added by Kubernetes
Tools like kubectl and the various controllers that run in Kubernetes may add annotations to your objects if the information can’t be stored in one of the object’s fields.

Annotations are often used when new features are introduced to Kubernetes. If a feature requires a change to the Kubernetes API (for example, a new field needs to be added to an object’s schema), that change is usually deferred for a few Kubernetes releases until it’s clear that the change makes sense. After all, changes to any API should always be made with great care, because after you add a field to the API, you can’t just remove it or you’ll break everyone that use the API.

Changing the Kubernetes API requires careful consideration, and each change must first be proven in practice. For this reason, instead of adding new fields to the schema, usually a new object annotation is introduced first. The Kubernetes community is given the opportunity to use the feature in practice. After a few releases, when everyone’s happy with the feature, a new field is introduced and the annotation is deprecated. Then a few releases later, the annotation is removed.

## Adding your own annotations
As with labels, you can add your own annotations to objects. A great use of annotations is to add a description to each pod or other object so that all users of the cluster can quickly see information about an object without having to look it up elsewhere.

For example, storing the name of the person who created the object and their contact information in the object’s annotations can greatly facilitate collaboration between cluster users.

Similarly, you can use annotations to provide more details about the application running in a pod. For example, you can attach the URL of the Git repository, the Git commit hash, the build timestamp, and similar information to your pods.

You can also use annotations to add the information that certain tools need to manage or augment your objects. For example, a particular annotation value set to `true` could signal to the tool whether it should process and modify the object.

## Understanding annotation keys and values
The same rules that apply to label keys also apply to annotations keys. For more information, see section 10.2.3. Annotation values, on the other hand, have no special rules. An annotation value can contain any character and can be up to 256 KB long. It must be a string, but can contain plain text, YAML, JSON, or even a Base64-Encoded value.

# 10.4.2 Adding annotations to objects
Like labels, annotations can be added to existing objects or included in the object manifest file you use to create the object. Let’s look at how to add an annotation to an existing object.

## Setting object annotations
The simplest way to add an annotation to an existing object is to use the kubectl annotate command. Let’s add an annotation to one of the pods. You should still have a pod named kiada-front-end from one of the previous exercises in this chapter. If not, you can use any other pod or object in your current namespace. Run the following command:

```shell
$ kubectl annotate pod kiada-front-end created-by='Marko Luksa <marko.luksa@xyz.com>'
pod/kiada-front-end annotated
```

This command adds the annotation `created-by` with the value `'Marko Luksa <marko.luksa@xyz.com>'` to the `kiada-front-end pod`.

## Specifying annotations in the object manifest
You can also add annotations to your object manifest file before you create the object. The following listing shows an example. You can find the manifest in the pod.pod-with-annotations.yaml file.

```shell
`Listing 10.6 Annotations in an object manifest`
apiVersion: v1
kind: Pod
metadata:
  name: pod-with-annotations
  annotations:
    created-by: Marko Luksa <marko.luksa@xyz.com>
    contact-phone: +1 234 567 890
    managed: 'yes'
    revision: '3'
spec:
  ...
```
{% hint style='info' %}
WARNING

Make sure you enclose the annotation value in quotes if the YAML parser would otherwise treat it as something other than a string. If you don’t, a cryptic error will occur when you apply the manifest. For example, if the annotation value is a number like 123 or a value that could be interpreted as a boolean (true, false, but also words like yes and no), enclose the value in quotes (examples: “123”, “true”, “yes”) to avoid the following error: “unable to decode yaml ... ReadString: expects “ or n, but found t”.
{% endhint %}

Apply the manifest from the previous listing by executing the following command:


```shell
$ kubectl apply -f pod.pod-with-annotations.yaml
```

# 10.4.3 Inspecting an object’s annotations
Unlike labels, the `kubectl get` command does not provide an option to display annotations in the object list. To see the annotations of an object, you should use `kubectl describe` or find the annotation in the object’s YAML or JSON definition.

Viewing an object’s annotations with kubectl describe
To see the annotations of the `pod-with-annotations` pod you created, use `kubectl describe`:

```
$ kubectl describe po pod-with-annotations
Name:         pod-with-annotations
Namespace:    kiada
Priority:     0
Node:         kind-worker/172.18.0.4
Start Time:   Tue, 12 Oct 2021 16:37:50 +0200
Labels:       <none>
Annotations:  contact-phone: +1 234 567 890
              created-by: Marko Luksa <marko.luksa@xyz.com>
              managed: yes
              revision: 3
Status:       Running
 
...
```

## Displaying the object’s annotations in the object’s JSON definition
Alternatively, you can use the `jq` command to extract the annotations from the JSON definition of the pod:

```
$ kubectl get po pod-with-annotations -o json | jq .metadata.annotations
{
  "contact-phone": "+1 234 567 890",
  "created-by": "Marko Luksa <marko.luksa@xyz.com>",
  "kubectl.kubernetes.io/last-applied-configuration": "..."
  "managed": "yes",
  "revision": "3"
}
```

You’ll notice that there’s an additional annotation in the object with the key kubectl.kubernetes.io/last-applied-configuration. It isn’t shown by the kubectl describe command, because it’s only used internally by kubectl and would also make the output too long. In the future, this annotation may become deprecated and then be removed. Don’t worry if you don’t see it when you run the command yourself.

## 10.4.4 Updating and removing annotations
If you want to use the `kubectl annotate` command to change an existing annotation, you must also specify the `--overwrite` option, just as you would when changing an existing object label. For example, to change the annotation `created-by`, the full command is as follows:

```
$ kubectl annotate pod kiada-front-end created-by='Humpty Dumpty' --overwrite
```

To remove an annotation from an object, add the minus sign to the end of the annotation key you want to remove:

```shell
$ kubectl annotate pod kiada-front-end created-by-
```
