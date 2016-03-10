+++
date = "2016-03-10T09:00:00-08:00"
tags = ["KubeCon", "Community", "Enterprise", "OpenStack", "Monitoring"]
title = "KubeCon London 2016 - Day 1"
slug = "kubecon-london-2016-day-1"
authors = "Mike Metral"
+++

Hello from London!

Today was the first day of [KubeCon](https://kubecon.io/) Europe in London, England, and the excitement & energy at this event has been nothing short of electrifying!

Lets review a few summaries & notes on a subset of the talks given today.

<!--more-->

Around 500 folks from around the globe gathered to review the latest & upcoming features of the Kubernetes project, as well as, how the community is converging on various efforts to strengthen & solidify the foundation, plus, unveil how they're using Kubernetes internally.

<br>
<div class="container">
  <div class="row text-center">
    <div class="col-xs-6 col-sm-offset-1 col-sm-4 nopadding">
		  <a href="/images/kubecon-london-2016/kubecon.jpg">
		      <img src="/images/kubecon-london-2016/kubecon.jpg" class="img-responsive"/>
		  </a>
	  </div>
    <div class="col-xs-6 col-sm-4 nopadding">
	  <a href="/images/kubecon-london-2016/crowd.jpg">
	      <img src="/images/kubecon-london-2016/crowd.jpg" class="img-responsive"/>
	  </a>
    </div>
  </div>
</div>
<br>

# **Outline**

* [Opening Keynote: Pushing Kubernetes Forward]({{<ref "#day-1-opening-keynote-kubernetes-update">}})
* [Kubernetes State of the Union]({{<ref "#kubernetes-state-of-the-union">}})
* [What is OpenStack's role in a Kubernetes world?]({{<ref "#what-is-openstack-s-role-in-a-kubernetes-world">}})
* [Hybrid Apps: Orchestrating Cloud-Native and Traditional Application Architectures]({{<ref "#hybrid-apps-orchestrating-cloud-native-and-traditional-application-architectures">}})
* [Monitoring Microservices: Docker, Kubernetes, and GKE Visibility at Scale]({{<ref "#monitoring-microservices-docker-kubernetes-and-gke-visibility-at-scale">}})

## **Day 1 Opening Keynote: Kubernetes Update**

* [Talk Abstract](https://kubeconeurope2016.sched.org/event/5oMA/day-1-opening-keynote-kubernetes-update)
* Summary
	* Kubernetes is a framework for building distributed systems
		* It is **not** a PaaS
	* Tip: Make sure you use version numbers with your images
		* The *latest* tag is **not** a version to deploy
	* If you're new to Kubernetes, an easy, out of the box route to play with the project is via `kubectl`
		* It allows you to on-ramp with using the project, rather than jumping straight into dealing with Kubernetes manifests
	* Pods Refresher
		* Logical application includes:
			* 1+ conatiners and Volumes
			* Shared namespaces
			* One **IP per Pod** model
	* Tip: Do **not** try to convert *the world* into Kubernetes right off-the-bat - start small
	* Note: ReplicaSets replaced ReplicationControllers
	* ConfigMaps (new as of v1.2)
		* **New Feature**: A mechanism to create loose descriptions for configurations either through environmental variables or files, and it notifies the application to restart itself so that it can reference the updated settings
	* Ingress offers:
		* Integration with load balancers
		* Manage SSL/TLS for the first time
		* *Note: This is still a work in progress*
	* Manifest features to use/integrate into your apps
		* `lifecycle`: steps to run on container quit/termination
		* `livenessProbe`: enable a check system for app readiness
	* Scaling with `kubectl` is too imperative
		* Use Deployments instead to be more state-aware & manage your apps server-side, rather than something like `kubectl scale` client-side


----------


## **Kubernetes State of the Union**

* [Talk Abstract](https://kubeconeurope2016.sched.org/event/6OA5/kubernetes-state-of-the-union)
* Summary
	* New releases are occuring, on average, every 3 months
	* Velocity
		* 5k commits in 1.2
		* +50% Unique contributors
		* 1200+ External projects based on k8s
	* Vision
		* Give everyone the power to run agile, reliable, distributed systems at scale
	* Kubernetes v1.2 Release
		* New UI
		* Improved scale - 400% increase in number of nodes used
		* Simplified application deployments
		* Automated cluster management
		* Third party extensibility
		* **Available by end of March**
	* The Future
		* Kubernetes 1.3 has already begun
			* Legacy app support (aka "Pet Set")
			* Cluster Federation (aka Ubernetes)
			* Scale++
			* In-cluster IAM
			* Cluster autoscaling
			* Scheduled Job object (i.e. cron jobs)
			* Public Cloud Dashboard (w/ nightly runs to indicate Cloud performance metrics and integration+support)
			* Open for proposals now
	* Cloud Native Computing Foundation (CNCF)
		* The entire Kubernetes code base has been [donated to the CNCF](http://www.linuxfoundation.org/news-media/announcements/2016/03/cloud-native-computing-foundation-accepts-kubernetes-first-hosted-0)
		* The CNCF has an aim to be an open foundation, such as the Apache Foundation, to support containers and the ecosystem around it
			* Kubernetes is one of the key projects in the CNCF
		* The CNCF is part of the Linux Foundation

<br>
<div class="container">
  <div class="row text-center">
    <div class="col-xs-offset-2 col-xs-8 col-sm-offset-2 col-sm-5">
	  <a href="/images/kubecon-london-2016/state-of-union.jpg">
	      <img src="/images/kubecon-london-2016/state-of-union.jpg" class="img-responsive center-block"/>
	  </a>
    </div>
  </div>
</div>
<br>

----------


## **What is OpenStack's role in a Kubernetes world?**

* [Talk Abstract](https://kubeconeurope2016.sched.org/event/6BYC/what-is-openstacks-role-in-a-kubernetes-world)
* Summary
	* What are some benefits of running Kubernetes on OpenStack?
		* OpenStack gives you options for containers
			* i.e. If you decide to run Mesos, or Swarm in addition to Kubernetes, you have that ability
		* OpenStack is natively multi-tenant
		* You get more than just infrastrucutre, you also get a toolkit
			* i.e. Swift, Trove, Sahara etc.
		* OpenStack is future-proof
			* It's ready for whatever is next because it not only serves the needs of its users today, but it also addresses the needs they'll have tomorrow
			* New projects are always being started with an aim at tackling existing problems & adding new functionality
				* i.e. Kuryr for advanced networking, potential Unikernel support in the future etc.

----------


## **Hybrid Apps: Orchestrating Cloud-Native and Traditional Application Architectures**

* [Talk Abstract](https://kubeconeurope2016.sched.org/event/68yE/hybrid-apps-orchestrating-cloud-native-and-traditional-application-architectures)
* Summary
	* Apps can be cloud native, enterprise IT **is not**
	* Options for adopting Kubernetes for Enterprise
		1. Decomposability requires conversion into miroservices - this just isn't possible for most enterprises
		2. Leveraging CI/CD tooling - this is an *ok* model and works for most enterprises, but only solves the "push" problem in deployments
		3. Leverage a platform that understands the mechanics of "mixed era" apps - Kubernetes becomes a big part of this mixed solution, and Apprenda aims to help enterprises solve most of "the rest"
	*  Kubernetes support for Windows is coming, and Apprenda plans on being a big part of this effort


----------


## **Monitoring Microservices: Docker, Kubernetes, and GKE Visibility at Scale**

* [Talk Abstract](https://kubeconeurope2016.sched.org/event/6BUy/monitoring-microservices-docker-kubernetes-and-gke-visibility-at-scale)
* Summary
	* Key features any container monitoring solution should have:
		* High level exploration and resource usage by Services, Pods, etc.
		* Low level visibility
		* Avoid container instrumentation
		* Low overhead
		* Smart configuration
	* Simple options for monitoring
		* Kubernetes UI
			* Great starting point to see resource utilization and give a logical view of your cluster
		* Configure Heapster + Grafana
	* Low level visibility is hard and doesn't play well with containers (see below)
	* Solutions for containers
		* Sysdig
			* Aims to capture events
			* Filter, drilldown, explore & aggregate information
			* Is container native
			* Open Source
			* Supports Kubernetes v1.2

<br>
<div class="container">
  <div class="row text-center">
    <div class="col-xs-offset-2 col-xs-8 col-sm-offset-2 col-sm-5">
	  <a href="/images/kubecon-london-2016/linux-tools.jpg">
	      <img src="/images/kubecon-london-2016/linux-tools.jpg" class="img-responsive center-block"/>
	  </a>
    </div>
  </div>
</div>
<br>