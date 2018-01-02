---
layout: post
title: Why Kubernetes Pod Quality of Service matters
---
I (painfully) discovered a few weeks ago why specifying how much CPU and RAM a container needs was so important on Kubernetes. In this post, I will explain why ;)

<h3>1. The basis</h3>

Each container in a Kubernetes pod can specifify the following:
- CPU request
- Memory request
- CPU limit
- Memory limit

The sum of all the requests and limits of all the containers in a pod defines the pod requests/limits.

When a pod is created, the Kubernetes scheduler selects a node which has enough CPU and memory according to the pod requirements (its requests). On the other hand, a container using more CPU or memory than its limits becomes a candidate for termination, and can therefore be terminated at any time.

<h3>2. The Quality of Service</h3>

This is where the QoS classes really matter:
- a pod with equal limits and requests set for each container will be classified as <b>Guaranteed</b>
- a pod with different limits and requests set for each container will be classified as <b>Burstable</b>
- a pod without limits and requests set for each container will be classified as <b>Best-Effort</b>.

As you probably guessed it, Guaranteed > Burstable > Best-Effort. A guaranteed pod is garanteed to not be killed until it exceed its limits (or the cluster is under memory pressure and there are no lower priority pods that can be killed). This is definitely what you want for you most important pods.

<h3>3. What happened in my case</h3>

At Hunter, we moved from OVH to DigitalOcean, and deployed for this occasion our Kubernetes cluster on instances with fewer RAM than our previous nodes. For this reason, our new cluster was under a constant memory pressure, whereas the old one always had plenty of memory left.

We had installed Flannel on our brand new cluster using [the official guide](https://kubernetes.io/docs/setup/independent/create-cluster-kubeadm/#pod-network){:target="_blank"}, that, as you may see, doesn't set requests and limits for the [<i>kube-flannel container</i>](https://raw.githubusercontent.com/coreos/flannel/v0.9.1/Documentation/kube-flannel.yml){:target="_blank"}. <b>The result is devastating if you run several pods with requests and limits set.</b>

As explained sooner, the <i>kube-flannel-ds</i> pod will be classified as Best-Effort. For this reason, it will be killed by Kubernetes if needed, making the Kubernetes internal network broken. Pods won't be able to ping each other, Services won't be reachable from pods, etc., obviously because the Pod network won't update the node routing table...

This had never happened on the old cluster (with the same setup) because it had never been under such a memory pressure.

<h3>4. How to prevent this</h3>

You should <b>always</b> set resources requests and limits to your pods, unless you agree to let Kubernetes kill them if the cluster is under memory pressure.

You should also spread the word every time you follow a tutorial that do no set requests or limit on containers (even if [I haven't been really successful](https://github.com/kubernetes/website/issues/6682){:target="_blank"} for my part ;)).
