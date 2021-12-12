# 10 Organizing objects using Namespaces, labels, and selectors

{% hint style='info' %}
This chapter covers
* Using namespaces to split a physical cluster into virtual clusters
* Organizing objects using labels
* Using label selectors to perform operations on subsets of objects
* Using label selectors to schedule pods onto specific nodes
* Using field selectors to filter objects based on their properties
* Annotating objects with additional non-identifying information
{% endhint %}

A Kubernetes cluster is usually used by many teams. How should these teams deploy objects to the same cluster and organize them so that one team doesn’t accidentally modify the objects created by other teams?

And how can a large team deploying hundreds of microservices organize them so that each team member, even if new to the team, can quickly see where each object belongs and what its role in the system is? For example, to which application does a config map or a secret belong.

These are two different problems. Kubernetes solves the first with the Namespace resource, and the other with object labels. In this chapter, you will learn about both.