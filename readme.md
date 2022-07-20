# Kubernetes learning nodes


## Part I Concept

### 1. The Kubernetes Object Model

Kubernetes objects are persistent entities in the Kubernetes system. Object entities describe:

- What containerized applications we are running.
- The nodes where the containerized applications are deployed.
- Application resource consumption.
- Policies attached to applications, like restart/upgrade policies, fault tolerance, ingress/egress, access control, etc.

you have to set `spec` when you create the object, providing a description of the characteristics you want the resource to have: its desired state.

The `status` describes the current state of the object, supplied and updated by the Kubernetes system and its components. 

#### 1.1. Required Fields in an object manifest

- `apiVersion` 		- Which version of the Kubernetes API you're using to create this object
- `kind` 				- What kind of object you want to create
- `metadata` 			- Data that helps uniquely identify the object, including a `name` string, `uid`, and optional `namespace`
- `spec` 				- What state you desire for the object

#### 1.2. Object Names and IDs

four types of commonly used name constraints for resources.
- DNS Subdomain Names: <= 253 char, alphanumeric or '-', start with and end with an alphanumeric char.
- RFC 1123 Label Names: <= 63 char, alphanumeric or '-', start with and end with an alphanumeric char.
- RFC 1035 Label Names: <= 63 char, alphanumeric or '-', start with an alphabetic char and end with an alphanumeric char.

#### 1.3. Namespaces

In Kubernetes, namespaces provides a mechanism for isolating groups of resources within a single cluster.

`namespace`s by default:

		default           Active   1d
		kube-node-lease   Active   1d
		kube-public       Active   1d
		kube-system       Active   1d

#### 1.4. Labels and Selectors

Labels are key/value pairs that are attached to objects, such as pods. Labels are intended to be used to specify identifying attributes of objects that are meaningful and relevant to users, but do not directly imply semantics to the core system.

**Labels syntax and character set**

Labels are **key/value** pairs. 

**Valid label keys have two segments**: an optional **prefix** and **name**, separated by a slash `/`. 

- The **name** segment is required and must be 63 characters or less, beginning and ending with an alphanumeric character ([a-z0-9A-Z]) with dashes (-), underscores (_), dots (.), and alphanumerics between. 
- The prefix is optional. If specified, the **prefix** must be a DNS subdomain: a series of DNS labels separated by dots (.), not longer than 253 characters in total, followed by a slash (/).
- The kubernetes.io/ and k8s.io/ prefixes are reserved for Kubernetes core components.

**Valid label value**:

- must be 63 characters or less (can be empty),
- unless empty, must begin and end with an alphanumeric character ([a-z0-9A-Z]),
- could contain dashes (-), underscores (_), dots (.), and alphanumerics between.


> Shared labels and annotations share a common prefix: `app.kubernetes.io`. 
> Labels without a prefix are private to users. 
> The shared prefix ensures that shared labels do not interfere with custom user labels.


**Selectors syntax**

**Equality-based requirement**:

		environment = production
		environment == production
		tier != frontend


**Set-based requirement**:

		environment in (production, qa)
		tier notin (frontend, backend)
		partition
		!partition


**Resources that support set-based requirement**

two kinds of selectors could be applied in the selector:

- `matchLabels`:			is a map of {key,value} pairs. 
- `matchExpressions`: 	is a list of pod selector requirements. 

examples:

		selector:
		  matchLabels:
		    component: redis
		  matchExpressions:
		    - {key: tier, operator: In, values: [cache]}
		    - {key: environment, operator: NotIn, values: [dev]}


#### 1.5. Field Selectors

Field selectors let you select Kubernetes resources based on the value of one or more resource fields. Here are some examples of field selector queries:

		metadata.name=my-service
		metadata.namespace!=default
		status.phase=Pending



		kubectl get pods --field-selector status.phase=Running


#### 1.6. Owners and Dependents

