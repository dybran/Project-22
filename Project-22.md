## __DEPLOYING APPLICATION INTO KUBERNETES CLUSTER__

In this project, I will initiate the deployment of applications within a Kubernetes (K8s) cluster. Kubernetes is a complex system with numerous components, working with multiple layers of abstraction that separate your application from the underlying host machines where it is executed.

we will explore and witness the following aspects in action:

- Implementing the deployment of software applications using __YAML manifest__ files, featuring various Kubernetes objects, including:
   - Pods
   - ReplicaSets
   - Deployments
   - StatefulSets
   - Services (ClusterIP, NodeIP, Loadbalancer)
   - Configmaps
   - Volumes
   - PersistentVolumes
   - PersistentVolumeClaims
   - And more.

- Understanding the distinctions between __stateful__ and __stateless__ applications.

- Demonstrating the deployment of __MySQL__ as a __StatefulSet__ and providing a rationale for this choice.

- Identifying the limitations associated with deploying applications directly using __YAML manifests__ in Kubernetes.

- Introducing __Helm templates__, exploring their components, and highlighting the significance of __semantic versioning__.

- Converting all the existing __.yaml templates__ into a Helm chart for more streamlined management.

- Deploying additional tools on AWS Elastic Kubernetes Service (EKS) using Helm charts, which include:
   - Jenkins
   - MySQL
   - Ingress Controllers (Nginx)
   - Cert-Manager
   - Ingress configurations for Jenkins and the primary application
   - Deploying Monitoring Tools, such as Prometheus and Grafana.

- Exploring the concept of Hybrid CI/CD by integrating various tools like Gitlab CI/CD and Jenkins. Additionally, we'll delve into GitOps principles using Weaveworks Flux.

When utilizing a Kubernetes cluster, the available options vary depending on its intended purpose.

Numerous organizations choose Managed Service solutions for various compelling reasons, such as:

- Less administrative overheads
- Reduced cost of ownership
- Improved Security
- Seamless support
- Periodical updates to a stable and well-tested version
- Faster cluster spin up

However, there is usually strong reasons why organisations with very strict compliance and security concerns choose to build their own Kubernetes clusters. Most of the companies that go this route will mostly use on-premises data centres. When there is need to store data privately due to its sensitive nature, companies will rather not use a public cloud provider. Because, if they do, they have no idea of the physical location of the data centre in which their data is being persisted. Banks and Governments are typical examples of this.

