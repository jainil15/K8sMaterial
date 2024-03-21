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


written by @jainil15