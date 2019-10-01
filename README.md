# Machine Learning with Kubernetes
![](readme_images/k8sgif.gif)

## What this tutorial covers:
1. Architecture
2. Pod
2. Replication controller
3. Deployment - how to update image in a deployment - healthcheck, 
4. Services - NodePort, ClusterIP, Loadbalancer, Ingress
5. Volumes - PVC, PV
6. Service Discovery???
7. ConfigMap
8. Namespaces


---
8. StatefulSets
9. DaemonSets
10. Resource Usage monitoring
11. Autoscaling
12. Affinity
13. Taints and tolerations
14. Custom resource definitions
15. Operators
16. Resource Quotas
17. Namespaces
18. User Management + RBAC
19. Node maintenance
20. High availability
21. Helm + Helm charts

---

## What is Kubernetes?
- Open source orchestration system for containers
- Lets you schedule containers on a cluster of machines
- You can run multiple containers on one machine
- You can run long running services
- Kubernetes will manage the state of these containers
	- Can start the container on specific nodes
	- Will restart a container when it gets killed
	- Can move containers from one node to another
-  Instead of running containers manually, kubernetes is a platform that will manage the containers for you
-  Kubernetes clusters can start with one node and be scaled to thousands of nodes
-  'Competitors': docker swarm, mesos

### Difference between Kubernetes and Docker Compose

### Kubernetes Architecture

## Pod

### Ensuring app is working and image is pushed

### What is a pod?

`pod-randomforest.yml`

```
apiVersion: v1
kind: Pod
metadata:
    name: example-pod-randomforest
    labels:
      app: random-forest-app
spec:
  containers:
    - name: simple-randomforest
      image: aishpra/simple-randomforest
      ports:
        - containerPort: 5000
```

- Then we run `kubectl apply -f pod-randomforest.yml`
- Then we use port forwarding to access the pod using `kubectl port-forward example-pod-randomforest 5000:5000`
- We can kill the pod using `kubectl delete pods <pod-name>`


### Useful commands
![](readme_images/useful.png)


## Replication Controller

(Recommended only if the application is stateless) 


### What is a replication controller?
- Scaling in Kubernetes can be done using the replication controller
- Replication controller will ensure a specified number of pod replicas will run at all times
- Pods created with a replica controller will automatically be replaced if they fail, get deleted or are terminated


```
apiVersion: v1
kind: ReplicationController
metadata:
  name: example-replicacontroller-randomforest
spec:
  replicas: 5
  selector:
    app: rc-randomforest 
  template:
    metadata:
      labels:
        app: rc-randomforest 
    spec:
      containers:
        - name: simple-randomforest
          image: aishpra/simple-randomforest
          ports:
          - containerPort: 5000
```

- Then we can run `kubectl apply -f replicacontroller-randomforest.yml` 
- We can check the pods using `kubectl get pods`. You should see something like this:
![](readme_images/rc1.png)
- We can check the replicationcontroller by running `kubectl get replicationcontrollers`. You should see something like this 
- We can port-forward from one of the pods to check if the app is working
- Even if we kill one of the pods, it will be restarted using the replication controller
![](readme_images/rc3.png)
- We can then kill the replication controller using `kubectl delete replicationcontrollers example-replicacontroller-randomforest` .
You will see something like this when you run `kubectl get pods`:
![](readme_images/rc4.png)


## Deployment

### What is a deployment?
- A deployment maintains a set of identical pods, ensuring that they have the correct config and that the right number exists.
- It  continuously watches all the pods related to itself and ensures that they are in the right state
- When a change is made in the deployment config, either the pods are altered or killed and a new one created

### Deployment vs Replication Controller

`deployment-randomforest.yml`

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: example-deployment-randomforest
  labels:
    app: dep-randomforest
spec:
  replicas: 3
  selector:
    matchLabels:
      app: dep-randomforest
  template:
    metadata:
      labels:
        app: dep-randomforest
    spec:
      containers:
        - name: simple-randomforest
          image: aishpra/simple-randomforest
          ports:
          - containerPort: 5000
```

- Run `kubectl apply -f deployment-randomforest.yml`
- Run `kubectl get ...` to check the state of the cluster
- We can run `kubectl port-forward deployment/example-deployment-randomforest 5000:5000` to test if the app is running
- Run `kubectl delete deployments <name-of-deployment>` to kill the deployment and associated pods

![](readme_images/dep1.png)

### Updating a Container Image in a Deployment

- We can update the container image using the command `kubectl set image deployment/<deployment-name> <container-name>=<new-image>`
- For this example we will run the command `kubectl set image deployment/example-deployment-randomforest simple-randomforest=aishpra/simple-randomforest:v2`
- We can monitor the rolling update using `kubectl get pods -w`. You should see something like this:
![](readme_images/rolling.png)
- A neat trick to use to tag your Docker images is to use the SHA string corresonding to your git commit. That way there is a clear link between the pushed code and version of the image created as a consequence
- We can also run `kubectl rollout status deployment/example-deployment-randomforest` to keep track of the rolling update process. You should see something like this:
![](readme_images/roll2.png)
- We can also check the image running on the pods using `kubectl get pods -o jsonpath="{..image}" |tr -s '[[:space:]]' '\n' |sort |uniq -c`
- We can use `kubectl rollout history deployment/example-deployment-randomforest`. We can increase the number of revisions that kubernetes keeps by editing the specs paramaters in the deployment.yaml file. You should see something like this:
![](readme_images/roll3.png)
- We can rollback the update by using `kubectl rollout undo deployment/example-deployment-randomforest`



### Useful commands
![](readme_images/dep2.png)


## Services






