name: nomad-concepts
class: title, shelf, no-footer, fullbleed
background-image: url(https://hashicorp.github.io/field-workshops-assets/assets/bkgs/HashiCorp-Title-bkg.jpeg)
count: false

# HashiCorp Nomad
## Advanced Concepts and Architecture

![:scale 15%](https://hashicorp.github.io/field-workshops-assets/assets/logos/logo_nomad.png)

???
Nomad is a highly advanced service scheduler and manager.  Within this slide deck, we'll be reviewing some of the more advanced concepts and architecture behind Nomad.

---
layout: true

.footer[
- Copyright © 2019 HashiCorp
- ![:scale 100%](https://hashicorp.github.io/field-workshops-assets/assets/logos/HashiCorp_Icon_Black.svg)
]

---
name: slides-link
# The Slide Show
## You can follow along on your own computer at this link:
### tbd

???
Here is a link to the slides so you can follow along, but please don't look ahead!

Hidden:  Custom diagrams can be found at Lucidchart
https://www.lucidchart.com/documents/edit/6cf954c6-16e9-4827-9d74-98417ba74444/0_0?beaconFlowId=D0003F1058D413A2

---
name:  Section Prerequisites
# Prerequisites
This deck assumes the following of the audience:
* Familiarity with Nomad Scheduler
* Familiarity working with HashiCorp Configuration Language
* A general understanding of container operations is recommended

???
This slide deck focuses on the concepts and architecture of Nomad.  Therefore the audience should have some basic understanding of the Nomad applications.

---
name:  High level Nomad Review
# Nomad Review
Let's review a few high level concepts about Nomad:
.smaller[
* Nomad is a flexible, lightweight, performance, easy to use orchestrator
* Nomad is used to deploy and manage containers and legacy applications simultaneously
* Nomad works across data centers and cloud platforms, providing universal scheduling
* Nomad manages services, batch functions, as well as global system services
]

???
Nomad runs as a single binary in just about any environment, making it one of the easiest and lightweight service scheduler and manager available.  It can be used to deploy both container applications, as well as legacy applications such as Java or raw executables.  Being an independent function, Nomad can run and communicate across data centers, and cloud platforms.  Truly cloud agnostic.  Lastly, Nomad can manage individual services, batch functions, or even global system services such as monitoring functions.

---
name:  Common Nomad Terms
# Nomad Terms
Let's review some terms and definitions regarding Nomad

.smaller[
* Nomad clusters consist of nodes running the Nomad binary, both Servers and Clients
* Servers provide the intelligence (scheduling, allocation) to the cluster
* Clients run the Nomad agent which registers with the server and executes local tasks.
* Jobs are submitted by users and represent the desired state of the associated workloads
* Drivers are used by Nomad to execute tasks (i.e., Docker, QEMU, Java, etc.)
* Tasks are the smallest unit of work executed by task drivers
]

???
Nomad operates as clusters of nodes, with anywhere from 3-5 server nodes, and an unconstrained number of client nodes.  All nodes run the same Nomad binary.  The Nomad server nodes provide the brains and intelligence to the cluster, performing all scheduling and allocations, while the clients execute the tasks as directed by the server cluster. Jobs are used to describe the desired state of the workloads.  This is what the servers and clients work towards.  Nomad also offers several drivers to help execute the defined tasks, getting the defined workloads into the desired state.

---
Name:  Nomad Layout and Comms
# Nomad Communications
.left-side[
* 3-5 Server Nodes
* Server Leaders Replicate to Followers
* Server Followers Send Request and Allocations to Server
* Clients Communicate with Servers over RPC
]
.right-side[
    ![:scale 60%](https://www.nomadproject.io/assets/images/nomad-architecture-region-a5b20915.png)
]

???
Let's jump right in with the communications among the Nomad nodes.  Within the Server Cluster, we have a Leader, and we have Followers.  The Leaders are elected via quorum (which is why it is important to have 3-5 nodes) using the Consensus, based on RAFT.  The Leader of the servers makes all allocation decisions, and distributes to Followers.  Clients pull allocation and task assignments via RPC from each Server.

---
Name:  Nomad Scheduler
# Nomad Scheduler Initiation
.left-side[
An Evaluation is "Kicked Off"
.smaller[
* New Job
* Job Update
* Job or Node Failure
]
An Evaluation can be sent to any of the Server nodes, but eventually they all make it to the Evaluation broker that resides on the leader.  Until queued, the evaluation is in 'pending' state
]
.right-side[
    ![:scale 60%](images/Nomad_Evalation_Kickoff.png)
]

???
Everything starts with something, regardless of your technical, moral, or spiritual beleifs.  With Nomad, we deal witih Evaluations to determine if any work is necessary.  What kicks off that evaluation can be a new job definition, and updated job definition, or some change to the infrastructure.


---
name:  Nomad Evaluation
# Nomad Evaluation
.left-side[
Once the Evaluation Broker recieves the evaluations, the Broker queues the changes in order based on prirority.

Scheduler on Follower Nodes pick off the queue and start planning!
]
.right-side[
    ![:scale 60%](images/Evaluation_Queue.png)
]

???
We're talking about Evaluation, not Evolution. Here the evaluation Broker, residing on the leader node, manages the queue of pending evaluations.  Priority is determined based on Job definition, and the Broker ensures that somebody picks up the evaluation for processing.  Once the evaluation is picked up by a Scheduler, the planning begins!

---
Name:  Scheduler Function
# Scheduler Operations

All Servers run Scheduling Functions
* One Scheduler per CPU core by default
* Four Default Schedulers Available
    * Service Scheduler optimized for long-lived services
    * Batch Scheduler for fast placement of batch jobs
    * System Scheduler for jobs to run on every node
    * Core Scheduler for Internal Purposes Only

???
The Scheduling of the job is really the work that all sever nodes perform.  By default, each server node runs one scheduler per CPU core.  Depending on the job at hand, the Server chooses the proper scheduler, either for standard services, batch jobs, system level jobs, or internal jobs.

What if two server nodes pick off the same evaluation/job?  We'll get to that later.

---
Name:  Scheduler Function Part 2
# Scheduler Processing
Now that the Scheduler has the job, let's look at what the it does...
.smaller[
1.  Identify available resources/nodes to run the job
2.  Rank resources based on bin packing and existing tasks/jobs
3.  Select highest ranking node, and create allocation plan
4.  Submit allocation plan to leader
]
???
Now that the Server node has picked off the job, there's a set of operations it has to run through.  First it identifies the potential nodes, or available resources, that could accept the job.  So ignore nodes that are out of resources, or no longer exist.  Next take a look at the ideal nodes, based on bin packing and existing tasks.  This is a very important step.  Bin packing ensures the most efficient usage of the resources.  Taking existing tasks into account minimizes co-locating tasks on the same servers.  Based on these factors, the highest ranking node is chosen, the allocation plan is created, and submitted back to the Leader.

---
Name:  Plan Queue Processing
# Plan Queue Processing
Back to the leader now...
.smaller.left-side[
5.  Evaluate all submitted allocation plans
6.  Accept, reject, or partially reject the plan
7.  Return response to Scheduler for implementation, reschedule, or terminate
8.  Scheduler Updates status of evaluation and confirms with Evaluation Broker
9.  Clients pick up allocation changes and act!
]
.right-side[
    ![:scale 60%](images/Queue_Processing.png)
]

???
Now that the evaluations have been processed, and allocations proposed, it's back to the leader for the final determination.  This allows the all pending plans to be prioritized and eliminate any concurrency if it exists.  The leader will either accept or reject (or partial reject) the plan.  The Scheduler can chose to reschedule or terminate the request.  The Scheduler updates the Evaluation Broker with the decision, and clients pick up any changes deemed necessary

Question from before:  What if two server nodes pick off the same evaluation/job? This step manages that situation.  Multiple schedulers can pick up the same job and propose an allocation plan.  The Leader, with all of the plans, determines any overlap and rejects any duplicate or overlapping allocations.

---
Name:  End to End Flow
#  Flow Recap
![:scale 90%](images/Nomad_Overall_Flow.png)

---
Name:  A Focus on Priority
# Let's Talk Priority
Priority Processed During Evaluation and Planning

What if higher priority jobs are scheduled?

.larger[PREEMPTION]

???
We all struggle with priority.  How does Nomad deal with the inevitable situation where a higher priority job is scheduled and resources are limited?  Preemption!
---
Name:  Preemption
# Preemption

Preemption allows Nomad to ensure higher priority (based on Job definition) are scheduled first.

.smaller[
* Occurs only when resources are constrained
* Only jobs with a priority delta > 10 evaluated
* Lowest priority jobs evicted first
* Output of 'Plan' identifies any preemptive actions
]

---





    Preemption
    - kill existing allocations in favor of higher priority jobs
    - Every job has a priority which is evaluated during planning
    - ONly allocations from jobs with a priority delta > 10 are eligible for preemption - minimize cascading preemptions
    - Lowest priority jobs are evicted first
    - alloc status to show preemtedallocs (jobs prempted) adn preemptedbyallocID (ID of allocation that took priority)
    - Nomad 'Plan' displays any preeempmtion that is necessary

















---
name:  notes/scratch
Evaluations start in pending state
Evaluation broker runs on leader server
    - Broker manages queue of pending evaluations, orders priority, and ensures at least once delivery


    Schedulers provide an allocation plan - evict/update/create based on existing state
    Two phase commit 
        - Find available resources/nodes
        - Rank resources based on bin packing and existing task groups to avoid colocating
        - Highest ranking node selected and added to allocation plan
        - Scheduler submits plan to leader who adds plan to the plan queue
            - plan queue manages pending plans, priority ordering, and concurrency
            - Leader manages reuqests by schedulers as multiple scheudlers can be running concurrently
            - Leader accepts or rejects (or partial rejects) plan, scheduler can replan or terminate
            - ONce finished, scheduler updates status of evaluation and confirms with evaluation broker
            - All allocations are picked up and executed by clients
    =>  Need a better image for this

---
## Part 1 of the Nomad OSS Workshop

![:scale 15%](https://hashicorp.github.io/field-workshops-assets/assets/logos/logo_nomad.png)

???
This is a title slide for the skeleton of a first part of a Nomad OSS workshop.

---
name: nomad-logo
class: col-3
# This is the Nomad logo (using 3 img tags)
<p>
  <img style="width:200px;height:200px;" src="https://hashicorp.github.io/field-workshops-assets/assets/logos/logo_nomad.png">
</p>

<p>
  <img style="width:200px;height:200px;" src="https://hashicorp.github.io/field-workshops-assets/assets/logos/logo_nomad.png">
</p>

<p>
  <img style="width:200px;height:200px;" src="https://hashicorp.github.io/field-workshops-assets/assets/logos/logo_nomad.png">
</p>

???
This slide has the Nomad logo.

---
layout: true

.footer[
- Copyright © 2019 HashiCorp
- ![:scale 100%](https://hashicorp.github.io/field-workshops-assets/assets/logos/HashiCorp_Icon_Black.svg)
]

---
name: agenda
# Agenda

1. What is Nomad?
2. Key Features
3. How Nomad Compares to Other Tools
4. Getting Started

???
This is our agenda slide.

---
name: what-is-nomad
class: col-2
layout: false
# What is Nomad?

* Nomad is a flexible workload orchestrator.
* It supports containerized and legacy applications.
* It uses a declarative syntax for specifying workloads.
* It uses bin packing to efficiently schedule jobs and optimize resource utilization.

Here is some more content.

And some more
* bullet 1
* bullet 2

???
This is a regular slide with content


# Notes/Scratch

What is Nomad?
- Flexible, lightweight, performance, easy to use orchestrator
- deploy and manage containers and legacy applications simultaneously
- Run all types of workloads (service, batch, system)

Benefits of Using an Orchestrator
- Efficient use of resources through bin-packing
- Developer Self-Service describing how their application should be run
- Focus jobs/tasks on resources suited for the purpose (ML, GPU, AI, Data Sciences etc.)
- Automate upgrades, application balancing
- Dynamically scale and migrate resources on demand
- Scale services independently instead of scaling entire applciation platforms


Nomad Application
- Single binary that can run as client and/or server
- Recommended at least 3 or 5 servers in a cluster
- Agent runs on host machines, performs health checks, executes tasks and jobs
- Not cloud or location specific - Schedule applications across resources regardless of cloud or location

Multi-region Federation
Nomad can distribute workloads across mutiple regions for resilience, availability, fault tolerance
Multi-Cloud fedeation
Nomad can distriubte workloads across multiple cloud providers (AWS, GCP, Azure) for cost savings and protection against lock-in

>> Need a side by side comparison between a container definition using K8s and a definition using Nomad

Large clusters and federation help provide a unified view /full picture of all of the different workloads regardless of where they live.

Agent can run in client or server node
recommended at least a cluster of 3 or 5 for reliability
Nomad client runs on host machines, performs health checks, and executes tasks

Deploy and manage any containerized, legacy, or batch application
Portable, scalable, efficient resource utilization wihtout the container investment.  Usually containerizing legacy applications also results in less efficient containers, resulting in one or two containers per machine.

Increase density of running applications through bin packing
run containers and legacy applications side by side
developers can self service their own applications

native multi-region/multi-cloud - clusters across regions/clouds

What is Nomad?

Native multi-region, multi-cloud capabilities
Accelerate developer velocity and increase operational efficiency

Nomad use cases:
Container Orchestration
- Container orchestrators allow for deploying, managing, and scaling of containerized apps.  Nomad supports Docker. lxc, rkt, and others
Legacy Applications
- Not all apps are containerized, and it can be expensive and complicatded to containerize them all.  Not always practical.  Nomad birngs core orchestration benefits to legacy apps (automation, efficiency, resilience, etc.)
Batch Workloads
- Batch workloads are common in high performance computing clusters.  Nomad runs batch applications and offers GPU support to enable ML, AI, data science, and other intensive workloads

Exmaples of workloads:
Batch:
Short lived workloads
Daily reports, transactions, billing invoices
Service:
Long living workloads
Business-Critical Applications
Customer-facing webapps
Databases
Ssytem:
Backgorund tasks
- Logging/Monitoring/
- Background processes

Native Fedaration Capabilities
- Create multi-region, multi-cloud clusters with a single CLI command
- multi-cloud and multi-region federation supported OOTB with nomad
- Enables the cost savings of multi-cloud, fualt tolerance of multi-region



Accelerate Developer Velocity
- Nomad's job files are where developers configure their apps.  Running Docker?  Write "docker."  Need to scale up?  Just set your desired count.  Seamless, intuitive, fast

Single binary, native federation, built in UI
- Scale applications across the entire organization

Built in ACL system for role segmentation at scale
Multiple upgrade strategies
Apache Spark and Nvidia GPU support

 
 
 Typically any agent running in client mode must be run with root level privilege. Nomad makes use of operating system primitives for resource isolation which require elevated permissions. The agent will function as non-root, but certain task drivers will not be available.

 The output shows our Node ID, which is a randomly generated UUID, its datacenter, node name, node class, drain mode and current status. 

 The output shows our own agent, the address it is running on, its health state, some version information, and the datacenter and region. Additional metadata can be viewed by providing the -detailed flag.

 By gracefully leaving, Nomad clients update their status to prevent further tasks from being scheduled and to start migrating any tasks that are already assigned. Nomad servers notify their peers they intend to leave. When a server leaves, replication to that server stops. If a server fails, replication continues to be attempted until the node recovers. Nomad will automatically try to reconnect to failed nodes, allowing it to recover from certain network conditions, while left nodes are no longer contacted.

 Nodes that are confifgured to leave the cluster, are dgone.  Nomad understands that.  However, nodes that failed, are regarded as failed and Nomad will automatically attempt to reconnect.

Nomad is mainly concerned with scheudling and running 'jobs'.  A job is the declarative specificaiton of what Nomad beleisves it should run.  Jobs can be specified in HCL or JSON.  

example job is a redis container using Docker

nomad status example

nomad alloc status <job id>

nomad alloc logs <job id> redis

For now, edit the example.nomad file to update the count and set it to 3. This line is in the cache section around line 143.

nomad job plan example.nomad

nomad job run -check-index <job modify index> example.nomad

nomad status example

Edit the example.nomad file and change the Docker image from "redis:3.2" to "redis:4.0". This is located around line 261.

# Configure Docker driver with the image



Nomad nodes utilize Gossip for communication

Single Data Center Architecture:
https://www.nomadproject.io/assets/images/nomad-architecture-region-a5b20915.png

Multi-Data Center Architecture
https://www.nomadproject.io/assets/images/nomad-architecture-global-a8f14b78.png

Regions are fully independent from each other, and do not share jobs, clients, or state. 

Using Gossip, users can submit jobs to any region or query the state of any region

requests are forwarded tot the appropriate server to be processed - Data is NOT replicated between regions.

Servers in one region create a 'consensus group'
A consensus group should consist of 3 or 5 servers.  Shouldn't have more than 7 servers.  Clients are unlimited
Leader processes all queries and transactions, but all servers make scheduling decisions in parallel. Leader coordinates to ensure clients aren't oversubscribed.

They are loosely-coupled using a gossip protocol, which allows users to submit jobs to any region or query the state of any region transparently. Requests are forwarded to the appropriate server to be processed and the results returned. Data is not replicated between regions.

clients communicate using remote procedure calls for heartbeats and status, and to receive allocations from servers

Job represents a desired state for a set of tasks
Servers schedule the tasks ensuring maximum resource utilization - using bin packing
    - bin packing makes use of all available resources of a machine without exhausting any dimension or violating any constraint


Deep dive on available constraints - Constraints can be technical requirements based on hardware features such as architecture and availability of GPUs, or software features like operating system and kernel version, or they can be business constraints like ensuring PCI compliant workloads run on appropriate servers.

Nomad uses the go-plugin architecture for devices and task drivers
    - base plugin provides core plugin functionality (version and configuration)
    - Task drivers execute workloads and operate independently of the client (Nomad will auto-launch the plugin if it fails, and the client can be restarted without restarting the plugin) - docker, Java, Qemu, Exec, LXC, Firecracker
    - Device Drivers enable scheduling tasks with GPUs (CPU/Mem/Network supported standard) (Nomad will launch the plugin if it fails)

Nomad Scheduling:
https://www.nomadproject.io/assets/images/nomad-data-model-39de5cfc.png
assigns tasks from jobs to client machines
- Evaluations occur every time the external state changes - new job, updated job, client report
https://www.nomadproject.io/assets/images/nomad-evaluation-flow-7629d361.png

Evaluations start in pending state
Evaluation broker runs on leader server
    - Broker manages queue of pending evaluations, orders priority, and ensures at least once delivery

Nomad servers run scheduling workers (one per CPU core by default)
    - Scheduling workers process evaluations
    - Workers dequeue evaluations from the broker and invoke appropriate scheudler
        - Service scheduler optimized for long lived services
        - Batch scheduler for fast placement of batch jobs
        - System scheduler runs jobs on every node
        - Core scheduler for internal purposes only

    Schedulers provide an allocation plan - evict/update/create based on existing state
    Two phase commit 
        - Find available resources/nodes
        - Rank resources based on bin packing and existing task groups to avoid colocating
        - Highest ranking node selected and added to allocation plan
        - Scheduler submits plan to leader who adds plan to the plan queue
            - plan queue manages pending plans, priority ordering, and concurrency
            - Leader manages reuqests by schedulers as multiple scheudlers can be running concurrently
            - Leader accepts or rejects (or partial rejects) plan, scheduler can replan or terminate
            - ONce finished, scheduler updates status of evaluation and confirms with evaluation broker
            - All allocations are picked up and executed by clients
    =>  Need a better image for this

    Preemption
    - kill existing allocations in favor of higher priority jobs
    - Every job has a priority which is evaluated during planning
    - ONly allocations from jobs with a priority delta > 10 are eligible for preemption - minimize cascading preemptions
    - Lowest priority jobs are evicted first
    - alloc status to show preemtedallocs (jobs prempted) adn preemptedbyallocID (ID of allocation that took priority)
    - Nomad 'Plan' displays any preeempmtion that is necessary


Consensus
- NOmad uses Consensus to identify the leader in the cluster
Because of the nature of Raft's replication, performance is sensitive to network latency. For this reason, each region elects an independent leader and maintains a disjoint peer set. Data is partitioned by region, so each leader is responsible only for data in their region. When a request is received for a remote region, the request is forwarded to the correct leader. This design allows for lower latency transactions and higher availability without sacrificing consistency.
Gossip
Used for communication across regions



* Allocations aer used to d
jobs - submitted by users and represent a desired state of workloads - bounded by constraints and resources
nodes - hosts the Nomad client and any tasks allocated to that client
allocations - used to declare that a set of tasks should be run on a particular node
evaluations - Determines the appropriate allocations and initiates scheduling