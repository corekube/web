+++
date = "2016-03-07T09:00:00-08:00"
tags = ["Deployments", "Pods", "Secrets", "Services", "Volumes", "config", "sidecar containers"]
title = "Building CoreKube on Kubernetes - Part #1"
slug = "building-corekube-on-kubernetes-part-1"
authors = "Mike Metral"
+++

CoreKube was built with the intention of educating the container and Kuberentes communities by showcasing the Kubernetes project's feature-rich environment, including its versatility around design & management of applications, not machines.

We figured that there was no better way to begin this journey than to build, run & manage the CoreKube site on Kubernetes, and to share our experiences & findings with the community.

Since we plan on walking through various topics, we're going to frame this post as a multi-part series to facilitate following along with both the architecture behind CoreKube, as well as the applied Kubernetes concepts.

Without further ado, let's begin.

<!--more-->

# **Outline**

* [Background]({{<ref "#background">}})
* [Architecture]({{<ref "#architecture">}})
* [Microservices]({{<ref "#microservices">}})
  * [Best Practices]({{<ref "#best-practices">}})
  * [The Pod]({{<ref "#the-pod">}})
  * [Configurations & Secrets]({{<ref "#configurations-secrets">}})
  * [Volumes]({{<ref "#volumes">}})
  * [Services]({{<ref "#services">}})
  * [Deployments]({{<ref "#deployments">}})
* [Conclusion]({{<ref "#conclusion">}})


# **Background**

