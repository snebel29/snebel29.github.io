---
layout: post
title: The kubernetes pause container
description: The kubernetes pause container
---

Have you ever ask yourself what those `/pause` containers are in your kubernetes nodes?

```console
$ docker ps |grep pause
435926c028e6        cfe-pause:5.9.4                             "/pause"                 2 weeks ago         Up 2 weeks                              k8s_POD_kube-state-metrics-67fbc6448-r5nmg_kube-system_a09d2d01-be5f-11e9-b831-fa163ec15d6b_0
6c9672d2ff6e        cfe-pause:5.9.4                             "/pause"                 2 months ago        Up 2 months                             k8s_POD_metrics-server-74b65b96cd-lldk4_kube-system_a826ea4a-98d4-11e9-96f8-fa163ec15d6b_0
2e3e6c871980        cfe-pause:5.9.4                             "/pause"                 2 months ago        Up 2 months
```

A [Pod](https://kubernetes.io/docs/concepts/workloads/pods/pod/) is a group of one or more containers which share resources such as network and storage, in fact they are containers which bound to the same kernel [namespaces](http://man7.org/linux/man-pages/man7/namespaces.7.html) typically `network`, `ipc`, `mount` and `pid` which are leveraged by container runtimes to isolate those processes from the host, and which belong to the same linux [cgroups](https://en.wikipedia.org/wiki/Cgroups).

The pause container set the network namespace and when sharing pid namespace is enable act as PID 1, reaping zombie processes and holding the network in case container restarts.

The image is slim (350KB) and pretty [simple](https://github.com/kubernetes/kubernetes/blob/e176e477195f92919724bd02f81e2b18d4477af7/build/pause/Dockerfile)
```console
$  docker images |grep cfe-pause
cfe-pause                                   5.9.4               2b58359142b0        3 years ago         350kB
```

```docker
FROM scratch
ARG ARCH
ADD bin/pause-${ARCH} /pause
ENTRYPOINT ["/pause"]
```
The [pause binary](https://github.com/kubernetes/kubernetes/blob/e176e477195f92919724bd02f81e2b18d4477af7/build/pause/pause.c#L32-L68) is also very simple and do the following

Set handlers to `SIGINT`, `SIGTERM` and `SIGCHLD` and then go to Sleep, that way 

1. Will hold the namespaces for created container, either new or restarted
2. Will gracefully exit when the process is terminated
3. Will reap zombie processed from other containers if shared pid namespace is enabled, since pause process will be PID 1 into the Pod.

You can see the world through the eyes of a container member of existing Pods using the [toolbox](https://github.com/snebel29/toolbox) docker image, for example to join to `kube-dns-67d4f46b79-f4kzb` pod

```console
my_pod=kube-dns-67d4f46b79-f4kzb
pod_id=$(docker ps | grep "k8s_POD_$my_pod" | awk '{print $1}')
docker run -it \
	--rm \
	--volume $(pwd):/data \
	--pid container:${pod_id} \
	--network container:${pod_id} \
	snebel29/toolbox 
```

Now you can interact and troubleshoot your pod from the inside using your favourite tools, you could even install new ones which would be deleted upon exiting the pod!
