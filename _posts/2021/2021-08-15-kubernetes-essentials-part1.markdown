---
layout: post
title: "Kubernetes Essentials: Pod, Service, Ingress"
---

In this article we'll learn what Kubernetes is, how it works and deploy an application in it.

Kubernetes is a platform for managing containerized applications.
It provides tools for deployment, scaling and maintaining applications in alive state 
(it restarts applications if they crashed).
It uses Docker or any other container runtime that is compatible with the Container Runtime Interface (CRI). 

When we deploy Kubernetes, we get a **cluster**: a set of virtual or physical machines
that work together as a single system. 
Each such machine is called **Node** in Kubernetes terminology. 

The Kubernetes heart is **Control Plane**. It consists of various components that manage user containers.
A common approach is to run control plane components and user containers in different nodes.

**Control plane** exposes REST **API** for providing access to Kubernetes functions via HTTP. 
Various clients use this API to communicate with Kubernetes. 
**Kubectl** and **Kubernetes Dashboard** are examples of such clients.
The **Kubernetes Dashboard** is a web-based Kubernetes user interface.
The **kubectl** (`ctl` part stands for `control`) is a command line tool which lets us control Kubernetes clusters.

Here's a simplified scheme of how Kubernetes works:

![Components of Kubernetes](/assets/images/kubernetes-components-simplified.svg)

### Pod  