[Kubernetes](http://kubernetes.io) is an open-source cluster manager for Linux container apps that was originally open-sourced by Google in June 2014, and is a descendant of their internal application management systems [Borg and Omega](http://blog.kubernetes.io/2015/04/borg-predecessor-to-kubernetes.html).

Kubernetes is aimed at removing the traditional operational burden of managing servers. It does so by leveling out the cluster to give the app a global set of resources to use, rather than a grouping of physical servers you have to manage. Apps in Kubernetes can certainly operate more [like cattle, than pets](http://www.networkworld.com/article/2165267/cloud-computing/why-servers-should-be-seen-like-cows--not-puppies.html).

It is also a set of opinionated and declarative concepts that serve as an excellent paradigm for how to structure and manage your apps. By complying with these concepts, Kubernetes will help you achieve a high level of composability and scale for your app from the start.

In this post, we will review how Kubernetes is an integral to CoreKube.

# **Architecture**

CoreKube was initially built with 3 main components, all of which are their own independent, open-source project available at [github.com/corekube](https://github.com/corekube).

The 3 components, at a high-level, are the following:

* [corekube/web](https://github.com/corekube/web) – This is the static site, in Markdown, which is powered & built into HTML with [Hugo](https://github.com/spf13/hugo).
* [corekube/nginx](https://github.com/corekube/nginx) – A Nginx webserver, and Kubernetes microservice, that serves the static site over SSL/TLS using free certificates validated & issued by [LetsEncrypt.org](https://letsencrypt.org).
* [corekube/cert-renew](https://github.com/corekube/cert-renew) – A Kubernetes microservice that is tasked with automatically renewing the [LetsEncrypt.org](https://letsencrypt.org) certs, as they require renewal [at least once in 90 days](https://letsencrypt.org/2015/11/09/why-90-days.html).  Once renewed, `corekube/cert-renew` can also update an existing Kubernetes Secret used to house the certs for a Pod i.e. this is how the Pods in `corekube/nginx` know which certs to use. It can also perform a [rolling-update](http://kubernetes.io/v1.1/docs/user-guide/kubectl/kubectl_rolling-update.html) on these same Pods to make use of the updated Secret, as this is only current way of [informing a Pod that a Secret has changed](http://kubernetes.io/v1.1/docs/user-guide/secrets.html#secret-and-pod-lifetime-interaction). 

Here's a diagram showcasing CoreKube's architecture:
<br>

<div class="container">
  <div class="row text-center">
    <div class="col-xs-12">
      <a href="/images/building-corekube-on-kubernetes-part-1/corekube-architecture-xl.png">
        <img src="/images/building-corekube-on-kubernetes-part-1/corekube-architecture.png" class="img-responsive inline-block"/>
      </a>
      <figcaption><b>CoreKube's Architecture</b><br>(<i>click to enhance</i>)</figcaption>
    </div>
  </div>
 </div>

<br>
Given the overall big picture, let's hone in on the first two components, the `corekube/web` and `corekube/nginx` projects, that drive the content behind the site.

***Note***: *We will cover the `corekube/cert-renew` microservice in the [next installment]({{<relref "posts/2016_03_06-building-corekube-on-kubernetes-part-2.md">}}) of this multi-part series, as it is deserving of its own post.*

# **Microservices**

### **`corekube/web`**

[Hugo](https://github.com/spf13/hugo), a static site generator, does a lot of the legwork for this project, as it provides a skeleton to create a static HTML site from Markdown files.

Therefore, the only real requirement of this project is to provide a means for the `corekube/nginx` microservice to consume the content & serve it to the user. Because we're dealing with Markdown, the [GitHub repo](https://github.com/corekube/web) for the project is a great means of not only applying version control to the content, but it also serves as a datasource to obtain the content, so long as we routinely sync against it.

Because this project's infrastructure doesn't exist outside of GitHub, in terms of the the CoreKube architecture as a whole, `corekube/web` is **not** a resource in Kubernetes other than being a datasource.

### **`corekube/nginx`**

In order to server the HTML content, we need a webserver -- specifically, we're going to be using Nginx.

To adapt a stock Nginx container image to be a Kubernetes microservice, we're going to want to really sink our teeth into Kubernetes and make use of its many features. In Kubernetes, defining a microservice can be as simple as combining a [ReplicationController](http://kubernetes.io/v1.1/docs/user-guide/replication-controller.html) + [Service](http://kubernetes.io/v1.1/docs/user-guide/services.html). Using a microservice in this fashion will enable us to create a hardened, resilient, and scalable architecture that naturally follows the [12-factor methodology](http://12factor.net/).

Let's examine a condensed, but slightly altered version of the `corekube/nginx` Kubernetes [manifest](https://github.com/corekube/nginx/blob/master/k8s/deployment/prod/create-nginx-deployment.yaml.sh), for the sake of brevity:

```
 apiVersion: v1
 kind: ReplicationController
 metadata:
   name: nginx-rc
   labels:
     name: nginx-rc
     rev: ${WERCKER_GIT_COMMIT}
   namespace: nginx-prod
 spec:
   replicas: 3
   template:
     metadata:
       labels:
         name: nginx
     spec:
       containers:
         - name: git-sync
           image: metral/git-sync
           imagePullPolicy: Always
           volumeMounts:
             - name: markdown
               mountPath: /git
           env:
             - <...>
         - name: hugo
           image: metral/hugo
           imagePullPolicy: Always
           args:
             - <...>
           volumeMounts:
             - name: markdown
               mountPath: /src
             - name: html
               mountPath: /dest
           env:
             - <...>
         - name: nginx
           image: ${DOCKER_REPO}:${WERCKER_GIT_COMMIT}
           ports:
             - name: http
               containerPort: 80
             - name: https
               containerPort: 443
           volumeMounts:
             - name: nginx-config-secret
               mountPath: /etc/nginx-config-secret
               readOnly: true
             - name: nginx-ssl-secret
               mountPath: /etc/nginx-ssl-secret
               readOnly: true
             - name: nginx-nfs-pvc
               mountPath: /srv/
             - name: html
               mountPath: /usr/share/nginx/html
               readOnly: true
       volumes:
         - name: nginx-config-secret
           secret:
             secretName: nginx-config-secret
         - name: nginx-ssl-secret
           secret:
             secretName: nginx-ssl-secret
         - name: nginx-nfs-pvc
           persistentVolumeClaim:
             claimName: nginx-nfs-pvc
         - name: markdown
           emptyDir: {}
         - name: html
           emptyDir: {}
```

In this manifest, we can start to see several key pieces of Kubernetes come together to create the microservice, in addition to some best-practices that we've come to define & use throughout our development process. Though this manifest is declarative in nature, one can imperatively interact with Kubernetes as well with tools such as [kubectl](http://kubernetes.io/v1.1/docs/user-guide/kubectl/kubectl.html).

We'll begin by reviewing some of these learned lessons from the manifest, and then we'll move on to describe the containers in detail.

### **Best Practices**

Starting at the top of the `corekube/nginx` manifest, lets review some fundamentals:

 - Using a naming schema for the resource that we're creating adds tremendous value throughout your usage and development with Kubernetes
   - It can help in ways such as identifying which app we're interacting with, or what Kubernetes resource we're creating with `kubectl`. A simple schema, like `<APP_NAME>-<KUBERNETES_RESOURCE>`, works best and is to the point
          - i.e. To create an Nginx ReplicationController's, we'd give it a resource name of be `nginx-rc` and its file would be named `nginx-rc.yaml`.
   - Naming schema's can also be applied into the organization of your code base and directory tree:
      - i.e. For environments: `dev`, `stage`, and `prod` your Service manifest files for the `corekube/nginx` Pods can be structred as such:

          ```
          └── svc
              ├── dev
              │   └── nginx-svc.yaml
              ├── prod
              │   └── nginx-svc.yaml
              └── stage
                  └── nginx-svc.yaml
          ```
        - ***Note***: *We like to name our Kubernetes resources using their shortened, singular name, such as `svc` for a Service, as presented by `kubectl get` - it helps keep commands shorter & easier to remember*
 - Metadata is your friend. It is helpful to both the user and the system as it allows:
	 - A user to query for specific results in the system
	 - The discovery of Pods by a Kubernetes [Service](http://kubernetes.io/v1.1/docs/user-guide/services.html#overview)
		 - i.e. By setting the key `metadata.labels.name` to a value of `nginx-rc`, a Service can locate which Pods in the cluster match that label, and then serves as a proxy for them though an internally accessible virtual IP address
	 - Metadata can be any `key/value` pair of your choosing
		 - i.e. It is common to set the git revision or tag in a key like `metadata.labels.rev`, to easily tell what version of the container image we're running.
 - *"Keep gutting the manifest until it looks like a template"*
	 - You may have noticed that `metadata.labels.rev` in the manifest above is set to `${WERCKER_GIT_COMMIT}`
		 - This is because we believe that you should always keep gutting the manifest until it looks like a template, rather than a fully-declarative, static-like manifest.
	     - Doing so, allows us to *stamp* out renditions of the microservice based on several factors that can change throughout an app's lifecycle, such as the:
	        - Docker image repo
	        - git revision/tag for Docker image being deployed
	        - Namespace being used
	        - Environmental variables

### **The Pod**

A [Pod](http://kubernetes.io/v1.1/docs/user-guide/pods.html) is a colocated collection of containers that share various Linux namespaces, such as the network stack, and even share [Volumes](http://kubernetes.io/v1.1/docs/user-guide/volumes.html) too. The Pod is *the* atomic unit of any microservice in the Kubernetes cluster, as it establishes the foundation for all of the other Kubernetes resources that consume & manipulate it, as well as the concepts that the system is based upon.

Needless to say, it's important that you properly design *what goes in* and *what stays out* of the Pod.

The Pod described in the manifest above for `corekube/nginx` serves as an exact example for what *types* of containers belong in a Pod. Specifically, you should think that a Pod always has one **primary** container, and many *other* containers that will support it in some nominal capacity.

The `nginx` container, which serves the HTML content to the user, is not the only container running within our Pod -- it is actually supported by **two** other containers, better known as *sidecar containers*, which are in charge of certain, smaller ancillary tasks: 

 1. `git-sync`: A container responsible for staying in sync with the `corekube/web` repo. This container assists with the retrieval of the Markdown files and Hugo static site by constantly fetching the latest source code, and placing it into a shared Volume in the Pod, known as the `markdown` Volume.
 2. `hugo`: A container responsible for reading & interpreting the source files placed into the shared Pod Volume, `markdown`, that `git-sync` is tasked with keeping up-to-date. The `hugo` container converts the Markdown files from their source into HTML, builds the static site with Hugo, and then outputs the generated site into another shared Pod Volume, known as `html`.

With the sidecars doing a lot of the foundational work, the last container `nginx`, also attaches itself to the `html` Pod Volume that the `hugo` container filled with the built static site -- this is denoted by the `volumeMounts` key in the manifest.  The `nginx` container then uses the `mountPath` of the `html` volume in the container as the `root` location in `nginx.conf` to ultimately serve the HTML content to the user.

<br>

<div class="container">
  <div class="row text-center">
    <div class="col-xs-offset-3 col-xs-6 col-sm-offset-2 col-sm-5">
      <img src="/images/building-corekube-on-kubernetes-part-1/sidecar.png" class="img-responsive inline-block"/>
      <figcaption><b>A retro Vespa with its sidecar</b></figcaption>
    </div>
  </div>
 </div>

<br>

Separating responsibilities across multiple containers like we've done for the `corekube/nginx` Pod not only makes for a cleaner, decoupled architecture within the Pod, but it sets the tone for how the rest of your architecture should share the same type of modularity.

As you can see by the `git-sync` and `hugo` containers, the sidecars should aim to enable simple functionality for the primary container, while inducing little to no complexity, overhead nor management headache. In short, sidecar containers should be relatively simple in nature, and easy to logically reason about & debug. Other examples of sidecar containers include: logging aggregators, file & object retrievers, and initialization tasks such as establishing a last-known state.

If the apps/containers in question *do not* fit this described mold, then they must independently exist as their own Pods. For example, you wouldn't want to place an entire Djano webapp in one container of a Pod and a separate Nginx container to proxy on its behalf, in the same Pod - not only is this complex from an architecture perspective, it simply doesn't allow each component to scale as it properly should.

### **Configurations & Secrets**

In Kubernetes, a [Secret](http://kubernetes.io/v1.1/docs/user-guide/secrets.html) is an object that holds sensitive information. In actuality, its a `tmpfs` volume in RAM that is created on the Node hosting the Pod which requires it. Once the Secret is created on the Node, it is bind mounted into the Pod and the Secret shares the lifetime of the Pod -- this is how the Pod gains access to information in the Secret.

Contrary to their name, Secrets don't have to *only* be for sensitive information, remember, it's a `tmpfs` mount so anything can go in it. Therefore, Secrets can also be leveraged to provide for the consumption of configuration settings through environmental variables stored in a file, or through other raw data stored in files.

You can see that Secrets are used liberally throughout the `corekube/nginx` manifest.

For instance, the `nginx` container doesn't *only* serve the HTML content, it does so over SSL/TLS using free certificates issued by [LetsEncrypt.org](https://letsencrypt.org). These certs are stored in a Secret named `nginx-ssl-secret`, alongside another Secret for configuration settings called `nginx-config-secret` -- these settings are needed by `nginx` to configure its `nginx.conf`, before the webserver daemon is started.

Because `nginx`specifically consumes these certs via Secret, it doesn't have to depend on some extraneous datasource nor have to worry about renewing the certs once every 90 days per [LetsEncrypt.org requirements](https://letsencrypt.org/2015/11/09/why-90-days.html). Instead, it leaves these tasks to the other one of main 3 components that make up CoreKube:  the`cert-renew` microservice.

***Reminder:*** *`cert-renew`, its responsibilities, and architecture, are covered in the [follow-up installment]({{<relref "posts/2016_03_06-building-corekube-on-kubernetes-part-2.md">}}) of this multi-part series.*

Making use of Secrets enforces that we don't hard-code configuration settings into the manifest, nor accidentally publish sensitive information such as usernames, passwords or the SSL/TLS certs. Secrets also aid in keeping the physical existence of sensitive information to a minimum. Given the fact that the Pod **must** define its Secrets in the manifest before it can be instantiated, this automatically ensures that the Pod **cannot** be stood up without the Secrets already existing in the Kubernetes system -- this approach creates a great proverbial system of checks & balances for your microservice and the informational items it relies on.

### **Volumes**

We established that a Secret is a `tmpfs` volume, but in CoreKube we make use of other sorts of Kubernetes [Volumes](http://kubernetes.io/v1.1/docs/user-guide/volumes.html) too. In the `nginx` container, there is a Volume labeled `nginx-nfs-pvc` which stands for a NFS [PersisitentVolumeClaim(PVC)](http://kubernetes.io/v1.1/docs/user-guide/persistent-volumes.html). This particular Volume houses the [LetsEncrypt.org](https://letsencrypt.org) data, as well as the physical SSL/TLS certs we've been issued and store in a NFS server.

Access to the NFS via the PVC is needed in order to respond to a HTTP request sent by [LetsEncrypt.org](https://letsencrypt.org) whenever a certificate renewal is initiated -- this is how [LetsEncrypt.org](https://letsencrypt.org) validates that the Domain in question is in fact being managed by this particular webserver, the `nginx` container, at the time of renewal.

A [PersistentVolume(PV)](http://kubernetes.io/v1.1/docs/user-guide/persistent-volumes.html) is a piece of system storage for which a Pod has a *claim* on - hence the term *PersistentVolumeClaim*, and it is this abstraction that facilitates access to the actual Volume type for the Pod. A PVC is nothing more than a *request* for a storage type instantiated by a PV. A PVC and a PV have a 1:1 ratio, and a Pod can only consume a PVC in its template specification, not a PV directly. This is intentional, and also great, because it abstracts away the details of the underlying volume type from both the Pod's definition, as well as the PVC's definition. In this manner, these two resources can ultimately reference the PV resource solely by name, making the PV object the sole bearer of any specific volume configuration & access settings. Therefore, the Pod doesn't need to know about any of the Volume's details, and it is one less piece of information that it needs to worry about declaring or maintaining.

In [The Pod,]({{<relref "#the-pod">}}) we saw that with content in hand from the `html` Volume populated by the efforts of the `hugo` and `git-sync` sidecars, as well its configurations, data and SSL/TLS certs mounted as Secrets and Volumes, the `nginx` container washes its hands clean of any of the *messy* work needed for our goals. It doesn't have to generate the HTML, renew the SSL/TLS certs, nor worry about accessing & passing data across the Pod's Volumes for that matter, and focuses strictly on being a webserver -- as it *should* be doing.

Using PersistentVolumes in this capacity further extends the modularity of your microservice and its design. Expect more information on the various Volume types in a future post.

As a quick note, you may be asking yourself, if the `nginx` container *already* had access to the SSL/TLS certs directly through this PVC, why would we also have put the certificate data in a Secret, as noted in the previous section? The simple answer: modularity & best-practice. By putting the SSL/TLS certs into a Secret, we instill good practice into the `nginx` container for future iteration, if we ever decide to remove the PVC from this Pod. Having the certs exist as a Secret also allows other microservices in the same Namespace to access these if they require it. Not to mention, it's just good, overall practice to create Secrets and Volumes for this sort of data because it keeps it out of the Pod's manifest.

### **Services**

Now that we've established how the `corekube/web` and `corekube/nginx` components function from the ground up to serve CoreKube's content, we need a way for Kubernetes system to provide both internal and external access to our app. This is where Kubernetes [Services](http://kubernetes.io/v1.1/docs/user-guide/services.html) come in to play.

Services are merely an abstraction for a set of Pods. A Service is tied to an administrative policy which governs how the Pods should be accessed and load balanced upon. You can think of a Service as a pseudo-layer 3 construct, as it generates a single virtual IP address internal to the Kubernetes cluster to route resources to the respective Pods. It is also capable of manipulating the networking rules of your Nodes to help establish the proper plumbing needed for external access to the app.

The Service created and used for the `corekube/nginx` component can be found [here](https://github.com/corekube/nginx/blob/master/k8s/svc/prod/nginx-svc.yaml), and is pretty straightforward as far as Kubernetes Services go.

### **Deployments**

As we approach the end of this post, it is worth noting that though the `corekube/nginx` manifest above is not a direct match to the [actual manifest used](https://github.com/corekube/nginx/blob/master/k8s/deployment/prod/create-nginx-deployment.yaml.sh) in CoreKube, though it shares a lot of the same foundation.

The main driver behind the discrepancies in both of the aforementioned manifests is because a new feature in Kubernetes, known as [Deployments](http://kubernetes.io/v1.1/docs/user-guide/deployments.html), has emerged as we get close to the v1.2 release. Deployments supply a more declarative means of pushing out changes to both your app and its infrastructure throughout its lifecycle.

Deployments make use of a new component, known as [ReplicaSets](https://github.com/kubernetes/kubernetes/blob/v1.2.0-alpha.8/pkg/apis/extensions/types.go#L817-L827), which are similar to [ReplicationControllers](http://kubernetes.io/v1.1/docs/user-guide/replication-controller.html), but aim to replace ReplicationControllers in both name and functionality for the sake of reducing complexity & confusion. If you are interested, you can track the history of this ongoing effort [here](https://github.com/kubernetes/kubernetes/issues/3024).

As it stands today, performing rolling-updates in Kubernetes with `kubectl rolling-update` is only possible on ReplicationController objects, but it offers a simple means to propagate changes to your Pods. It must be stated, that though you can perform a `rolling-update` on ReplicationController to change, add or remove components of the Pod, its **unofficial**, primary use-case is really only limited to modifying the Docker Image being used by the Pod's containers. We make this claim because once you make the switch to Deployments, you'll notice that they provide a more applicable, declarative standard to use when pushing out new *deployments* of your app, where several of its components could have changed.

You can expect to find more information on Deployments and ReplicaSets from the project and community once v1.2 has landed.

# **Conclusion**

By this point, we hope that you've been exposed to and have learned a vast amount about Kubernetes in a short period of time. From its rich feature-set, to its many resources, as well as its laser focused vision on proper app design, Kubernetes equips you with some powerful tools. These capabilities aid in the resiliency and scale of your apps, and leverage decades of best practices from the kind folks over at Google.
 
Kubernetes and the container ecosystem as a whole, is in a state of high activity where lots of great work is actively being done, so its crucial to stay tuned into the latest developments. It is worth stating that every time we visit [github.com/kubernetes/kubernetes](https://github.com/kubernetes/kubernetes), it is a fact that we walk away learning something new.

We hope that you've enjoyed this post as much as we did writing it. We look forward to providing you with not only the next installment of the **Building CoreKube on Kubernetes** multi-part series, but with many other topics in the Kubernetes space.
