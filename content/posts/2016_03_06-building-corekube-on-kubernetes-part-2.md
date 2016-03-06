+++
date = "2016-03-07T09:01:00-08:00"
tags = ["AdmissionController", "LetsEncrypt", "Namespaces", "ServiceAccounts"]
title = "Building CoreKube on Kubernetes - Part #2"
slug = "building-corekube-on-kubernetes-part-2"
authors = "Mike Metral"
+++

Welcome Back!

In this installment, we will be covering the third and final component that operates CoreKube, and that is the `corekube/cert-renew` project that we left out from our initial post: [Building CoreKube on Kubernetes - Part #1]({{<relref "posts/2016_03_06-building-corekube-on-kubernetes-part-1.md">}}).

If you haven't read [Part #1]({{<relref "posts/2016_03_06-building-corekube-on-kubernetes-part-1.md">}}) yet, now is a great time to do so!

We plan on covering the `corekube/cert-renew` project at a fair-length, as well as how we leverage various other components of the Kubernetes project to assist in the renewal of our SSL/TLS certs from [LetsEncrypt.org](https://letsencrypt.org).

<!--more-->

# **Outline**

* [Background]({{<ref "#background">}})
* [Microservice]({{<ref "#microservice">}})
* [The Manifest]({{<ref "#the-manifest">}})
* [Namespaces & ServiceAccounts]({{<ref "#namespaces-serviceaccounts">}})
	* [Namespaces]({{<ref "#namespaces">}})
	* [ServiceAccounts]({{<ref "#serviceaccounts">}})
* [Conclusion]({{<ref "#conclusion">}})

# **Background**

The `corekube/cert-renew` project is a fork of [ployst/docker-letsencrypt](https://github.com/ployst/docker-letsencrypt), a Nginx Docker image that handles both the renewal of the certificates (certs), as well as, uses the certs in a Nginx webserver to enable SSL/TLS. 

We were looking for a bit more of a delineation between the tasks of renewing the certs, and consuming them in a webserver, so we ended up splitting the responsibilities into separate microservices. This affords us modularity and it enforces scope at many layers. In otherwords, `ployst/docker-letsencrypt` was split into the microservices `corekube/nginx` and `corekube/cert-renew` that power CoreKube.

# **Microservice**

The `corekube/cert-renew` microservice is comprised of a set of scripts tasked with the following duties:

 - Renew/fetch the new certs from [LetsEncrypt.org](https://letsencrypt.org).
 - Generate or update the Kubernetes Secret that the webserver container will use to retrieve the certs from -- in `corekube/nginx`, this is `nginx-ssl-secret` which we covered in [Configurations & Secrets]({{<relref "posts/2016_03_06-building-corekube-on-kubernetes-part-1.md#configurations-secrets">}}) in Part #1.
 - And lastly, perfom a [rolling-update](http://kubernetes.io/v1.1/docs/user-guide/kubectl/kubectl_rolling-update.html) on a [ReplicationController](http://kubernetes.io/v1.1/docs/user-guide/replication-controller.html), as this is the only current way available to [inform a Pod that a Secret has changed](http://kubernetes.io/v1.1/docs/user-guide/secrets.html#secret-and-pod-lifetime-interaction).

Most importantly, it runs the preceding steps as a collective cron job, once a month, to ensure we are actively renewing our certs on a regular cadence.

# **The Manifest**

Let’s examine the `corekube/cert-renew` Kubernetes [manifest](https://github.com/corekube/cert-renew/blob/master/k8s/rc/prod/create-cert-renew-rc.yaml.sh):

```
 apiVersion: v1
 kind: ReplicationController
 metadata:
   name: cert-renew-rc
   labels:
     name: cert-renew-rc
   namespace: cert-renew-prod
 spec:
   replicas: 1
   selector:
     name: cert-renew
     rev: ${WERCKER_GIT_COMMIT}
   template:
     metadata:
       labels:
         name: cert-renew
         rev: ${WERCKER_GIT_COMMIT}
     spec:
       containers:
         - name: cert-renew
           image: ${DOCKER_REPO}:${WERCKER_GIT_COMMIT}
           volumeMounts:
             - name: cert-renew-config-secret
               mountPath: /etc/cert-renew-config-secret
               readOnly: true
             - name: cert-renew-nfs-pvc
               mountPath: /srv/
               readOnly: false
       volumes:
         - name: cert-renew-config-secret
           secret:
             secretName: cert-renew-config-secret
         - name: cert-renew-nfs-pvc
           persistentVolumeClaim:
             claimName: cert-renew-nfs-pvc
```

As you can tell, the `corekube/cert-renew` manifest shares a lot of the same styling and [Best Practices]({{<relref "posts/2016_03_06-building-corekube-on-kubernetes-part-1.md#best-practices">}}) discussed in Part #1 of this series:

 - We enforce templating in our Pod's specification to accommodate for variable parameters & settings
 - The usage of Secrets is exercised to hold the configuration data for the `cert-renew` container, as seen in `cert-renew-config-secret`
 - And lastly, `cert-renew-nfs-pvc` - a [PersistentVolumeClaim](http://kubernetes.io/v1.1/docs/user-guide/persistent-volumes.html) for a NFS server, is made available so that when the cert renewal takes place using the [LetsEncrypt.org](https://letsencrypt.org) client, it has all the data it needs to execute properly.

The `corekube/cert-renew` Pod differs from `corekube/nginx` not only in their responsibilities and manifest details, but the former utilizes a [ReplicationController](http://kubernetes.io/v1.1/docs/user-guide/replication-controller.html), where the latter makes use of a [Deployment](http://kubernetes.io/v1.1/docs/user-guide/deployments.html) controller as detailed in [Part #1's Deployments]({{<relref "posts/2016_03_06-building-corekube-on-kubernetes-part-1.md#deployments">}}) section.

Though it is still in its early phase, one can expect that Deployments will supersede ReplicationControllers as the default controller for constructing your app's Pod.

Also, because there aren't any sidecar containers supporting `cert-renew` as we saw with the `corekube/nginx` microservice, we generally don't need this Pod to have more than one replica because it is a single, monthly task which executes and then just waits. It may even appear that using a ReplicationController for `corekube/cert-renew` seems a bit like overkill for this simple Pod, but it is idiomatic in Kubernetes to always define your Pod within the scope of a *controller* type. The Kubernetes [documentation on Pods](http://kubernetes.io/v1.1/docs/user-guide/pods.html#durability-of-pods-or-lack-thereof) states that:

> Pods aren't intended to be treated as durable pets. They won't survive scheduling failures, node failures, or other evictions, such as due to lack of resources, or in the case of node maintenance.

>In general, users shouldn't need to create pods directly. They should almost always use controllers (e.g., replication controller), even for singletons. Controllers provide self-healing with a cluster scope, as well as replication and rollout management.

Going one step further, a ReplicationController for the `cert-renew` container isn't the most appropriate controller to be using with it either - realistically, a Kubernetes [Job](http://kubernetes.io/v1.1/docs/user-guide/jobs.html) is better suited as it was specifically created with the intention to run a one-time type of task like this, or better stated, a *job* is responsible for assuring that a specified number of Pods successfully terminated. The only drawback to a Job, for our specific purposes, is that we would like to run it on a schedule of some sort to deterministically know when the cert renewal was taking place, and a *cron-enabled* Job is actually in the works as speak - if you are interested, you can track it [here](https://github.com/kubernetes/kubernetes/issues/2156).

# **Namespaces & ServiceAccounts**

### **Namespaces**

Resources in Kubernetes are automatically shared between microservices *if* their Pods exist in the *same* [Namespace](http://kubernetes.io/v1.1/docs/user-guide/namespaces.html), or better stated, if these microservices are a part of the same virtual grouping of interrelated apps, then they naturally share objects with one another.

Kubernetes Namespaces bring forth a healthy dose of organization and scope for your project's components, and they *can and should be* used generously in the system. Doing so, will allow you to differ between your various apps/microservices, teams, and deployment environments such as: development, stage & production.	 

### **ServiceAccounts**

In the **Components** section above, we stated that the `corekube/cert-renew` project is comprised of a set of scripts tasked to do many efforts with regards to the renewal of the SSL/TLS certs:

 - One of these tasks, is to have `corekube/cert-renew` save the renewed certs to a Kubernetes Secret that exists in the distinct Namespace of the`corekube/nginx` Pod, as we use different Namespaces for each microservice.
	 - This Secret,`nginx-ssl-secret`, is the resource that contains the data that `corekube/nginx` needs to secure the connection in its configuration file, `nginx.conf`, as described in [Part #1]({{<relref "posts/2016_03_06-building-corekube-on-kubernetes-part-1.md#configurations-secrets">}}).
	 - Another task that it performs, is the rolling-update of a ReplicationController, so that the Pod can be given any *renewed* SSL/TLS certs stored in its Secret `nginx-ssl-secret`, as this is the only current way available to [inform a Pod that a Secret has changed](http://kubernetes.io/v1.1/docs/user-guide/secrets.html#secret-and-pod-lifetime-interaction).
	 - 	Thus, in order for both of these tasks that `corekube/cert-renew` performs to actually function, it **must** be able to interact with the Kubernetes system to perform these actions in some fashion.

If you haven't yet realized it, we're stating that the `cert-renew` container, which is a part of the `corekube/cert-renew` Pod running in its own Namespace, is executing *system-level* commands in the Kubernetes cluster, such as updating a Secret or performing a rolling-update on a ReplicationController, from *within* the Pod itself. *How is it able to perform these softs of administrative actions in Kubernetes? And with what credentials?*

With [ServiceAccounts](http://kubernetes.io/v1.1/docs/user-guide/service-accounts.html).

ServiceAccounts in Kubernetes supply Pods with a means of authentication into the cluster by establishing an approved *identity* for them to utilize. ServiceAccounts also have some neat bonus features, like being configured to ensure that **all** Pods in a Namespace start up with the same [imagePullSecret](http://kubernetes.io/v1.1/docs/user-guide/service-accounts.html#adding-imagepullsecrets-to-a-service-account) in its template spec, so you don't have to do this act yourself. 

ServiceAccounts can empower Pods, because when a Pod is instantiated, it is *automatically* added to the existing *default* ServiceAccount. This *default* ServiceAccount comes configured with an identity in the Namespace for the Pods to use, if need be. This automatic capability is only possible if the ServiceAccount is *implemented* in the Namespace at the time the cluster was configured & started. The way in which ServiceAccounts become implemented is through the [ServiceAccount AdmissionController](http://kubernetes.io/v1.1/docs/admin/admission-controllers.html#serviceaccount) that **must** be enabled in the APIServer beforehand, if you intend to work with ServiceAccounts.

In this *default* ServiceAccount, there is a Kubernetes Secret with type *kubernetes.io/service-account-token* that the Pod has access to. The Secret has a name in the scheme of `default-token-<ID>`, and when examined, reveals a CA certificate and a token that will be end up mounted as files in a Volume for use by the Pod.

It is with this certificate and token, that `kubectl` [has all the necessary information](https://godoc.org/k8s.io/kubernetes/pkg/kubectl/cmd/util#DefaultClientConfig) needed to perform actions in the Kubernetes environment, such as updating a Secret or performing a rolling-update on a ReplicationController.

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
***Note***: *The reason why the certificate and token already existed in the ServiceAccount as a Secret, is because we previously enabled SSL/TLS in our Kubernetes cluster, and configured the usage of [Client certificate authentication](http://kubernetes.github.io/docs/admin/authentication) for controlled access. It is with the keys generated in this process, that we can equip the APIServer to properly enable the ServiceAccount feature, so that it can automatically generate authenticated identities for the Pod -- more on this topic in a future post*.

Lastly, it's worth mentioning that the ability to use ServiceAccounts with bonus features such as seen with an [imagePullSecret](http://kubernetes.io/v1.1/docs/user-guide/images.html#specifying-imagepullsecrets-on-a-pod), is quite useful. In this particular example, the ServiceAccount is able to give the [Kubelet](http://kubernetes.io/v1.1/docs/admin/kubelet.html), which if you remember runs on any Node hosting a particular Pod, the necessary credentials needed to pull down container images from a private registry, so that it can instantiate the Pod on its hardware. This necessary, but repetitive task, can be taken care of for your Pods automatically once it's been setup, and expecting more types of functionality of this caliber in the future should prove to be an interesting follow.

# **Conclusion**

This concludes the multi-part series on **Building CoreKube on Kubernetes** *for now*.

We hope that you've learned a tremendous amount about the Kubernetes including some best practices and its features that we covered ranging from the Namespace, to ServiceAccounts, and AdmissionControllers.

We have an assortment of topics that we plan on posting in the near future, so please stay tuned. As always, we welcome any discussion, questions, comments and feedback!

P.S. We will be attending [KubeCon](https://kubecon.io/) this week in London and plan on posting updates as the conference unfolds. We're looking forward to seeing what great work the community has been up to!