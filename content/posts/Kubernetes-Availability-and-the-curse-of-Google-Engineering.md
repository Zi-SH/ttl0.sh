---
title: "Kubernetes, Availability, and the curse of Google Engineering"
draft: false
readingTime: true
date: 2025-09-27T12:00:00+00:00
tags: ["kubernetes", "container", "sysadmin"] 
categories: ["tech"]
summary: "\"I have 10 users so it's time to deploy Kubernetes\", but unironically."
author: ttl0
---
---
Despite what anyone seems to say, the verdict is out on Kubernetes. 

Programmers love it for its flexibility, especially when it comes to runners. Why shouldn't they, after all, when it allows them to stuff 10 pounds of shit in a 5 pound bag. Sysadmins tend to hate it; it's complicated, adds a lot of overhead, and no one seems to be a master of it. Members of both groups, however, generally love to agree on one argument against Kubernetes:

> You aren't Google and you aren't operating at Google's scale. Why are you using Google's orchestrator.

Or at least they really love to parrot that when the prospect is raised. 

If you're looking for an orchestrator at this point, perhaps it's because you're sick of container sprawl (Swarm is a joke) or have requirements such as scaling that you'd like to meet. If you'd allow me, I'll like to make an entirely different argument in favor of Kubernetes. One not of scale but of availability. Let's paint a picture. 

---

### Kernel Panic Attack

If you're running a Linux environment you likely already have several servers running various docker containers. You also likely have that one container Mike from Engineering asked for, likely either shoved into a server it doesn't belong on or (*painfully*) delegated to a dedicated system, free to idle with that 2 GB of RAM you gave Ubuntu because you felt guilty watching it choke during install. 

What's more, all these systems have to be managed and maintained. New systems are added for new containers, and existing servers built for particular workloads are either left with unused resources or patched with a hodgepodge of services of varying importance. 

All this is to say that you, my humble computer whisperer, are not a very *efficient* orchestrator. 

All these systems *also* need to be monitored, maintained, and updated. Unless you're in a startup and you're still running a catalogue of Awesome-Selfhosted suggestions, you might even have uptime requirements. This results in a cat-and-mouse game of patches and reboots, and you're left doing extra work orchestrating (see what I did there) these tasks. It's not like you wanted to go to your OSR campaign on Sunday. 

What if, my fellow Byte Bard, you could abstract the underlying system away. What if you could solve your optimization ordeal and maintenance misery with a small overhead and a bit of extra complexity. 

---

### Pods over easy

