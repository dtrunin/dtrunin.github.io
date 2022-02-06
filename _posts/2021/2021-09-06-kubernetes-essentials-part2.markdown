---
layout: post
title: "Kubernetes Essentials: ReplicaSet, Deployment"
---

[Kubernetes Essentials: Pod, Service, Ingress][part1]

In this article we'll learn how to scale applications and release new application versions in Kubernetes.
As an application we'll continue using nginx server which we deployed in the [part 1][part1] of this tutorial.

In order to scale an application we should create multiple Pods with this application.
Here's an example of nginx server deployed on 2 Pods. 
We need manifest files for two Pods, one Service and one Ingress.

<nav>
  <div class="nav nav-tabs" id="nav-tab" role="tablist">
    <button class="nav-link active" id="nav-pod1-tab" data-bs-toggle="tab" data-bs-target="#nav-pod1" type="button" role="tab" aria-controls="nav-pod1" aria-selected="true">Pod 1</button>
    <button class="nav-link" id="nav-pod2-tab" data-bs-toggle="tab" data-bs-target="#nav-pod2" type="button" role="tab" aria-controls="nav-pod2" aria-selected="false">Pod 2</button>
    <button class="nav-link" id="nav-service-tab" data-bs-toggle="tab" data-bs-target="#nav-service" type="button" role="tab" aria-controls="nav-service" aria-selected="false">Service</button>
    <button class="nav-link" id="nav-ingress-tab" data-bs-toggle="tab" data-bs-target="#nav-ingress" type="button" role="tab" aria-controls="nav-ingress" aria-selected="false">Ingress</button>
  </div>
</nav>
<div class="tab-content" id="nav-tabContent">
<div markdown="1" class="tab-pane fade show active" id="nav-pod1" role="tabpanel" aria-labelledby="nav-pod1-tab">

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod1
  labels:
    app: nginx-demo
spec:
  containers:
  - name: nginx
    image: nginx:1.19-alpine
    ports:
    - containerPort: 80
```

</div>
<div markdown="1" class="tab-pane fade" id="nav-pod2" role="tabpanel" aria-labelledby="nav-pod2-tab">

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod2
  labels:
    app: nginx-demo
spec:
  containers:
  - name: nginx
    image: nginx:1.19-alpine
    ports:
    - containerPort: 80
```

</div>
<div markdown="1" class="tab-pane fade" id="nav-service" role="tabpanel" aria-labelledby="nav-service-tab">

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

</div>
<div markdown="1" class="tab-pane fade" id="nav-ingress" role="tabpanel" aria-labelledby="nav-ingress-tab">

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

</div>
</div>

All manifests are located in one folder "2-pods-manual".
Create all these objects:

```shell
$ kubectl apply -f 2-pods-manual 
ingress.networking.k8s.io/nginx-demo created
pod/nginx-pod1 created
pod/nginx-pod2 created
service/nginx-service created
```

Attach to Pods and modify default index.html file to distinguish our Pods:

```shell
$ kubectl exec -i -t nginx-pod1 -- /bin/sh
$ echo "hello from Pod1" > /usr/share/nginx/html/index.html

$ kubectl exec -i -t nginx-pod2 -- /bin/sh
$ echo "hello from Pod2" > /usr/share/nginx/html/index.html
```

Check that both Pods work:

```shell
$ curl -H "Host: nginx-demo" 192.168.64.2
hello from Pod1
$ curl -H "Host: nginx-demo" 192.168.64.2
hello from Pod2
$ curl -H "Host: nginx-demo" 192.168.64.2
hello from Pod1
$ curl -H "Host: nginx-demo" 192.168.64.2
hello from Pod2
```

### ReplicaSet

Creating a manifest file for each Pod of one application is inconvenient. 
To avoid this Kubernetes has a **ReplicaSet** object.

**ReplicaSet** is a Kubernetes object that ensures that a specified number of Pod replicas are running at any given time.
It can create and delete Pods to meet the specified number of replicas criteria. It creates Pods from templates.

Here's a ReplicaSet manifest for our nginx server:

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx-demo
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx-demo
  template:
    metadata:
      labels:
        app: nginx-demo
    spec:
      containers:
      - name: nginx
        image: nginx:1.19-alpine
        ports:
          - containerPort: 80
