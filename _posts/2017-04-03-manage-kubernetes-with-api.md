---
layout: post
title: Manage your Kubernetes deployments with the Kube API
---
At [Hunter](https://hunter.io), we recently decided to move parts of our containers management from Cloud66 to Kubernetes. We wanted to have more control over the resources requested by our apps, and also speed up the deployment process which was really slow at C66.

Kubernetes comes with a great CLI called `kubectl` to manage and run commands into your cluster. Thing is, it's not really convenient to install and use it inside build machines such as a CircleCI machine. [Some scripts](https://github.com/kubernetes/contrib/tree/master/continuousdelivery) have been created by the community to help you but my own experience shows that's not a really smooth solution. And things get worse if you want to make a guy such as [Hubot](https://hubot.github.com/) (Node.js-based bot built by GitHub) manage your deployments.

Under the hood, `kubectl` uses the Kubernetes API so why not use it directly through plain HTTP calls? This is what I'll be presenting in this article, to accomplish two tasks: update your apps to a newer version and scale your apps.

<h3>1. Authentication</h3>

The first step there is to create a service account (let's call it <i>hubot</i> for the rest of the article) that we'll use to talk to the Kubernetes API:

```
$ kubectl create serviceaccount hubot
serviceaccount "hubot" created​
```

<pre>
$ kubectl get serviceaccounts hubot -o yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  ...
secrets:
- name: <b>hubot-token-abcd</b>
</pre>

<pre>
$ kubectl get secret hubot-token-abcd -o yaml
apiVersion: v1
data:
  ca.crt: <b>base64-encoded certificate</b>
  token: <b>base64-encoded bearer token</b>
  kind: Secret
metadata:
...
</pre>

With the certificate and the token, we've got everything we need to talk to the Kubernetes API so let's continue!

<h3>2. Your first request</h3>
To make sure we're fine, let's try to make our first request to the Kubernetes API: listing all the nodes of your cluster.

Un-decode the certificate and the bearer token, and save them in two separate files on your  filesystem (called `ca.crt` and `token` in my examples). Then, execute:

```
curl --cacert ca.crt -H "Authorization: Bearer $(cat token)" \
https://{YOUR_KUBE_MASTER_IP}/api/v1/nodes
```

If everything's ok, you should receive a response from the Kubernetes API listing all the nodes from your cluster:
```
{
  "kind": "NodeList",
  "apiVersion": "v1",
  "items": [
    ...
  ]
}
```

The hardest part if behind us!

<h3>3. Updating your deployments to a newer version</h3>

Now, we would like to be able to update our deployments (see [Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) in case you're not familiar with this concept). Basically a deployment is just a set of resources to be managed by Kubernetes.

Let's pretend we have a deployment called `my-app`, containing a simple container based on image `docker.io/bastien/my-container:1`. You updated `my-container`, and would like the new version to get deployed to all the pods managed through `my-app`. Easy!

With `kubectl`, you would normally do:
```
kubectl set image deployment/my-app my-container=my-container:2
```

With cURL, this is (almost) as simple:
````
curl --cacert ca.crt -H "Authorization: Bearer $(cat token)" \
-H "Content-Type: application/json-patch+json" -X PATCH \
-d '[{ \
    "op":"replace", \
    "path":"/spec/template/spec/containers/0/image", \
    "value": "docker.io/bastien/my-container:2" \
  }]' \
https://{YOUR_KUBE_MASTER_IP}/apis/extensions/v1beta1/namespaces/default/deployments/my-app
```

Basically, we're sending a PATCH request to our deployment endpoint, with a BODY containing the patch definition (a `replace` operation on the provided `path` with the new `my-container:2` value), with the appropriate Content-Type. That's it!

<h3>4. Scale your deployments</h3>

Imagine you're under an heavy load and want to scale up your deployment with more replicas? Then you just need one HTTP call, that will do the same thing as:
```
kubectl scale deployment nginx-deployment --replicas 2
```

With cURL, this is what you have to do:
```
curl --cacert ca.crt -H "Authorization: Bearer $(cat token)" \
-H "Content-Type: application/json-patch+json" -X PATCH \
-d '[{ \
    "op":"replace", \
    "path":"/spec/replicas", \
    "value": "2" \
  }]' \
https://{YOUR_KUBE_MASTER_IP}/apis/extensions/v1beta1/namespaces/default/deployments/my-app
```

I think you got it, we just patched our deployment with a new definition for the `spec/replicas` path, with a new value of 2.

<h3>5. Conclusion</h3>

These two examples show how to communicate with the Kubernetes API to manage your deployments, but of course you can manage any Kubernetes resource through the API!

At Hunter, we manage our deployments from CircleCI and Hubot, with plain API calls, without needing to install `kubectl` ✌️

[Let me know](mailto:bastien.libersa@gmail.com) if you have any question or feedback!
