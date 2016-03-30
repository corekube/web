+++
date = "2016-03-31T09:00:00-08:00"
tags = ["Basics", "Borg"]
title = "Why Does Kubernetes Matter?"
slug = "why-does-k8s-matter"
authors = "Mike Metral"
+++

In this segment, we want to take a step back from all of the in-depth technical details, and assess why it is that containers, Kubernetes, and the paradigm it presents, is worth using & integrating into your stack.

Our aim is to provide you with a holistic view of how you should be thinking about and visualizing your infrastructure, and to get to the root of the topic: *why does Kubernetes matter?*
<!--more-->

# **Outline**

* [Introduction]({{<ref "#introduction">}})
* [Kubernetes' Roots]({{<ref "#kubernetes-roots">}})
* [Why Kubernetes Matters]({{<ref "#why-kubernetes-matters">}})
	* [Functional]({{<ref "#functional">}})
	* [Personas]({{<ref "#personas">}})
* [The Big Picture]({{<ref "#the-big-picture">}})
* [Conclusion]({{<ref "#conclusion">}})


## **Introduction**

[Kubernetes](http://kubernetes.io) is aimed at being the management layer for your containers. However, it also has a heavy focus on seamlessly providing & satisfying all of the real-world requirements your app needs and relies on. Examples of these app necessities, which are provided by Kubernetes, include: access to vendor agnostic volumes, load-balancing, redundancy, resiliency, issuing updates, and the management of configurations & secrets. 

It is with capabilities & features such as the ones mentioned above, plus the packaging provided by runtimes like Docker with the containers themselves, that the practice of managing apps, not servers, starts to unfold by utilizing Kubernetes.

## **Kubernetes' Roots**

Kubernetes, which was originally started by Google, has its roots in the Google systems: [Borg](http://research.google.com/pubs/pub43438.html) and [Omega](http://research.google.com/pubs/pub41684.html). Many of the same concepts that drove the design and implementation of these systems has permeated into Kubernetes as a new manifestation which includes todays standards, as well as, incorporates many of the best practices that Google learned the *hard way* over the last decade.

Kubernetes is not an "open-source version" of Borg or Omega as many like to initially think; rather, it is a grand effort by Google to create a new management tool for your jobs & services, out in the open, where anyone and any idea is welcome.

Kubernetes began by leveraging many years of architectural & operational expertise at Google, but because it is open source and has proven itself to truly simplify development, operations and administrative duties, it has amassed an enourmous following & contributions since its [initial public commit in June 2014](https://github.com/kubernetes/kubernetes/commit/2c4b3a562ce34cddc3f8218a2c4d11c7310e6d56).

Here is a snapshot of the activity being drummed up by the community:

<br>
  <div class="row text-center">
    <div class="col-xs-offset-1 col-xs-10">
      <div>
        <a href="/images/why-does-k8s-matter/commits.png">
	        <img src="/images/why-does-k8s-matter/commits.png" class="img-responsive center-block"/>
	    </a>
	      <br>
	      <figcaption>(<i>Source: Google</i>)</figcaption>
      </div>
    </div>
  </div>
<br>

<br>
  <div class="row text-center">
    <div class="col-xs-offset-1 col-xs-10">
      <div>
        <a href="/images/why-does-k8s-matter/velocity.png">
	        <img src="/images/why-does-k8s-matter/velocity.png" class="img-responsive center-block"/>
	    </a>
	      <br>
	      <figcaption>(<i>Source: Google</i>)</figcaption>
      </div>
    </div>
  </div>
<br>

<br>
  <div class="row text-center">
    <div class="col-xs-offset-1 col-xs-10">
      <div>
        <a href="/images/why-does-k8s-matter/top10.png">
	        <img src="/images/why-does-k8s-matter/top10.png" class="img-responsive center-block"/>
	    </a>
	      <br>
	      <figcaption>(<i>Source: RedMonk</i>)</figcaption>
      </div>
    </div>
  </div>
<br>

These diagrams are nothing short of describing a true, exceptional, and collaborative technical community.

## **Buy-In**

As we just reviewed, there is a great deal of buy-in by many individuals and companies into Kubernetes. However,  the real question is, do you buy into, what initially began as, the *Google way* of running your apps & services?

What we're getting at, is that Kubernetes isn't just a container management system, it is an *insight* into how Google runs their infrastructure for practically every service & product ranging from Gmail, Search, Maps, to even Google Cloud Engine (GCE), and the list goes on.

Accordingly, Kubernetes is also a means to *run like Google,* so what you're essentially signing up for is access to a prescriptive set of design principles that allow your apps to operate efficiently in, with a proven track record established by Google which allows you to scale & manage your apps with ease. This is not to say that underlying systems like OpenStack or AWS, which are aimed at handling IaaS resources, go away, it simply means that these systems stay put doing what they do best, and Kubernetes is brought in for all of your apps needs. Ultimately, integrating Kubernetes creates a marriage of favorable components.

Therefore, if you're considering Kubernetes for your stack, you must *trust* the foundation and the paradigms presented by the project, starting at the Pod, and the rest of the concepts will naturally follow. Doing so, will reveal an unparalleled combination of functionality and flexibility that you have at your disposal, as well as, the vision Kubernetes embodies to aid in redefining how your apps are constructed.

## **Why Kubernetes Matters**

As detailed in the most recent paper in the [ACM](https://www.acm.org/), [*Borg, Omega, and Kubernetes*](http://queue.acm.org/detail.cfm?id=2898444), Kubernetes helps establish a toolset to help manage & scale both your apps and teams.

The following are ways in which Kubernetes enables improved app development:

### **Functional**

 - Containers encapsulate the application environment, abstracting away    many details of machines and operating systems from the application    developer and the deployment infrastructure.
 - By shifting the management APIs from machine-oriented to application-oriented, application deployment and introspection dramatically improves, all while shifting the "primary key" of the data center from machine to application.
 - The shift of APIs to apps allows teams to not worry about specifics of machines and operating systems.
 - Focusing on apps over machines, also allows teams to operate in a much more flexible, smaller, and modular manner.
	 - The reason being is because a common use pattern of Kubernetes is for a Pod to hold an *instance*, or rather a component, of a complex application, which can be written, ran, and managed by a small team solely responsible for that functionality.
	 - Even other components of Kubernetes intended to improve your app are built-upon the concept of the Pod such as ReplicaSets, Deployments, Services etc., thus, consolidating all of the application's requirements, business policies, and team, become simple & seamless. You can explore the various concepts of Kubernetes that not only interact with a Pod, but which allow you to create new functionality for your apps in the [user guide's Glossary](http://kubernetes.io/docs/user-guide/pods/).

### **Personas**

Kubernetes also aids your organization by appealing to a slew of different personas:

 - **Developers**: can not only create versatile apps, but they can utilize instrinsic properties of the cluster to achieve particular requirements of any and all apps
	 - For usecases where devs want to target a particular Node, or set of Nodes, specific labels which denote different hardware specifics can be used for individual Pod scheduling, i.e. if you want to run your app on AMD CPU architectures instead of Intel, or if you wish to make use of GPUs, or even Nodes with large amounts of RAM.
	 - Consuming a fleet of hetergenous machines is not only possible in Kubernetes, but it actually levels out all of the machines by presenting them as a sea of generic computing resources.
		 - This embodies the Pets vs. Cattle ideology with not only your apps, but with your machines as well.
 - **Devops**: Kubernetes concepts such as Deployments, ReplicaSets, Services etc. all help alleviate ops by ensuring that not only does each app have a declarative set of business policies, but that these specs are enforced and maintained at all times.
 - **Admins**: As a part of Kubernetes, admins can gain access to & process container resource utilizaiton using tools like [Heapster](https://github.com/kubernetes/heapster) and [cAdvisor](https://github.com/google/cadvisor), as well as, examine cluster events, API requests, monitoring data, and make use of features like [kubedash](https://github.com/kubernetes/kubedash) for analytics, that either comes out of the box or is easily pluggable.
	 - These various metrics don't only provide insight into the Kubernetes cluster, it provides an even granular understanding of the apps themselves, since they are each individually contained.
	 - Analysis at the app layer is not possible in many other systems without some heavy implementation left to the user. The native inclusion of these analysis capabilities in Kubernetes speaks to the project's efforts that aren't only aimed at prepping your app for growth, but that information of this caliber is a definite requirement for your app and you shouldn't have to build this functionality yourself, but rather, it should be provided to you by the system.
 - **Others**: Lastly, many different personas can interact with Kubernetes at a basic level by making use of and visualizing your cluster with the [dashboard](https://github.com/kubernetes/dashboard), all while performing actions such as creating new resources, and performing queries against resources using Labels for cases such as inspection or report generation.

## **The Big Picture**

In an attempt to help capture the big picture of how Kubernetes operates, we wanted to showcase a couple of visuals that may help you garner a better understanding of the compartmental aspects of the project.

As the adage by James Burke says:

> You can only know where you're going if you know where you've been

This statement is immensely relevant to Kubernetes given that Borg is its ancestor, and it sets the stage for how you should organizationally think about your cluster. 

Lets examine some visuals drafted from the [Borg paper](http://research.google.com/pubs/pub43438.html), which will not only give us a peak into how Borg is deployed, but how this same model can apply to Kubernetes.

Here we can visualize what our overarching cloud architecture would look like from a top-down approach:

<br>
  <div class="row text-center">
    <div class="col-xs-offset-1 col-xs-10">
      <div>
        <a href="/images/why-does-k8s-matter/sites.png">
	        <img src="/images/why-does-k8s-matter/sites.png" class="img-responsive center-block"/>
	    </a>
	      <br>
	      <figcaption>(<i>A deployment of Borg across various Datacenters</i>)</figcaption>
      </div>
    </div>
  </div>
<br>


----------

If we zoom in further, we can examine that each building in a Datacenter contains at least one Borg cluster, which is divided into a cell of ~10,000 machines:

<br>
  <div class="row text-center">
    <div class="col-xs-offset-1 col-xs-10">
      <div>
        <a href="/images/why-does-k8s-matter/cluster.png">
	        <img src="/images/why-does-k8s-matter/cluster.png" class="img-responsive center-block"/>
	    </a>
	      <br>
	      <figcaption>(<i>An inspection of a specific cluster within a Datacenter in Borg</i>)</figcaption>
      </div>
    </div>
  </div>
<br>


----------

Further reviewing the cell itself, we can catch a glimpse as to how the control plane components differ from the workers / Nodes, and how Borg allocs, similar to Kubernetes Pods, are the only atomic unit for any app or service across your entire deployment:

<br>
  <div class="row text-center">
    <div class="col-xs-offset-1 col-xs-10">
      <div>
        <a href="/images/why-does-k8s-matter/cell.png">
	        <img src="/images/why-does-k8s-matter/cell.png" class="img-responsive center-block"/>
	    </a>
	      <br>
	      <figcaption>(<i>An inspection of a specific cell within a Datacenter in Borg</i>)</figcaption>
      </div>
    </div>
  </div>
<br>

As you've probably noticed by now, there are parallels into the components of Borg and what exists today in Kubernetes, particularly the 1:1 mapping of clusters and Pods - these similarities really start to make more sense as you think of a [federated Kubernetes](https://github.com/kubernetes/kubernetes/blob/master/docs/proposals/federation-lite.md) deployment.

Though Kubernetes cannot currently scale to 10,000 Nodes per cluster like Borg can, it has been [recently optimized](http://blog.kubernetes.io/2016/03/1000-nodes-and-beyond-updates-to-Kubernetes-performance-and-scalability-in-12.html) to allow clusters to support upto 1000 Nodes and 30,000 Pods where 99% of API calls respond in less than 1 second, and 99% of Pods (with pre-pulled images) start within 5 seconds - huge gains representing a 10x gain in scale, which reportedly is actually closer to 14x increase according to folks at Google.

Kubernetes is certainly ready for prime-time not only because many companies are using it in production today (see community diagrams above), but also from a pure performance & scale play.

## **Conclusion**

We hope that you've been able to gain a better understanding as to the various reasons why Kubernetes matters in this day in age of software development, and how you can begin to organize & architect your clusters to allow for its integration.

Embracing Kubernetes' prescriptive ways may seem like a fire-hose at first, but its intrinsically well-thought out design principles really showcase not only the proper methodologies for software development, but for every important, ancillary component around the app from ops, to administration and others.

Til next time!