```

In the manifest we set values for the following fields:

- `apiVersion`, `kind`, `metadata` - Already known for us fields that needed by all Kubernetes objects.
- `spec.replicas` - Number of Pod replicas maintained by the ReplicaSet.
- `spec.selector.matchLabels` - Defines a set of Pods that belong to the ReplicaSet.
  Our ReplicaSet targets any Pod with the `app=nginx-demo` label.
- `spec.template` - Is a Pod template. Kubernetes uses this template to create ReplicaSet Pods.
    As we can see the `spec.template` field repeats a Pod manifest.

A ReplicaSet can be easily scaled up or down by simply updating the `spec.replicas` field. 
If we update the `spec.containers` field it's applied only for newly created Pods, existing Pods aren't updated.

Create our ReplicaSet:

```shell
$ kubectl apply -f rs.yaml
replicaset.apps/nginx-demo created                               
```

Show the created ReplicaSet:

```shell
$ kubectl get rs
NAME         DESIRED   CURRENT   READY     AGE
nginx-demo   3         3         3         51s                   
```

Check that 3 Pods with our application running:

```shell
$ kubectl get pods --show-labels=true
NAME               READY     STATUS    RESTARTS   AGE       LABELS
nginx-demo-vdvrz   1/1     Running   0          2m53s   app=nginx-demo
nginx-pod1         1/1     Running   0          4m51s   app=nginx-demo
nginx-pod2         1/1     Running   0          4m47s   app=nginx-demo
```

We specified that we need 3 Pod replicas. Kubernetes created only one Pod (`nginx-demo-vdvrz`) 
because two Pods with the `app=nginx-demo` label already existed.

If we remove one Pod (e.g. `nginx-pod1`) then Kubernetes creates another one (`nginx-demo-8l2fpx`):

```shell
$ kubectl delete pod nginx-pod1                    
pod "nginx-pod1" deleted

$ kubectl get pods --show-labels
NAME               READY     STATUS    RESTARTS   AGE       LABELS
nginx-demo-8l2fp   1/1     Running   0          8s      app=nginx-demo
nginx-demo-vdvrz   1/1     Running   0          4m41s   app=nginx-demo
nginx-pod2         1/1     Running   0          6m35s   app=nginx-demo
```

If we create a third Pod then Kubernetes removes this new Pod:

```shell
$ kubectl apply -f 2-pods-manual/pod1.yaml
pod/nginx-pod1 created

$ kubectl get pods --show-labels=true
nginx-demo-8l2fp   1/1     Running       0          2m36s   app=nginx-demo
nginx-demo-vdvrz   1/1     Running       0          7m9s    app=nginx-demo
nginx-pod1         0/1     Terminating   0          5s      app=nginx-demo
nginx-pod2         1/1     Running       0          9m3s    app=nginx-demo
```

### Deployment

**Deployment** is a Kubernetes object that describes how to create and update instances of an application.
Deployment is a parent object for ReplicaSet. 
When we create a deployment Kubernetes creates a ReplicaSet, which creates Pods. 

Here's a manifest for a deployment where we update nginx from 1.19 to 1.20 version:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-demo
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx-demo
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
  template:
    metadata:
      labels:
        app: nginx-demo
    spec:
      containers:
        - name: nginx
          image: nginx:1.20-alpine
          ports:
            - containerPort: 80
```

As with all other Kubernetes manifests, a Deployment needs `apiVersion`, `kind`, and `metadata` fields.
`spec.replicas`, `spec.selector`, and `spec.template` are identical to the same fields in a ReplicaSet.

The only one new field here is `spec.strategy`. It specifies the strategy used to replace old Pods by new ones. 
`spec.strategy.type` can be "Recreate" or "RollingUpdate". "RollingUpdate" is the default value.

In "Recreate" Deployment all existing Pods are killed before new ones are created.

In "RollingUpdate" Deployment existing Pods are killed and new ones are created gradually.
We can specify `maxUnavailable` and `maxSurge` to control the rolling update process.

`spec.strategy.rollingUpdate.maxUnavailable` is an optional field that specifies the maximum number 
of Pods that can be unavailable during the update process. 
The value can be an absolute number (for example, 5) or a percentage of desired Pods (for example, 10%). 
The absolute number is calculated from percentage by rounding down. 
The value cannot be 0 if `spec.strategy.rollingUpdate.maxSurge` is 0. The default value is 25%.

