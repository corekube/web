+++
date = "2016-03-11T09:00:00-08:00"
tags = ["KubeCon", "Performance", "Trusted Computing", "Hardware"]
title = "KubeCon London 2016 - Day 2"
slug = "kubecon-london-2016-day-2"
authors = "Mike Metral"
+++

Hello **again** from London!

Today was the second & final day of [KubeCon](https://kubecon.io/) Europe in London, England, and we've got even more great content from the great folks in the community.

Lets review a few summaries & notes on a subset of the talks given today.

<!--more-->

<br>
<div class="container">
  <div class="row text-center">
    <div class="col-xs-offset-2 col-xs-8 col-sm-offset-2 col-sm-5">
	  <a href="/images/kubecon-london-2016/kubecon2.jpg">
	      <img src="/images/kubecon-london-2016/kubecon2.jpg" class="img-responsive center-block"/>
	  </a>
    </div>
  </div>
</div>
<br>

# **Outline**

* [Day 2 Keynote: Pushing Kubernetes Forward]({{<ref "#day-2-keynote-pushing-kubernetes-forward">}})
* [Integrated Trusted Computing in Kuberentes]({{<ref "#integrated-trusted-computing-in-kuberentes">}})
* [My Microservices are not all stateless!]({{<ref "#my-microservices-are-not-all-stateless">}})
* [Kubernetes Hardware Hacks: Exploring the Kubernetes API Through Knobs, Faders, and Sliders]({{<ref "#kubernetes-hardware-hacks-exploring-the-kubernetes-api-through-knobs-faders-and-sliders">}})


## **Day 2 Keynote: Pushing Kubernetes Forward**

* [Talk Abstract](https://kubeconeurope2016.sched.org/event/68lU/day-2-keynote-pushing-kubernetes-forward)
* Summary
	* Where is CoreOS pushing Kubernetes towards?
		* Aim to simplify the deployment and configuration of clusters
		* They want to increase the scale of clusters throughout the stack
		* Want to establish security based on good practices
		* rkt engine powering Kubernets nodes
		* Standards to ensure portability
	* They want to answer a few key questions for Kubernetes
		* How do we install it all?
		* How do we upgrade it all?
		* The Answer: [Monokube](https://github.com/polvi/monokube) (*currently a prototype*)
			* Monokube is compilation of all Kuberentes components & dependencies, including etcd, into a single binary
			* What are the advantages of using Monokube?
				* It bootstraps requirements down to woring SSsH
				* Rolling updates for Kubernetes itself
				* Kubelet version controlled by the API
				* Help is wanted - the goal is get this working in v1.3
	* Increasing scale on Kubernetes
		* [Recent work](https://coreos.com/blog/improving-kubernetes-scheduler-performance.html) provided a 10x (closer to 14x actually) improvement in the scheduler throughput
		* etcd v3 in Kubernetes
			* etcd was originally theorized to be the bottleneck in Kubernetes - currently its not necessarily the case, but with time, it could affect performance
				* v3.0 is intended to help scale Kubernetes to 1000s of nodes
	* rkt Powered Kubernetes aka *rktnetes*
		* rkt: a new container runtime aimed at being secure & simpler focused on supporting a lot of the same abstractions that exist in Kubernetes i.e. Pods are a 1st class citizen
		* rktnetetes is nearly complete
			* 80% of end-to-end tests passing
			* cAdvisor integration is in progress
	* Security
		* TPM Log
			* TPM's usually work at the bootloader level, but CoreOS wants to extend this all the way up to the application layer
			* They can store a last-known state for containers to secure your stack, as well as, verify the images themselves
		* TLS bootstrap of Nodes to verify that they can in fact join the Kubernetes cluster
	* Industry Movement
		* Open Container Initiative
			* Creating technical standards for containers
			* Started with runC and a runtime spec
			* Large mandate to standardize on an image format - *very much still in progress*
		* Cloud Native Computing Foundation
			* Coordinate the promotion of Cloud Native architectures
			* A home for Cloud Native OSS projects like Kubernetes
			* Provie shared resources to projects like videos & other educational material
	* Multiple Image Formats in v1.3 API
		* Today, Kubernetes only supports the Docker Image Format & naming
		* Use cases for other formats
			* OCI Image Format
			* tar archive chroots
			* jar?
			* static binary?
		* Support signing and content verification

<br>
<div class="container">
  <div class="row text-center">
    <div class="col-xs-offset-2 col-xs-8 col-sm-offset-2 col-sm-5">
	  <a href="/images/kubecon-london-2016/coreos-security.jpg">
	      <img src="/images/kubecon-london-2016/coreos-security.jpg" class="img-responsive center-block"/>
	  </a>
    </div>
  </div>
</div>
<br>

----------


## **Integrated Trusted Computing in Kuberentes**

* [Talk Abstract](https://kubeconeurope2016.sched.org/event/67eX/integrated-trusted-computing-in-kubernetes)
* Summary
	* Secure runtimes **requires** a secure base
	* How do we trust a system?
		* We need mechanisms to trust & verify the underlying components
	* Trusted Computing
		* A set of technologies implemented by the [Trusted Computing Group](http://www.trustedcomputinggroup.org/) in the early 2000's
		* However, these tools haven't been widely deployed outside of specialized usecases
		* Basis of Trusted Computing
			* Trusted Platform Module (TPM): is able to sign & encrypt data with crypotographic keys, which never leaves the chip
			* One of the keys in the TPM is the *Endorsement Key*, and is a unique per-system identity burnt into it at manufacturing
			* Another function of the TPM is to verify system state by crypotographically hashing blocks of data, starting at firmware, and moving onto the bootloader, kernel etc., all the way up the stack
				* Thus, the TPM is able to provide *attestation*, a cryptographically verifiable state of the system
	* How does trusted computing fit into Kubernetes?
		* Basic model: Allow Kubernetes to verify system state before providing access (via the hashes performed by the TPM)
		*  Two-pronged approach:
			1. Authentication Controller - usage & interaction with initial authentication that is TPM-based; however, the main bottleneck here is that attestation is slow, so after initial auth, Secrets are consumed to interact with the system via SSL/TLS certs
			2. Admission Controller - on certain API calls, we can provide additional layers of TPM-based validation & verification. For example, this is used to validate that Nodes are allowed to join the cluster and gain access to its data
	* Can trusted computing go beyond the Authenticaton & Admission controller functions?
		* Yes - TPM's can be used at any time you wish. Therefore, you can extend this security layer to extend into measuring initial container state to verify that the container launched is in fact the one we expect
			* This extensiveness creates a cryptographically verifiable audit trail
	* Currently, all of this work is mainly aimed at bare-metal, but work to abstract it and be able to consume it on public cloud providers is on the roadmap, but requires providers to provide support for TPM-based functionality
		* Work & code for all of this functionality can be found [here](https://github.com/mjg59/kubernetes)


----------


## **My Microservices are not all stateless!**

* [Talk Abstract](https://kubeconeurope2016.sched.org/event/6BZy/my-microservices-are-not-all-stateless)
* Summary
	* Microservices:
		* Are lots of processes
		* Communicate over the network
		* Come in different languages
		* Each service uses a specific data store
	* 12 Factor Apps
		* aka The 'Heroku' way of running apps
		* Are strictly 'stateless'
		* Forces someone else to deal with the state problem
		* Leads to a fragmented deployment system
	* Stateful data will in fact come and be managed by the right tools and confidence, even though the current status quo is to be weary about running apps like databases in a container
	* Kubernetes to the rescue for stateful data!
		* It treats volumes as 1st class citizens
		* It can also move volumes around the cluster
		* Treats the container and the storage as a single, atomic unit - they're not separate entities
	* PersistentVolumes
		* A storage resource that Kubernetes knows about
		* Lives outside of the lifecycle of any pod
		* Abstracts the implementation of specific storage providers
	*  PersistentVolumeClaim
		* The linkage between the fact that some volume storage exists, and the fact a Pod will consume it
		* Kubernetes will automatically pick a volume meeting the requirements
		* The volume will remain alongside the Pod while the claim exists
	* Lifecycle of a Volume
		1. The admin creates a volume resource
		2. Create a PersistentVolume manifest to tell k8s about it
		3. Run a Pod with a PersistentVolumeClaim
		4. K8s will match the volume to the claim and mount it onto the Pod
		5. User deletes the PersistentVolumeClaim (or Pod)
		6. The PersistentVolume is now 'released'
		7. The PersistentVolume can now be 'reclaimed'
	* PersistentVolume Provisioning (*experimental feature*)
		* Automatically creates volumes to be 'cliamed'
		* Workflow is now reversed comapred to the previous lifecycle established
			* Run a Pod with a PersistentVolumeClaim
			* k8s will automatically provision a volume
			* k8s will match the volume to the claim and mount it onto the Pod
	* Flocker is an abstraction for other volume storage
		* Distributed volume orchestration
		* Supports many types of storage
		* Why use Flocker?
			* Storage Support
	* Flocker Use Cases
		* Reblance the volumes in a cluster
		* Drain a node
		* HA when a node fails
		* Speed up distributed data recovery
	* Future Developments
		* Volume Hub: SaaS-like platform to visualize your volumes
		* dvol: "like git for data"
		* Flocker
		* All together, the flow looks like this:
			* dvol <-------> Volume Hub <---------> Flocker


----------

## **Kubernetes Hardware Hacks: Exploring the Kubernetes API Through Knobs, Faders, and Sliders**

* [Talk Abstract](https://kubeconeurope2016.sched.org/event/6Bl3/kubernetes-hardware-hacks-exploring-the-kubernetes-api-through-knobs-faders-and-sliders)
* Summary
	* The API spec can be located at http://localhost:8888/swagger-ui and will be the basis for the hardware hack discussed
	* Most objects have an operational loop on objects that moves between states as such:
		* Act -> Observe -> Diff (cyclicly repeat)
	* API Clients
		* Not in the best state, and is still a work in progress
	* ReplicationControllers & rolling-updates have been deprecated
		* ReplicaSets have replaced ReplicationControllers
		* Deployments have superceded how updates to your microservices should be performed
	* The Hardware Hack
		* A way to visualize the cluster and the microservices running on top of it
			* It uses an Ableton MIDI controller, where it receives the input from the controller's faders & sliders via MIDI, and the MIDI input gets converted into a Kubernetes API request to alter the state of the cluster
			* The colors denote differnet microservices, and their quantity signify the number of Pods

<br>
<div class="container">
  <div class="row text-center">
    <div class="col-xs-6 col-sm-offset-1 col-sm-4 nopadding">
		  <a href="/images/kubecon-london-2016/hardware-hack-1.jpg">
		      <img src="/images/kubecon-london-2016/hardware-hack-1.jpg" class="img-responsive"/>
		  </a>
	  </div>
    <div class="col-xs-6 col-sm-4 nopadding">
	  <a href="/images/kubecon-london-2016/hardware-hack-2.jpg">
	      <img src="/images/kubecon-london-2016/hardware-hack-2.jpg" class="img-responsive"/>
	  </a>
    </div>
  </div>
</div>
<br>