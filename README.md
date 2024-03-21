# KUBERNETES
## Control Plane
A kubernetes control plane node is a server running collection of system services that make up the control plane of the cluster.

The simplest setup runs one control plane node. For production multiple control plane nodes configured for HA is vital (3 or 5).
### Api service
The API server is the Grand central of Kubernetes. **All communication between all components, must go through the API server.**

It exposes a RESTful API that you POST YAML configuration files to over HTTPS. These YAML files, which we sometimes call *manifests*, describe the desired state of application. The desired state includes things like which container image to use, which port to expose, how many Pod replicas to run.

All request to the API server are subject to authentication and authorization checks.

### The cluster store
The cluster store is only stateful part of the control plane and persistently stores the entire configuration and state of the cluster. As such, it's vital component of every Kubernetes cluster - no cluster store, no cluster.

The cluster store is currently based on `etcd` database. As it's the single source of truth for a cluster, you should run between 3-5 etcd replicas for high availability.
etcd uses RAFT consensus algorithm to accomplish this.
### The controller manager and controllers
The controller manager implements all the background controllers that monitor cluster components and respond to events.

It is controller of controllers meaning it spawns all the independent controllers and monitors them.

Example of controllers: Deployment controller, Stateful controller, ReplicaSet controllers.

Desired state -> Observed state

The logic implemented by each controllers is as follows and it is heart of the kubernetes and declartive design patterns.
1. Obtain desired state
2. Observe current state
3. Determine differences
4. Reconsile differences

Each controller is also extremely specialized and only interested in its own little conner of kubernetes cluster. 
### The scheduler
The scheduler watches the API server for new work tasks and assigns them to appropriate healthy worker nodes. Behind the scenes, it implements complex logic that filters out nodes incapable of running tasks, and then ranks the nodes that are capable of running tasks. The node with the highest ranking is selected to run the task.

When identifying nodes capable of running tasks, the scheduler performs various predicate checks. This include is the node tainted, are there any affinity or anti affinity rules, is the required port available on the node, does it have sufficient resources, etc.

The scheduler is not responsible of running tasks. just picking the nodes to run them. A task is normally a POD/CONTAINER.
### The cloud controller manager
If you are running your cluster  on a supported public cloud platform (AWS, AZURE), then control plane will be running the cloud control manager. Its job is to facilitate intergration with cloud services such as instances, lb, storage, etc.

## Worker nodes
Nodes are servers that are the workers of the kubernetes cluster.
At high level they do three things:
1. Watch the API server for the new work assignments.
2. Execute work assignments.
3. Report back to the control plane.

### Kubelet
The kubelet is main Kubernetes agent and run on every cluster node.

When you join a node to cluster, the process installs the kubelet, which is then responsible for registering with cluster. 
This process  registers the node's CPU, memory and storage into wider cluster pool.

One of the main jobs of the kubelet is to watch the API server for new work tasks.

If the kubelet can't run the task, it reports it back to the control plane and lets the control plane decide what action to take.

### Container runtime
The kubelet needs a container runtime to perform container related tasks - things like pulling images and starting and stopping containers.

Kubernetes has moved to plugin model called **Container Runtime Interface**. At high level the CRI masks the internal machinery of Kubernetes and expose clean documented interface for 3rd Party container runtimes to plug into.

Kubernetes is dropping Docker as container runtime. This is because docker is bloated and doesn't support the CRI. **containerd** is replacing it as the most common container runtime for kubernetes.

### Kube-proxy
This runs on every node and is responsible for the local cluster networking. It ensures that node gets its own unique IP address and implementes local iptables of IPVS rules to handle routing and load balancing of traffic on the Pod network.

## Kubernetes DNS
Every kubermetes cluster has an internal DNS service that is vital to service discovery.

The cluster's DNS service has a static IP address that is hard-coded into every Pod on the cluster. This ensures that every container and Pod can locate it and use it for discovery. Service registration is also automatic. This means apps don't need to be coded with the intelligence to register with the Kubernetes service discovery.

Cluster DNS is based on CoreDNS project.

## Packaging Application
An application needs to tick a few boxes to run on a Kubernetes cluster. These include:
1. Packaged as a container
2. Wrapped in a pod
3. Deployed via a declarative manifest file

We need to write microservices, then build it into container image and store it in registry. At this point, the application service is *containerized*.

Next you need to define a Kubernetes Pod to run the containerized application. At the kind of high level we're at, a Pod is just a wrapper that allows a container to run on a cluster. Once you have defined the Pod, you are ready to deploy the app to Kubernetes.

The most common controller is the Deployment. It offers scalability, self-healing, and rolling updates for stateless apps. You define Deployments in YAML manifest files and specify things how many replicas to deploy and how to perform updates.

Once everything is deployed in the Deployment YAML file, you can use the Kubernetes command-line tool to post it to API server as the desired state of the application and Kubernetes will implement it.

## The declarative model and desired state
The declarative model and the concept of desired state are the very heart of Kubernetes.

In Kubernetes, the declarative model works like this.
1. Declare the desired state of an applicaiton microservice in a manifest file
2. Post it to the API server
3. Kubernetes stores it in the cluster store as the application's desired state
4. Kubernetes implements the desired state on the cluster
5. A controller makes sure the observed state of the application doesn't vary from the desired state

Manifest files are written in simple YAML and tell Kubernetes what an application should look like. This is called desired state. It includes things such as which image to use, how many replicas to run, which network ports to listen on, and how to perform updates.

Once manifest is created, post it to the API server. The easiest way to do this is with `kubectl` command line utility. This sends the manifest to the control plane as an HTTP POST, usually on port 443.

Once the request is authenticated and authorized, Kubernetes inspects the manifest, it identifies which controller to send it to (Deployment server), and records the config in the cluster store nodes where the kubelt co-ordinates the hard work of pulling images, starting containers, attaching networks, and starting application processes.

Finally controllers run as background reconcilation loops that constantly monitor the state of things. If the *observed state* varies from *desired state*, Kubernetes performs the tasks are necessary to reconcile the issue.

## Pods

written by @jainil15