`spec.strategy.rollingUpdate.maxSurge` is an optional field that specifies the maximum number 
of Pods that can be created over the desired number of Pods. 
The value can be an absolute number (for example, 5) or a percentage of desired Pods (for example, 10%). 
The value cannot be 0 if MaxUnavailable is 0. The absolute number is calculated from the percentage 
by rounding up. The default value is 25%.

Remove our ReplicaSet from previous chapter and create the Deployment:

```shell
$ kubectl delete rs nginx-demo
replicaset.apps "nginx-demo" deleted

$ kubectl apply -f deployment-1.20.yaml 
deployment.apps/nginx-demo created
```

Check that the Deployment was created:

```shell
$ kubectl get deployments
NAME         READY   UP-TO-DATE   AVAILABLE   AGE
nginx-demo   3/3     3            3           8m10s
```

Check that a ReplicaSet was created:

```shell
$ kubectl get rs
NAME                   DESIRED   CURRENT   READY   AGE
nginx-demo-9b59dc694   3         3         3       10m
```

Check that Pods were created:

```shell
$ kubectl get po
NAME                         READY   STATUS    RESTARTS   AGE
nginx-demo-9b59dc694-f6px9   1/1     Running   0          11m
nginx-demo-9b59dc694-phrmk   1/1     Running   0          11m
nginx-demo-9b59dc694-szszn   1/1     Running   0          11m
```

To update nginx from 1.20 to 1.21:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-demo
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx-demo
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
  template:
    metadata:
      labels:
        app: nginx-demo
    spec:
      containers:
        - name: nginx
          image: nginx:1.21-alpine
          ports:
            - containerPort: 80
```

```shell
$ kubectl apply -f deployment-1.21.yaml 
deployment.apps/nginx-demo configured
```

This time Kubernetes didn't create a new deployment. It updated already existing `nginx-demo` deployment:

```shell
$ kubectl get deployment     
NAME         READY   UP-TO-DATE   AVAILABLE   AGE
nginx-demo   3/3     3            3           85m

$ kubectl describe deployment nginx-demo | grep Image
    Image:        nginx:1.21-alpine
```

Kubernetes created a new ReplicaSet for the updated deployment `nginx-demo-748d94467` 
and set replicas to 0 in the ReplicaSet from the previous deployment `nginx-demo-9b59dc694`:

```shell
$ kubectl get rs
NAME                   DESIRED   CURRENT   READY   AGE
nginx-demo-748d94467   3         3         3       1m
nginx-demo-9b59dc694   0         0         0       96m

$ kubectl describe rs nginx-demo-748d94467 | grep Image
    Image:        nginx:1.21-alpine
$ kubectl describe rs nginx-demo-9b59dc694 | grep Image
    Image:        nginx:1.20-alpine
```

Show new Pods:

```shell
$ kubectl get po
NAME                         READY   STATUS    RESTARTS   AGE
nginx-demo-748d94467-cxbw4   1/1     Running   0          1m
nginx-demo-748d94467-m5tm9   1/1     Running   0          1m
nginx-demo-748d94467-qpjx2   1/1     Running   0          1m

$ kubectl describe po nginx-demo-748d94467-qpjx2 | grep Image
    Image:          nginx:1.21-alpine
    Image ID:       docker-pullable://nginx@sha256:859ec6f2dc548cd2e5144b7856f2b5c37b23bd061c0c93cfa41fb5fb78307ead
```

Kubernetes doesn't remove ReplicaSets after Deployment update to have an ability to
rollback to a previous version.

```shell
$ kubectl rollout undo deployment nginx-demo
deployment.apps/nginx-demo rolled back
```

Nginx version in the deployment became 1.20:

```shell
kubectl describe deployment nginx-demo | grep Image
    Image:        nginx:1.20-alpine
```

The `nginx-demo-748d94467` ReplicaSet replicas was set to 0 and `nginx-demo-9b59dc694` replicas set to 3:

```shell
$ kubectl get rsNAME                   DESIRED   CURRENT   READY   AGE
nginx-demo-748d94467   0         0         0       40m
nginx-demo-9b59dc694   3         3         3       113m
```

[Code samples](https://github.com/dtrunin/kube-replicaset-deployment)

[part1]: {% post_url 2021/2021-08-15-kubernetes-essentials-part1 %} "Kubernetes Essentials: Pod, Service, Ingress"