Some setup options can combine both public and private cloud together. For example, the master nodes, etcd clusters, and some worker nodes that run [__stateful__](https://www.techtarget.com/whatis/definition/stateful-app) applications can be configured in private datacentres, while worker nodes that require heavy computations and [__stateless__](https://www.redhat.com/en/topics/cloud-native-apps/stateful-vs-stateless) applications can run in public clouds. This kind of hybrid architecture is ideal to satisfy compliance, while also benefiting from other public cloud capabilities.

![](./images/arc.PNG)

We will be using the Elastic Kubernates Service(EKS) for this project. To set up the EKS, we need to install WSL. To install WSL click [here](https://learn.microsoft.com/en-us/windows/wsl/install).

OR

Run the following to enable WSL

`dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart`

![](./images/b.PNG)

Reboot your system.  

After the reboot, open the Microsoft Store, search for your preferred Linux distribution (e.g., Ubuntu), and install it.

![](./images/d.PNG)

Complete the initial setup of the Linux distribution by creating a user and password.

Update the ubuntu packages On the WSL

`$ sudo apt update && sudo pat upgrade`

![](./images/c.PNG)

Download and install __eksctl__

`$ curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp`

Move to __/usr/local/bin__

`$ sudo mv /tmp/eksctl /usr/local/bin`

`$ eksctl version`

![](./images/f.PNG)
![](./images/f1.PNG)

For the WSL to interact with the AWS we need to install __awscli__ and configure

`$ sudo apt install awscli`

`$ aws configure`

Install __pip__

`$ sudo apt install python3-pip`

Upgrade the __awscli__

`$ pip install --upgrade awscli`

![](./images/g.PNG)
![](./images/g1.PNG)

To verify run any __aws__ command

`$ aws s3 ls`

Setup __kubectl__

To setup kubectl we will refer to the [kubernetes documentation](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/#install-kubectl-binary-with-curl-on-linux).

Install kubectl

`$ curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"`

Download the kubectl checksum file

`$ curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl.sha256"`

Validate the kubectl binary against the checksum file

`$ echo "$(cat kubectl.sha256)  kubectl" | sha256sum --check`

Install kubectl

`$ sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl`

Verify

`$ kubectl version --client`

![](./images/kkk.PNG)
![](./images/kkk1.PNG)


Now, you have __eksctl__ installed on your Windows system through __WSL__. You can use it to interact with Amazon EKS clusters.

```
$ eksctl create cluster \
  --name deploy \
  --region us-east-1 \
  --nodegroup-name worker \
  --node-type t2.micro \
  --nodes 2
```
![](./images/ss.PNG)

__Configure kubectl__

After the cluster is created, you need to configure kubectl to connect to the cluster. Run the command

`$ aws eks --region us-east-1 update-kubeconfig --name deploy`

Then run to get nodes

`$ kubectl get nodes`

![](./images/rew.PNG)
![](./images/qsw.PNG)

__Creating A Pod For The Nginx Application__


Create nginx pod by applying the manifest file
__nginx-pod.yml__ manifest file shown below

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels: 
    app: nginx-pod
spec:
  containers:
  - image: nginx:latest
    name: nginx-pod
    ports:
    - containerPort: 80
      protocol: TCP
```
Create the pods

`$ kubectl apply -f nginx-pod.yml`

We can access the pod using

`$ kubectl get pod nginx-pod.yml`

To access the information about the pod

`$ kubectl describe pod nginx-pod.yml`

![](./images/ngx.PNG)

__ACCESSING THE APP FROM THE BROWSER__

The primary objective of any solution is to enable access through either a web portal or an application, such as a mobile app. In our current setup, we have a Pod equipped with an Nginx container. However, this Pod can't be accessed directly from a web browser due to its unique IP address.

To resolve this issue, we introduce another Kubernetes component known as a "Service." 

A Service acts as an intermediary that receives requests and forwards them to the respective Pod's IP address.

In essence, a Service acts as a gateway that accepts incoming requests on behalf of the Pods and routes them to the appropriate Pod's IP address. If you execute the provided command, you can obtain the IP address of the Pod. Nonetheless, it's important to note that there is no direct means of accessing this Pod from the external world.

`$ kubectl get pod nginx-pod  -o wide`

![](./images/vvv.PNG)

__Expose a Service on a server’s public IP address & static port__

Sometimes, it may be needed to directly access the application using the public IP of the server (when we speak of a K8s cluster we can replace ‘server’ with ‘node’) the Pod is running on. This is when the NodePort service type comes in handy.

A Node port service type exposes the service on a static port on the node’s IP address. NodePorts are in the __30000-32767__ range by default, which means a NodePort is unlikely to match a service’s intended port (for example, 80 may be exposed as 30080).


Create a Service - __nginx-svc.yml__ manifest file

```
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: NodePort
  selector:
    app: nginx-pod
  ports:
    - protocol: TCP
      port: 80
      nodePort: 30080
```

Create the service

`$ kubectl apply -f nginx-svc.yml`

We can access the svc using

`$ kubectl get svc -o wide`

To access the information about the service

`$ kubectl describe svc nginx-service`

![](./images/ssvvcc.PNG)

To access the service,

Allow the inbound traffic in your EC2’s Security Group to the NodePort range __30080__

![](./images/wwweee.PNG)

Access the nginx using the public IP address of the node the Pod is running on

![](./images/111.PNG)

The port number __30080__ designates the specific port associated with the node where the Pod is currently scheduled to operate. Should the Pod undergo rescheduling to a different node, it will retain this very port number on its new hosting node. Consequently, if you have multiple Pods concurrently running on diverse nodes, each of them will be accessible via their respective node IP addresses, all employing the same consistent port number.

To delete the pod

`$ kubectl delete nginx-pod`

To delete the service

`$ kubectl delete nginx-service`

__CREATE A REPLICA SET__

Let us create a __rs.yml__ manifest for a ReplicaSet object

```
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx-rs
  labels:
    app: nginx-pod
    tier: frontend
spec:
  replicas: 2
  selector:
    matchLabels:
      tier: frontend
  template:
    metadata:
      name: nginx-pod
      labels:
         app: nginx-pod
         tier: frontend
    spec:
      containers:
      - image: nginx:latest
        name: nginx-pod
        ports:
        - containerPort: 80
          protocol: TCP
```
![](./images/1q.PNG)

The manifest file of ReplicaSet consist of the following fields

- __apiVersion:__ This field specifies the version of kubernetes Api to which the object belongs. ReplicaSet belongs to apps/v1 apiVersion.
- __kind:__ This field specify the type of object for which the manifest belongs to. Here, it is ReplicaSet.
- __metadata:__ This field includes the metadata for the object. It mainly includes two fields: name and labels of the ReplicaSet.
- __spec:__ This field specifies the label selector to be used to select the Pods, number of replicas of the Pod to be run and the container or list of containers which the Pod will run. In the above example, we are running 3 replicas of nginx container.

Create the nginx replicaset

`$ kubectl apply -f nginx-rs.yml`

We can access the pods using

`$ kubectl get pods`

To access the information about the pods

`$ kubectl describe pod <pod-id>`

OR 

`$ kubectl get pod <pod-id> -o yaml`

![](./images/vv.PNG)

Detailed information about the replicaset

`$ kubectl get rs nginx-rs -o yaml`

OR

`$ kubectl get rs nginx-rs -o json`

![](./images/111222.PNG)

We can easily scale our ReplicaSet by specifying the desired number of replicas

`$ kubectl scale rs nginx-rs --replicas=<number-of-pods>`

![](./images/scale.PNG)



__Advanced label matching__

As Kubernetes mature as a technology, so does its features and improvements to k8s objects. __ReplicationControllers__ do not meet certain complex business requirements when it comes to using __selectors__. Imagine if you need to select Pods with multiple lables that represents things like

- Application tier: such as Frontend, or Backend
- Environment: such as Dev, SIT, QA, Preprod or Prod

So far, we used a simple selector that just matches a key-value pair and check only __equality__

```
  selector:
    app: nginx-pod
```

But in some cases, we want ReplicaSet to manage our existing containers that match certain criteria, we can use the same simple label matching or we can use some more complex conditions, such as:

 - in
 - not in
 - not equal
 - etc...
Let us create the __rs.yml__ manifest file

```
apiVersion: apps/v1
kind: ReplicaSet
metadata: 
  name: nginx-rs
spec:
  replicas: 2
  selector:
    matchLabels:
      env: prod
    matchExpressions:
    - { key: tier, operator: In, values: [frontend] }
  template:
    metadata:
      name: nginx
      labels: 
        env: prod
        tier: frontend
    spec:
      containers:
      - name: nginx-container
        image: nginx:latest
        ports:
        - containerPort: 80
          protocol: TCP
```

In the above spec file, under the selector, matchLabels and matchExpression are used to specify the key-value pair. The matchLabel works exactly the same way as the equality-based selector, and the matchExpression is used to specify the set based selectors. This feature is the main differentiator between ReplicaSet and previously mentioned obsolete ReplicationController.

Create the replicaset

`$ kubectl apply -f rs.yml`

Get the replication set

`$ kubectl get rs nginx-rs -o wide`

![](./images/rs--.PNG)
![](./images/rs-.PNG)

__USING AWS LOAD BALANCER TO ACCESS YOUR SERVICE IN KUBERNETES.__

We have previously interacted with the Nginx service using ClusterIP and NodeIP. However, there's yet another service type known as __LoadBalancer__. This particular service not only establishes a Service object within Kubernetes but also sets up an actual external Load Balancer, such as the Elastic Load Balancer (ELB) in AWS, if available.

Create the service and ensure that the selector references the Pods in the replica set.

```
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: LoadBalancer
  selector:
    tier: frontend
  ports:
    - protocol: TCP
      port: 80 # This is the port the Loadbalancer is listening at
      targetPort: 80 # This is the port the container is listening at
```

Create the __nginxlb-svc.yml__

`$ kubectl apply -f nginxlb-svc.yml`

An ELB resource will be created in your AWS console

![](./images/lbs3.PNG)

A Kubernetes component in the control plane called __Cloud-controller-manager__ is responsible for triggering this action. It connects to your specific cloud provider’s (AWS) APIs and create resources such as Load balancers. It will ensure that the resource is appropriately tagged

![](./images/555.PNG)

To get the endpoint for the load balancer

`$ kubectl get svc`

![](./images/lbs.PNG)
![](./images/lbs2.PNG)

Access the nginx from the browser

![](./images/ilk.PNG)

To get information about the __nginx-service__

`$ kubectl describe svc nginx-service`

OR

`$ kubectl get svc nginx-service -o yaml`

![](./images/89.PNG)

A clusterIP key is updated in the manifest and assigned an IP address. Even though you have specified a Loadbalancer service type, internally it still requires a clusterIP to route the external traffic through.
In the ports section, nodePort is still used. This is because Kubernetes still needs to use a dedicated port on the worker node to route the traffic through. Ensure that port range 30000-32767 is opened in your inbound Security Group configuration.

Delete the replicaset and the nginx service

`$ kubectl delete rs nginx-rs`

`$ kubectl delete svc nginx-service`

__USING DEPLOYMENT CONTROLLERS__

A __Deployment__ is another layer above ReplicaSets and Pods, newer and more advanced level concept than ReplicaSets. It manages the deployment of ReplicaSets and allows for easy updating of a ReplicaSet as well as the ability to roll back to a previous version of deployment. It is declarative and can be used for rolling updates of micro-services, ensuring there is no downtime.

Officially, it is highly recommended to use __Deployments__ to manage replica sets rather than using __replica sets__ directly.

Create a __deploy.yml__ manifest

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deploy
  labels:
    tier: frontend
spec:
  replicas: 3
  selector:
    matchLabels:
      tier: frontend
  template:
    metadata:
      labels:
        tier: frontend
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 8
```

Create the deployment

`$ kubectl apply -f deploy.yaml`

Get the deployment

`$ kubectl get deploy`

Get the replicaset

`$ kubectl get rs`

Get the pods

`$ kubectl get pods`

![](./images/234.PNG)

From the above we will find that one f the pods is pending. To get information about the pod

`$ kubectl describe pod <pod-id>`

![](./images/events.PNG)

The __Event__ will help us with the problem with the pod.

The warning indicates that there are no available nodes to schedule your pod because all nodes are occupied because I used __t2.micro__ to create the clsuter. 

Some of the ways to resolve this include:

- Add more nodes to your cluster.
- Adjust resource requests and limits.
- Prioritize pods.
- Use PodDisruptionBudgets.
- Scale down or remove unnecessary pods.

I had to update the depoy.yml file to scale down the replicaset to 2.

Connect into one of the Pods' container to run Linux commands

`$ kubectl exec -it nginx-deploy-7d476d754d-fcd55 /bin/bash`

List the files and folders in the Nginx directory

`# ls -latr /etc/nginx`

We can access the content of the __default.conf__

`# cat /etc/nginx/conf.d/default.conf`

![](./images/90.PNG)

Create the __nginxlb-svc.yml__ service and access the nignx from the browser using the loadbalancer endpoint.

![](./images/ilk.PNG)

The set up looks like this

![](./images/pix.PNG)

__PERSISTING DATA FOR PODS__

Deployments are stateless by design. Hence, any data stored inside the Pod’s container does not persist when the Pod dies.

If you were to update the content of the __index.html__ file inside the container, and the Pod dies, that content will not be lost since a new Pod will replace the dead one.

To see this in action, scale down the pods from 2 to 1

`$ kubectl scale deployment nginx-deploy --replicas=1`

![](./images/sc.PNG)

Connect into the pod and install __vim__

`$ kubectl exec -itnginx-deploy-7d476d754d-fcd55 /bin/bash`

`# apt update && apt install vim -y`

![](./images/nn.PNG)

Update the content of the file and add the code below __/usr/share/nginx/html/index.html__

`$ vim /usr/share/nginx/html/index.html`

```
<!DOCTYPE html>
<html>
<head>
<title>Welcome to Solomon's Page!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to Solomon's Page!</h1>
<p>Learning by doing is so much fun</p>

<p>Project Based learning at 
<a href="https://darey.io/">www.darey.io</a>.<br/>
Start your Learning today at
<a href="https://darey.io/">www.darey.io</a>.</p>

<p><em>Thanks</em></p>
</body>
</html>
```
![](./images/index.PNG)

Reload the browser to see the changes made

![](./images/red.PNG)

Now, delete the only running Pod

`$ kubectl delete pod nginx-deploy-7d476d754d-fcd55`

We will see that the replicaset creates another pod with a different pod ID.

Refresh the web page

![](./images/ilk.PNG)

You will see that the content that was saved in the container is no longer there. That is because Pods do not store data when they are being recreated – that is why they are called __ephemeral__ or __stateless__.

Storage is a critical part of running containers, and Kubernetes offers some powerful primitives for managing it. Dynamic volume provisioning, a feature unique to Kubernetes, which allows storage volumes to be created on-demand. Without dynamic provisioning, DevOps engineers must manually make calls to the cloud or storage provider to create new storage volumes, and then create PersistentVolume objects to represent them in Kubernetes. The dynamic provisioning feature eliminates the need for DevOps to pre-provision storage. Instead, it automatically provisions storage when it is requested by users.

To make the data persist in case of a Pod’s failure, you will need to configure the Pod to use following objects:

- __Persistent Volume or pv__ – is a piece of storage in the cluster that has been provisioned by an administrator or dynamically provisioned using Storage Classes.
- __Persistent Volume Claim or pvc__. Persistent Volume Claim is simply a request for storage, hence the "claim" in its name.


In the next project,

- I will use Terraform to create a Kubernetes EKS cluster in AWS, and use some powerful features such as __PV, PVCs, ConfigMaps__.
-I will also be packaging Kubernetes manifests using Helm
- Dynamic provisioning of volumes to make Pods __stateful__, using Kubernetes __Statefulset__
- Deploying applications into Kubernetes using Helm Charts
And many more awesome technologies.

__PROBLEMS ENCOUNTERED__

Could not connect the __kubectl__ to the kubernetes api server using

`aws eks --region us-east-1 update-kubeconfig --name deploy`. 

I had to upgrade the __awscli__ to version __1.29__ using 

`$ pip install --upgrade awscli`