A valid owner reference consists of the object name and a UID within the same namespace as the dependent object.


### 1.7. Finalizers

Finalizers are namespaced keys that tell Kubernetes to wait until specific conditions are met before it fully deletes resources marked for deletion. Finalizers alert controllers to clean up resources the deleted object owned.

## 2. Cluster Architecture

Overall Architecture
![Overall Architecture](https://courses.edx.org/assets/courseware/v1/524d69e2587d4a25ce96fdd0bd51eb06/asset-v1:LinuxFoundationX+LFS158x+1T2022+type@asset+block/Components_of_Kubernetes_Architecture.png)


- kube-apiserver

	+ The API Server is the only control plane component to talk to the key-value store, both to read from and to save Kubernetes cluster state information
	+ acting as a middle interface for any other control plane agent inquiring about the cluster's state.

- kube-scheduler

	+ The role of the kube-scheduler is to assign new workload objects, such as pods encapsulating containers, to nodes - typically worker nodes.
	+ The scheduler is highly configurable and customizable through scheduling policies, plugins, and profiles.
	
- kube-controller-manager

	+ monitor the availability of worker nodes 
	+ ensure pod counts are as expected
	+ create endpoints
	+ create service accounts and access tokens

- cloud-controller-manager

	+ The cloud-controller-manager runs controllers or operators responsible to interact with the underlying infrastructure of a cloud provider when nodes become unavailable.
	+ to manage storage volumes when provided by a cloud service.
	+ to manage load balancing and routing.
	
- etcd 

	+ all cluster configuration data is saved to a distributed key-value store which only holds cluster state related data, no client workload generated data.
	+ etcd is a strongly consistent, distributed key-value data store used to persist a Kubernetes cluster's state.
	
	
	* stacked etcd HA

	![](https://courses.edx.org/assets/courseware/v1/de93f27f3cf43ffecf1bd32ac7b0eee9/asset-v1:LinuxFoundationX+LFS158x+1T2022+type@asset+block/Stacked_etcd_topology.png)

	
	* external etcd HA

	![](https://courses.edx.org/assets/courseware/v1/4e5258a38354c2d18fc66321c73416e1/asset-v1:LinuxFoundationX+LFS158x+1T2022+type@asset+block/External_etcd_topology.png)
	
- worker node

	* Container Runtime
	
		+ CRI-Ocontainerd 
		+ docker 
		+ mirantis container runtime
		
	* Node Agent - kubelet

		+ The kubelet is an agent running on each node, control plane and workers, and communicates with the control plane. 
		+ It receives Pod definitions, primarily from the API Server, and interacts with the container runtime on the node to run containers associated with the Pod. 
		+ It also monitors the health and resources of Pods running containers.
		+ The kubelet connects to container runtimes through a plugin based interface - the Container Runtime Interface (CRI). 
			
			![](https://courses.edx.org/assets/courseware/v1/ab209f7c32ceb17ed43dcf6b66056cea/asset-v1:LinuxFoundationX+LFS158x+1T2022+type@asset+block/CRI.png)
			
		
	* Proxy - kube-proxy
		
		+ kube-proxy maintains network rules on nodes. These network rules allow network communication to your Pods from network sessions inside or outside of your cluster.
		+ kube-proxy uses the operating system packet filtering layer if there is one and it's available. Otherwise, kube-proxy forwards the traffic itself.
		+ kube-proxy is a network proxy that runs on each node in your cluster, implementing part of the Kubernetes Service concept.

	
	* Addons for DNS, Dashboard user interface, cluster-level monitoring and logging.

		Addons use Kubernetes resources (DaemonSet, Deployment, etc) to implement cluster features. Because these are providing cluster-level features, namespaced resources for addons belong within the kube-system namespace.

		+ DNS 
		+ Web UI (Dashboard) 
		+ Container Resource Monitoring
		+ Cluster-level Logging

## 3. Container

Each container that you run is repeatable; the standardization from having dependencies included means that you get the same behavior wherever you run it.

Containers decouple applications from underlying host infrastructure. This makes deployment easier in different cloud or OS environments.


### 3.1. Container Images

A container image is a ready-to-run software package, containing everything needed to run an application: the code and any runtime it requires, application and system libraries, and default values for any essential settings.


### 3.2 Container Environment

The Kubernetes Container environment provides several important resources to Containers:

- A filesystem, which is a combination of an image and one or more volumes.
- Information about the Container itself.
- Information about other objects in the cluster.

**Container information**

- the `hostname` of a container is the name of the pod in which the container is running, 

**Cluster information**

- A list of all services that were running when a Container was created is available to that Container as environment variables. 
- This list is limited to services **within the same namespace** as the new Container's Pod and **Kubernetes control plane services**.


## 4. Workload resources

### 4.1. [Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) and [ReplicaSet](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)

A Deployment provides declarative updates for Pods and ReplicaSets.
A ReplicaSet's purpose is to maintain a stable set of replica Pods running at any given time. As such, it is often used to guarantee the availability of a specified number of identical Pods.
A ReplicaSet ensures that a specified number of pod replicas are running at any given time. However, a Deployment is a higher-level concept that manages ReplicaSets and provides declarative updates to Pods along with a lot of other useful features. Therefore, we recommend using Deployments instead of directly using ReplicaSets, unless you require custom update orchestration or don't require updates at all.
ReplicaSet acquires the newly created bare pods which have one key/value pair under the `labels` property set the same as the key/value pair under the `selector` property of the ReplicaSet's `spec`.

**Use Cases**: 
- Create a Deployment to rollout a ReplicaSet. 
- Declare the new state of the Pods.
- Rollback to an earlier Deployment revision.
- Scale up the Deployment to facilitate more load.
- Pause the rollout of a Deployment.
  
**Spec**

    # required general information section
    apiVersion: v1
    kind: Deployment
    metadata: 
      name: <name-of-the-Deployment>
      namespace: <namespace-of-the-Deployment>
      labels:
        <key1>: <value1>
        <key2>: <value2>
    
    # spec section
    spec: 
      replicas: <number-of-Pods>  #optional, default to 1
      selector:   # required field
        matchLabels:
          <key1>: <value1>
          <key2>: <value2>
      template:   # required field 
        metadata: 
          name:
          namespace: 
          labels:
            <key1>: <value1>
            <key2>: <value2>
        spec: 
          containers: 
          - name: <name-of-the-Container>
            image: <image-of-the-Container>
            ports:
              - containerPort: 
              - name:
      

### 4.2. [StatefulSets](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/)

StatefulSet is the workload API object used to manage stateful applications.
StatefulSets are valuable for applications that require one or more of the following.

Like a Deployment, a StatefulSet manages Pods that are based on an identical container spec. Unlike a Deployment, a StatefulSet maintains a sticky identity for each of their Pods. These pods are created from the same spec, but are not interchangeable: each has a persistent identifier that it maintains across any rescheduling.

- Stable, unique network identifiers.
- Stable, persistent storage.
- Ordered, graceful deployment and scaling.
- Ordered, automated rolling updates.

Limitations
- The storage for a given Pod must either be provisioned by a `PersistentVolume` Provisioner based on the requested storage class, or pre-provisioned by an admin.
- Deleting and/or scaling a StatefulSet down will not delete the volumes associated with the StatefulSet. This is done to ensure data safety, which is generally more valuable than an automatic purge of all related StatefulSet resources.
- StatefulSets currently require a **Headless Service** to be responsible for the network identity of the Pods. You are responsible for creating this Service.
- StatefulSets do not provide any guarantees on the termination of pods when a StatefulSet is deleted. To achieve ordered and graceful termination of the pods in the StatefulSet, it is possible to __scale the StatefulSet down to 0__ prior to deletion.
- When using Rolling Updates with the default Pod Management Policy (OrderedReady), it's possible to get into a broken state that requires manual intervention to repair.

**Spec**

    # the headless Service manifest
    apiVersion: v1
    kind: Service
    metadata:
      name: nginx
      labels:
        app: nginx
    spec:
      ports:
      - port: 80
        name: web
      clusterIP: None
      selector:
        app: nginx
    ---
    # the StatefulSet manifest
    apiVersion: apps/v1
    kind: StatefulSet
    metadata:
      name: web
    spec:
      selector:
        matchLabels:
          app: nginx          # has to match .spec.template.metadata.labels
      serviceName: "nginx"
      replicas: 3 # by default is 1
      minReadySeconds: 10 # by default is 0
      template:
        metadata:
          labels:
            app: nginx        # has to match .spec.selector.matchLabels
        spec:
          terminationGracePeriodSeconds: 10
          containers:
          - name: nginx
            image: k8s.gcr.io/nginx-slim:0.8
            ports:
            - containerPort: 80
              name: web
            volumeMounts:
            - name: www
              mountPath: /usr/share/nginx/html
      volumeClaimTemplates:
      - metadata:
          name: www
        spec:
          accessModes: [ "ReadWriteOnce" ]
          storageClassName: "my-storage-class"
          resources:
            requests:
              storage: 1Gi


### 4.4. [DaemonSet](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/) 

- DaemonSets are operators designed to manage node agents.
- DaemonSets present a distinct feature that enforces a single Pod replica to be placed per Node, on all the Nodes. 
- DaemonSet operators are commonly used in cases when we need to collect monitoring data from all Nodes, or to run a storage, networking, or proxy daemons on all Nodes, to ensure that we have a specific type of Pod running on all Nodes at all times.
- Whenever a Node is added to the cluster, a Pod from a given DaemonSet is automatically placed on it.
- If a DaemonSet is deleted, all Pod replicas it created are deleted as well. 
- When any one Node crashes or it is removed from the cluster, the respective DaemonSet operated Pods are garbage collected.

**Spec**

    apiVersion: apps/v1
    kind: DaemonSet
    metadata:
      name: fluentd-elasticsearch
      namespace: kube-system
      labels:
        k8s-app: fluentd-logging
    spec:
      selector:
        matchLabels:
          name: fluentd-elasticsearch
      template:
        metadata:
          labels:
            name: fluentd-elasticsearch
        spec:
          tolerations:
          # these tolerations are to have the daemonset runnable on control plane nodes
          # remove them if your control plane nodes should not run pods
          - key: node-role.kubernetes.io/control-plane
            operator: Exists
            effect: NoSchedule
          - key: node-role.kubernetes.io/master
            operator: Exists
            effect: NoSchedule
          containers:
          - name: fluentd-elasticsearch
            image: quay.io/fluentd_elasticsearch/fluentd:v2.5.2
            resources:
              limits:
                memory: 200Mi
              requests:
                cpu: 100m
                memory: 200Mi
            volumeMounts:
            - name: varlog
              mountPath: /var/log
            - name: varlibdockercontainers
              mountPath: /var/lib/docker/containers
              readOnly: true
          terminationGracePeriodSeconds: 30
          volumes:
          - name: varlog
            hostPath:
              path: /var/log
          - name: varlibdockercontainers
            hostPath:
              path: /var/lib/docker/containers

### 4.5 Jobs

A Job creates one or more Pods and will continue to retry execution of the Pods until a specified number of them successfully terminate. 
A simple case is to create one Job object in order to reliably run one Pod to completion. 

**Spec**

    apiVersion: batch/v1
    kind: Job
    metadata:
      name: pi
    spec:
      template:
        spec:
          containers:
          - name: pi
            image: perl:5.34
            command: ["perl",  "-Mbignum=bpi", "-wle", "'print bpi(2000)'"]
          restartPolicy: Never
      backoffLimit: 4



### 4.6. CronJob

One CronJob object is like one line of a crontab (cron table) file. It runs a job periodically on a given schedule, written in Cron format.


    apiVersion: batch/v1
    kind: CronJob
    metadata:
      name: hello
    spec:
      schedule: "* * * * *"
      jobTemplate:
        spec:
          template:
            spec:
              containers:
              - name: hello
                image: busybox:1.28
                imagePullPolicy: IfNotPresent
                command:
                - /bin/sh
                - -c
                - date; echo Hello from the Kubernetes cluster
              restartPolicy: OnFailure

### 4.7. ReplicationController

Prefer using `Deployment` instead of `ReplicationController`


## 5. Service, Load Balancing, and Networking 

### 5.1. The Kubernetes Networking Model

- "IP per Pod" let pod work like a VM on a physical machine. 
- `Containers` within a pod share their network namespaces - including their IP addresses and Mac addresses, containers within a Pod must coordinate port usage.
- `Containers` within a Pod can all reach each other's ports on `localhost` (use networking to communicate via loopback).
- Cluster networking provides communication between different pods, pods can communicate with all other pods on any other node without NAT.
- The `Service` resource lets you expose an application running in Pods to be reachable from outside your cluster.
- You can also use `Services` to publish services only for consumption inside your cluster.

### 5.2. Service

In Kubernetes, a `Service` is an abstraction which defines a logical set of Pods and a policy by which to access them (sometimes this pattern is called a micro-service). 

A Service in Kubernetes is a REST object, Kubernetes assigns this Service an IP address (sometimes called the "cluster IP"), which is used by the Service proxies (see Virtual IPs and service proxies below).

A Service object use the `selector` to match the pods which has labels as that of the `selector`.

It is convenient to use the name of the Pod's port as the value for the `targetPort`, leveraging the advanced features to evolve services.

    apiVersion: v1
    kind: Pod
    metadata:
      name: nginx
      labels:
        app.kubernetes.io/name: proxy
    spec:
      containers:
      - name: nginx
        image: nginx:stable
        ports:
          - containerPort: 80
            name: http-web-svc
            
    ---
    apiVersion: v1
    kind: Service
    metadata:
      name: nginx-service
    spec:
      selector:
        app.kubernetes.io/name: proxy
      ports:
      - name: name-of-service-port
        protocol: TCP
        port: 80
        targetPort: http-web-svc


***Services without selectors***

Services most commonly abstract access to Kubernetes Pods thanks to the selector, but when used with a corresponding Endpoints object and without a selector, the Service can abstract other kinds of backends, including ones that run outside the cluster.

    apiVersion: v1
    kind: Service
    metadata:
      name: my-service
    spec:
      ports:
        - protocol: TCP
          port: 80
          targetPort: 9376



    apiVersion: v1
    kind: Endpoints
    metadata:
      # the name here should match the name of the Service
      name: my-service
    subsets:
      - addresses:
          - ip: 192.0.2.42
        ports:
          - port: 9376


**Service with `ExternalName`**

An `ExternalName` Service is a special case of Service that does not have selectors and uses DNS names instead.

[Use Cases:](https://akomljen.com/kubernetes-tips-part-1/)

- You want to have an external database cluster in production, but in your test environment you use your own databases.
- You want to point your Service to a Service in a different Namespace or on another cluster.
- You are migrating a workload to Kubernetes. While evaluating the approach, you run only a portion of your backends in Kubernetes.


**Virtual IPs and service proxies**

Every node in a Kubernetes cluster runs a kube-proxy. kube-proxy is responsible for implementing a form of virtual IP for Services of type other than `ExternalName`.

Kubernetes relies on proxying to forward inbound traffic to backends. 


**Discovering services**

Kubernetes supports 2 primary modes of finding a Service 

  - environment variables.
  - DNS.

 When a Pod is run on a Node, the kubelet adds a set of environment variables for each active Service. It adds {SVCNAME}_SERVICE_HOST and {SVCNAME}_SERVICE_PORT variables, where the Service name is upper-cased and dashes are converted to underscores. It also supports variables (see makeLinkVariables) that are compatible with Docker Engine's "legacy container links" feature.

 A cluster-aware DNS server, such as CoreDNS, watches the Kubernetes API for new Services and creates a set of DNS records for each one. If DNS has been enabled throughout your cluster then all Pods should automatically be able to resolve Services by their DNS name.

For example, if you have a Service called my-service in a Kubernetes namespace my-ns, the control plane and the DNS Service acting together create a DNS record for my-service.my-ns. Pods in the my-ns namespace should be able to find the service by doing a name lookup for my-service (my-service.my-ns would also work).

Pods in other namespaces must qualify the name as my-service.my-ns. These names will resolve to the cluster IP assigned for the Service.

The Kubernetes DNS server is the only way to access ExternalName Services.

**Headless Services**

For headless Services, a cluster IP is not allocated, kube-proxy does not handle these Services, and there is no load balancing or proxying done by the platform for them. How DNS is automatically configured depends on whether the Service has selectors defined:

With selectors
For headless Services that define selectors, the endpoints controller creates Endpoints records in the API, and modifies the DNS configuration to return A records (IP addresses) that point directly to the Pods backing the Service.

Without selectors
For headless Services that do not define selectors, the endpoints controller does not create Endpoints records. However, the DNS system looks for and configures either:

CNAME records for ExternalName-type Services.
A records for any Endpoints that share a name with the Service, for all other types.


**Publishing Services (ServiceTypes)**

For some parts of your application (for example, frontends) you may want to expose a Service onto an external IP address, that's outside of your cluster.

**`type` value of the Service:**

- `ClusterIP`: Exposes the Service on a cluster-internal IP only reachable from within the cluster.
- `NodePort`:  Exposes the Service on each Node's IP at a static port.
- `LoadBalancer`: Exposes the Service externally using a cloud provider's load balancer.
- `ExternalName`: Maps the Service to the contents of the externalName field (e.g. foo.bar.example.com), by returning a CNAME record with its value. 

You can also use `Ingress` to expose your Service. Ingress is not a Service type, but it acts as the entry point for your cluster. It lets you consolidate your routing rules into a single resource as it can expose multiple services under the same IP address.


### 5.3. DNS

Kubernetes creates DNS records for Services and Pods. You can contact Services with consistent DNS names instead of IP addresses.

Kubernetes DNS schedules a DNS Pod and Service on the cluster, and configures the kubelets to tell individual containers to use the DNS Service's IP to resolve DNS names.

By default, a client Pod's DNS search list includes the Pod's own namespace and the cluster's default domain.

> The dns setting of each container locates in the `/etc/resolv.conf` file.
> It look like this.

    nameserver 10.96.0.10
    search default.svc.cluster.local svc.cluster.local cluster.local
    options ndots:5

The returns of DNS query is `namespace` specified, they are limited to the Pod's namespace if no specified namespace provided.

Only the `Service` and `Pod` object get the DNS records.

**DNS Records for Services**

- A normal `Service` is assigned an A/AAAA record resolves to the cluster IP of the Service.
- A headless `Service` is assigned an A/AAAA record resolves to the IPs of the `Pod` selected by the Service.
- A `Service` with an `ExternalName` is assigned a CNAME record resolves to the DNS name of the external Service. 

{{< note >}}
this is a note
{{< /note >}}

## Part II: Basic Tasks

- Creating a Namespace
- Creating a `Deployment` for a Stateless Application
- Mounting a Persistent Volume with a `hostPath` Type
- Creating a `ClusterRoleBinding` to enable the Kubernetes-Dashboard
- Creating a `Role` for the default `ServiceAccount`  accessing API Server from within a `Pod`