In the age of virtual machines and containerized workloads bare metal systems are generally a thing of the past (unless you're a very small business). VMs can be stateful or stateless, irreplacable or ephemeral, complex or mundane. They can be spun up and destroyed with ease, and Out-Of-Band Management is rarely a concern. Their configs are flexible and their entire existence can be altered on a whim.

You can spin up these VMs from a pool of resources on a Hypervisor, more effectively allocating resources such as CPU, Memory, and Diskc and place QoS on networking and CPU priority. You sacrifice a little overhead for the running of these systems but are awarded with an, overall, more effectively allocated pool of resources. These VMS almost entirely abstract the systems and software from the underlying hardware; if you have access to ESXi vMotion, Proxox Live Migration or something similar you've essentially cut the cord entirely. 

You can use Kubernetes to cut the cord between VM and application in the same way; with a little overhead you can allocate applications to resource pools, more effectively allocating resources from your cluster and placing limiters on resources to protect your workloads. You can force separations of workload to limit blast radius, limit resources to prevent a faulty apps from choking out systems, and restart and reorganize automatically or with a little bit of on-the-spot YAML. 

Best of all is that your hosts are now almost entirely stateless, ephemeral, and mundane, and your apps are now free-range. Updates or replacements are a simple as a quarantine and drain.

---

### Who let the Pods out?

All of this isn't without catches of course. You need storage external to your Hypervisor (iSCSI, SMB, NFS) that is sufficient for your workloads. You're also going to want decently reliable networking. A private git repository is also ideally, as you can store the many YAML you will write in your lifetime. These catches, however, aren't much of an ordeal if you're running any modern infrastructure in any modern company that cares an iota about tech. The fact that you're here, now, positing the implementation of a orchestrator, does imply that you or your organization fall into this category. So let's talk about the trench work here.

First you need to consider what your infrastructure needs to look like. How many pods do you need to run, what is the resource requirements of your largest pods, and do you have any services such as databases that benefit from K8S concepts like replicas and deployments? You also need to consider how *H* your *HA* needs to be as this will change your architecture. Be mindful that a minimum HA cluster of Control Nodes (the nodes with the Homunculus inside) is 3; this is due to etcd and other various quorum pills that you have to swallow. You can have more than 3, but you probably don't need it unless you plan to host your servers in a war zone. 

An HA cluster will also require a virtual IP to share between them. Your API server will listen on this IP, and everything Kubernetes will rely on that IP-bound API port, so you're gonna want it tight. HAProxy/Keepalived are time-tested options you can run directly on your masters and there are other, more modern and shiny, possibly even Kubernetes native, solutions you can consider. 

Your workers are entirely expendable if your Control Node game is solid and you correctly offload your stateful data to a external, networked storage. All your pods should make claims on Persistent Volumes leveraging iSCSI, NFS or SAMBA depending on your workload, and zero data should be hosted on the local PV. When these boxes are checked you can treat your Worker VMs as expendable compute nodes. 

---

### Do Androids Dream of Stateful Sheep

Given a infrastructure of 3 Control and 3 Worker VMs, loosely based on my previous ramblings, your duties shift from the Sysadmin Daycare you previously provided to one of monitoring and code writing.

Your Control and Worker nodes are ephemeral. If one of the former fails, the remaining nodes will continue to handle the system with grace until the issue is addressed. If one of the latter fails, you can simply blow it away and add a new VM to the cluster without any tweaking or testing needed for your applications. Your nodes can be taken down at any time for maintenance, with as little as a few seconds of downtime as the pod is reschedule to a node not being touched. Your storage follows your pods, now Electric Sheep herded to sufficiently ample pastures of compute. 

YAML is now your new best friend. Twice as much YAML as Docker, but equally as recyclable. Your services (pods) will use YAML, your storage will use YAML, your networking will use YAML, and all your security will leverage.... YAML. This will be painful at first, but as you build out your environment and catalog these files your entire cluster will become ephemeral. Like tear in rain. 

---

### Go with the workflow

Adapting your existing container infrastructure to an orchestration setup appears daunting but can be done piecemeal. I preach reconsideration of Kubernetes outside of "Google Scale" not as a contrarian, however fun that may be, but as someone who has done this exact configuration and benefited from the extended flexibility provided by *ephemeral* hosts, orchestrated optimization, and containers powered by networked storage. I can even argue that it's improved my quality of life, fixed my sleep schedule, and even saved Christmas (once).

To those pondering this alleged miracle I have exposed your neurons to, I would like to offer a few words of caution for you to consider. Mostly because this sort of infrastructure configuration is different from the average setup a Sysadmin will experience and it requires a bit of reading and prep. 

- A full Infrastructure-As-Code environment that can be burned down and reconstructed in an hour requires little in terms of backups for the hosts. You'll instead find yourself attempting to backup against the PVCs mapped to containers using concepts such as sidecars or cronjobs, also YAML goodies. If your PVC supports volume snapshots this is even easier, allowing frozen-in-time block snapshots of your PVC. 

- For monitoring, one cannot go wrong with the Kubernetes Dashboard. Grafana can also be quite useful for deep analytics of resource utilization from a pod's lifetime. Orchestrated pods not bound to a host are far less likely to experience issues given an unresponsive pod will be killed and rebuilt, but monitoring is always one of those things you don't need until you do. 

- You will often find the concept of Infrastructure-As-Code difficult to sell, even to those *unknowingly already doing it*. This infrastructure will also make onboarding additional admin more difficult as they will have to be brought up to speed. Feel free to send them this post so they can process docker compose as what it is. 


I hope these ramblings of a Sysmadman have inspired consideration of an orchestrated environments for stateful work loads, or at least had you reconsider. You aren't Google and you don't Google's scale; that doesn't mean you can't have their *uptime*. 