Let's deploy our first application in Kubernetes. As the application we'll go for nginx web server.
We need to have a Kubernetes cluster, and the kubectl command-line tool must be configured to communicate with our cluster.
For learning purposes we created a cluster by using [minikube](https://minikube.sigs.k8s.io/docs/start/).

**Pod** is the smallest and simplest Kubernetes object. A Pod represents a set of running containers.
A Pod is typically set up to run a single primary container. 
It can also run optional sidecar containers that add supplementary features like logging.

This means to deploy our application we should tell to Kubernetes to create a Pod 
and run a container from `nginx` image in the Pod.

In order to create a Pod or any other object in Kubernetes, we must provide the manifest that describes the object.
`kubectl` accepts such a manifest in a .yaml file.

Here is a manifest describing a Pod for our application:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-demo
spec:
  containers:
  - name: nginx
    image: nginx:1.20.1-alpine
    ports:
    - containerPort: 80
```

In the manifest we set values for the following fields:

- `apiVersion` - Which version of the Kubernetes API we're using to create this object.
  With new Kubernetes releases API may change. For backward compatibility old API is also maintained for some period of time.
- `kind` - What kind of object we want to create. In our case this is `Pod`.
- `metadata` - Data that helps uniquely identify the object. Here we specify a `name`. 
  Each object in a cluster has a name that is unique for that type of object.
- `spec` - What state we desire for the object. Here we want to run one container from the `nginx:1.20.1-alpine` image. 

All Kubernetes objects needs `apiVersion`, `kind`, and `metadata` fields. The `spec` field is also used for many objects.
The precise format of the object spec is different for every Kubernetes object, 
and contains nested fields specific to that object.

To create an object we should use `kubectl apply` command and pass a `.yaml` file as an argument:

```shell
$ kubectl apply -f nginx-demo-pod.yaml
pod/nginx-demo created
```

To check that our pod is created we could use the `kubectl get pods` command:

```shell
$ kubectl get pods
NAME         READY     STATUS    RESTARTS   AGE
nginx-demo   1/1       Running   0          7m18s
```

Show detailed description of our Pod:

```shell
$ kubectl describe pod nginx-demo
```

So far so good. Nginx is started, but we should create 2 more Kubernetes objects to access it.

### Service

Kubernetes gives Pods their own IP addresses in the cluster network.
We can scale our application and create multiple Pods with it.
This leads to a problem: how do clients of our application find out and keep track of which IP address to connect to?
To solve this problem Kubernetes introduces **Services**.

**Service** is an abstraction over a set of Pods. It allows to expose an application running on the set of Pods as a network service. 
Kubernetes assigns each Service an IP address and DNS name only routable within the cluster network.
A Service can load-balance across its Pods.

To create a Service we should create a file with a manifest describing the Service:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx-demo
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```

Root fields repeat Pod fields. In the `spec` field we define `port` which our Service listens to 
and the `targetPort` is a port which Pods listen to.

The `selector` field defines a set of Pods that belong to the Service.
Our Service targets any Pod with the `app=nginx-demo` label.
Labels are key/value pairs that are attached to objects, such as pods.
Labels can be used to organize and to select subsets of objects.

Here's an example of a Service:

![Service Example](/assets/images/service.svg)

Our nginx Pod has no labels because in the Pod manifest we didn't specify labels. 
To see Pod labels we can use the `kubectl get pods --show-labels=true` command:

```shell
$ kubectl get pods --show-labels=true
NAME         READY     STATUS    RESTARTS   AGE       LABELS
nginx-demo   1/1       Running   0          2d2h      <none>
```

Here is an updated manifest for our Pod with the `app=nginx-demo` label:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-demo
  labels:
    app: nginx-demo
spec:
  containers:
    - name: nginx
      image: nginx:1.20.1-alpine
      ports:
        - containerPort: 80
```

Let's remove our Pod and create a new one with the `app=nginx-demo` label:

```shell
$ kubectl delete -f ./nginx-demo-pod.yaml 
pod "nginx-demo" deleted

$ kubectl apply -f nginx-demo-pod-with-label.yaml 
pod/nginx-demo created
```

Show the created Pod:

```shell
$ kubectl get pods --show-labels=true
NAME         READY     STATUS    RESTARTS   AGE       LABELS
nginx-demo   1/1       Running   0          5m5s      app=nginx-demo
```

Now let's create a Service:

```shell
$ kubectl apply -f service.yaml 
service/nginx-service created
```

Show the created service:

```shell
$ kubectl get services
NAME            TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
kubernetes      ClusterIP   10.96.0.1       <none>        443/TCP   2d4h
nginx-service   ClusterIP   10.102.91.244   <none>        80/TCP    51s
```

### Ingress

Now our nginx server is available for clients inside the Kubernetes cluster.
To access it from outside the Kubernetes cluster we need an **Ingress** and an **Ingress Controller**.  

**Ingress** is an object that defines routing rules for matching incoming requests with Services.
**Ingress controller** consists of a controller daemon and a load balancer e.g. nginx.
The controller daemon receives from Kubernetes Ingress objects and transforms them into the load balancer configuration file.
The load balancer receives HTTP (or HTTPS) requests and redirects them to corresponding Services.

![Ingress Example](/assets/images/ingress.svg)

Kubernetes supports different [Ingress controllers](https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/).
We will use [nginx](https://github.com/kubernetes/ingress-nginx/blob/main/README.md#readme) Ingress controller.
How to set up nginx Ingress controller on minikube read [here](https://kubernetes.io/docs/tasks/access-application-cluster/ingress-minikube/).  

Here's a manifest describing our Ingress:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx-demo
spec:
  rules:
    - host: nginx-demo
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: nginx-service
                port:
                  number: 80
```

Each HTTP rule contains the following information:

- An optional host. If no host is specified, the rule applies to all inbound HTTP traffic. 
  If a host is provided (for example, foo.bar.com), the rules apply to that host.
- A list of paths, each of which has an associated backend defined with a `service.name` 
  and a `service.port.name` or `service.port.number`.
  Path type `Prefix` means that matched any url that has `path` prefix.

Create an Ingress

```bash
$ kubectl apply -f ingress.yaml                    
ingress.networking.k8s.io/nginx-demo created
```

Show Ingress list:

```bash
$ kubectl get ingress                              
NAME         CLASS    HOSTS        ADDRESS          PORTS   AGE
nginx-demo   <none>   nginx-demo   192.168.99.100   80      8h
```

Show detailed Ingress description:

```bash
$ kubectl describe ingress nginx-demo
```

From the description we can see that our Ingress is available outside the Kubernetes cluster
at the IP address 192.168.99.100. 
To check that our web server works let's send an HTTP request to the address 192.168.99.100. 
In the request header we specify our host:

```bash
$ curl -H "Host: nginx-demo" 192.168.99.100
```

In the response we see an nginx welcome page. Hooray, our first application is successfully deployed!

[Code samples](https://github.com/dtrunin/kube-replicaset-deployment)

