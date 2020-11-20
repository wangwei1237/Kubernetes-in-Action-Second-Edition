# Mounting storage volumes into the Pod’s containers
This chapter covers

* Persisting files across container restarts
* Sharing files between containers of the same pod
* Sharing files between pods
* Attaching network storage to pods
* Accessing the host node filesystem from within a pod

The previous two chapters focused on the pod’s containers, but they are only half of what a pod typically contains. They are typically accompanied by storage volumes that allow a pod’s containers to store data for the lifetime of the pod or beyond, or to share files with the other containers of the pod. This is the focus of this chapter.
