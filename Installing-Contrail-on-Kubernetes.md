Contrail can be installed on a Kubernetes cluster through multiple provisioning schemes.

This wiki will describe the most simplest of all: **A single yaml based install**

# Pre-requisites
**A running Kubernetes cluster**

   There are multiple options available to user. The most simplest being [kubeadm](https://kubernetes.io/docs/setup/independent/create-cluster-kubeadm/)

   Alternatively if you would like to install Contrail and K8s cluster together, you can use [Contrail Ansible Deployer](https://github.com/Juniper/contrail-ansible-deployer/wiki/Contrail-microservice-installation-with-kubernetes).

# Installation
  Installation of Contrail is a **2**-step process.

  **Step 1**

   Download Contrail single yaml file
```
curl https://github.com/Juniper/contrail-controller/wiki/contrail.yml | awk '/<pre><code>/{flag=1;next}/<\/pre>/{flag=0}flag' > contrail.yml
```
  **Step 2**

  Install Contrail.
  Note: Replace x.x.x.x with the IP of your Kubernetes Master node.
```
K8S_MASTER_IP=x.x.x.x; cat contrail.yml | sed "s/{{ K8S_MASTER_IP }}/$K8S_MASTER_IP/g" | kubectl apply  -f -
```












all pods in a namespace, are to be provisioned.

The annotation to specify this custom virtual-network is:

"opencontrail.org/network: <fq_network_name>" 

```
Example:

apiVersion: v1
kind: Pod
metadata:
  name: ubuntuapp
  labels:
    app: ubuntuapp
  annotations: {
    "opencontrail.org/network" : '{"domain":"default-domain", "project": "admin", "name":"Custom"}'
  }
spec:
  containers:
    - name: ubuntuapp
      image: ubuntu-upstart

```
If this annotation is configured on a pod spec then the pod is launched in that network. 
If the annotation is configured in the namespace spec then all the pods in the namespace will be launched in the provided network. 

NOTE: Creation of the virtual-network should be done using Contrail VNC API's / Contrail-UI, prior to configuring it in the pod/namespace spec. 

### 3.4 __Nested Mode__ (BETA feature in Contrail 4.0.0.0)

Contrail brings together the excellence of kubernetes with that of Openstack, by enabling
the provisioning of Kubernetes cluster inside an Openstack cluster. While this nesting of
clusters by themselves is not unique, what Contrail offers is a "collapsed" control and data
plane whereby a single contrail control plane and single network stack can manage and service
both the Openstack and Kubernetes clusters. With this unified control and data planes,
interworking and configuring these clusters becomes seemless and the lack of replication
and duplicity makes this a very efficient proposition.

In a nested mode, a Kubernetes cluster is provisioned in virtual-machines of an Openstack cluster.
The CNI plugin and the Contrail-kubernetes manager of the kubernetes cluster, interface
directly with Contrail componenets that manage Openstack cluster.

In a nested mode deployment, ALL kubernetes features, functions and specifications are
supported as is. Infact such a nested deployment stretches the boundaries and limits of
Kubernetes by allowing it to operate on the same plane as underlying Openstack cluster.

### 3.5 __Services__
A Kubernetes _Service_ is an abstraction which defines a logical set of _Pods_ and policy by which to access the _Pods_. The Pods implementing a Service are selected based on `LabelSelector` field in _Service_ definition.
In OpenContrail, Kubernetes Service is implemented as **ECMP Native LoadBalancer**. 

The OpenContrail Kubernetes integration supports following `ServiceTypes`:

    * `clusterIP`: This is the default mode. Choosing this 'serviceType' makes 
       the service reachable via cluster-network.

    * `LoadBalancer`: Designating a ServiceType as LoadBalancer, exposes the service
       to external world. The `LoadBalancer` _Service_ is assigned both a CluserIP
       and ExternalIP addresses. This assumes that user has configured public network 
       with a floating-ip pool.

OpenContrail K8S _Service_ integration supports `TCP` and `UDP` for protocols. Also _Service_ can expose more than one port where `port` and `targetPort` are different. For example:

```yaml
kind: Service
apiVersion: v1
metadata:
  name: my-service
spec:
    selector:
      app: MyApp
    ports:
      - name: http
        protocol: TCP
        port: 80
        targetPort: 9376
      - name: https
        protocol: TCP
        port: 443
        targetPort: 9377
``` 
K8S user can specify `spec.clusterIP` and `spec.externalIPs` for both `LoadBalancer` and `ClusterIP` `ServiceTypes`.

If `ServiceTypes` is `LoadBalancer` and no `spec.externalIP` is specified by the user, then contrail-kube-manager allocates a floating-ip from public pool and associates it to `externalIP`. 


### 3.6 __Ingress__ 
k8s services can be exposed to external (outside of the cluster) in many
ways. Popular ways are <https://kubernetes.io/docs/concepts/services-networking/ingress/#alternatives>. Ingress is another way to expose the service to external. Ingress provides layer 7 load balancing whereas the others provide layer 4 load balancing.

## 4. __Contrail Kubernetes Solution__
### 4.1 __Contrail kubernetes manager__
Opencontrail implementation requires listening to K8s API messages and create corresponding resources in the Opencontrail API database. New module called "contrail-kube-manager" will run in a container to listen to the messages from the K8s api-server.

### 4.2 __Contrail CNI plugin__

### 4.3 __ECMP Loadbalancer for K8s service__
Each service in K8s will be represented by a loadbalancer object. The service IP allocated by K8s will be used as the VIP for the loadbalancer. Listeners will be created for the port on which service is listening. Each pod will be added as a member for the listener pool. contrail-kube-manager will listen for any changes based on service labels or pod labels to update the member pool list with the add/updated/delete pods.

Loadbalancing for services will be L4 non-proxy loadbalancing based on ECMP. The instance-ip (service-ip) will be linked to the ports of each of the pods in the service. This will create an ECMP next-hop in Opencontrail and traffic will be loadbalanced directly from the source pod.

### 4.4 __Haproxy Loadbalancer for K8s ingress__
K8s Ingress is implemented through haproxy load balancer feature in contrail.
[Please refer contrail feature guide for more information on contrail
load balancer].<br/><br/> Whenever ingress is configured in k8s, contrail-kube-manager creates the load balancer object in contrail-controller. Contrail service contrail-svc-monitor listens for the load balancer objects and launches the haproxy with appropriate configuration based on the ingress spec rules in active-standby mode in two active compute nodes.<br/><br/>
For more information please refer k8s-ingress.md

### 4.5 __Security groups for K8s network policy__

Network policies can be applied in a cluster configured in isolation mode, to define which pods can communicate with each other or with other endpoints.
The cluster admin will create a Kubernetes API NetworkPolicy object. This is an ingress policy, it applies to a set of pods, and defines which set of pods is allowed access. Both source and destination pods are selected based on labels. The app developer and the cluster admin can add labels to pods, for instance “frontend” / “backend”,  and “development” / “test” / “production”. Full specification of the Network Policy can be found here:

http://kubernetes.io/docs/user-guide/networkpolicies/

Contrail-kube-manager will listen to Kubernetes NetworkPolicy create/update/delete events, and will translate the Network Policy to Contrail Security Group objects applied to Virtual Machine Interfaces. The algorithm will dynamically update the set of Virtual Machine Interfaces as pods and labels are added/deleted.

### 4.6 __DNS__
Kubernetes(K8S) implements DNS using SkyDNS, a small DNS application that responds to DNS requests for service name resolution from Pods. On K8S, SkyDNS runs as a Pod.


## 5. __Performance and scaling impact__

### 5.1 __Forwarding performance__

## 6. __Upgrade__

## 7. __Deprecations__

## 8. __Dependencies__
* Native loadbalancer implementation is needed to support service loadbalancing. https://blueprints.launchpad.net/juniperopenstack/+spec/native-ecmp-loadbalancer
* Health check implementation

## 9. __Debugging__

### 9.1 __Pod IP Address Info:__

    The following command can be used to determine the ip address assigned to a pod:

    kubectl get pods --all-namespaces -o wide

    Example:

    [root@k8s-master ~]# kubectl get pods --all-namespaces -o wide
    NAMESPACE     NAME                                 READY     STATUS    RESTARTS   AGE       IP              NODE
    default       client-1                             1/1       Running   0          19d       10.47.255.247   k8s-minion-1-3
    default       client-2                             1/1       Running   0          19d       10.47.255.246   k8s-minion-1-1
    default       client-x                             1/1       Running   0          19d       10.84.31.72     k8s-minion-1-1

### 9.2 __Check Pods reachability:__

    To verify that pods are reachable to each other, we can run ping among pods:

    kubectl exec -it <pod-name> ping <dest-pod-ip>

    Example:

    [root@a7s16 ~]# kubectl get pods -o wide
    NAME                        READY     STATUS    RESTARTS   AGE       IP              NODE
    example1-2960845175-36xpr   1/1       Running   0          43s       10.47.255.251   b3s37
    example2-3163416953-pldp1   1/1       Running   0          39s       10.47.255.250   b3s37

    [root@a7s16 ~]# kubectl exec -it example1-2960845175-36xpr ping 10.47.255.250
    PING 10.47.255.250 (10.47.255.250): 56 data bytes
    64 bytes from 10.47.255.250: icmp_seq=0 ttl=63 time=1.510 ms
    64 bytes from 10.47.255.250: icmp_seq=1 ttl=63 time=0.094 ms

### 9.3 __Verify that default virtual-network for a cluster is created:__

    In the Contrail GUI, verify that a virtual-network named “cluster-network” is created in your project.

### 9.4 __Verify a virtual-network is created for an isolated namespace:__

    In the Contrail-GUI, verify that a virtual-network with the name format: “<namespace-name>-
    vn” is created.

### 9.5 __Verify that Pods from non-isolated namespace CANNOT reach Pods in isolated namespace.__

    1.  Get the ip of the pod in isolated namespace.
    [root@a7s16 ~]# kubectl get pod -n test-isolated-ns -o wide
    NAME                        READY     STATUS    RESTARTS   AGE       IP              NODE
    example3-3365988731-bvqx5   1/1       Running   0          1h        10.47.255.249   b3s37

    2.  Ping the ip of the pod in isolated namespace from a pod in another namespace:
    [root@a7s16 ~]# kubectl get pods
    NAME                        READY     STATUS    RESTARTS   AGE
    example1-2960845175-36xpr   1/1       Running   0          15h
    example2-3163416953-pldp1   1/1       Running   0          15h

    [root@a7s16 ~]# kubectl exec -it example1-2960845175-36xpr ping 10.47.255.249
            --- 10.47.255.249 ping statistics ---
     2 packets transmitted, 0 packets received, 100% packet loss

### 9.6 __Verify that Pods in isolated namespace can reach Pods in in non-isolated namespaces.__

    1.  Get the ip of the pod in non-isolated namespace.

    [root@a7s16 ~]# kubectl get pods -o wide
    NAME                        READY     STATUS    RESTARTS   AGE       IP              NODE
    example1-2960845175-36xpr   1/1       Running   0          15h       10.47.255.251   b3s37
    example2-3163416953-pldp1   1/1       Running   0          15h       10.47.255.250   b3s37

    2.  Ping the ip of the pod in non-isolated namespace from a pod in isolated namespace:

    [root@a7s16 ~]# kubectl get pods -o wide
    NAME                        READY     STATUS    RESTARTS   AGE       IP              NODE
    example1-2960845175-36xpr   1/1       Running   0          15h       10.47.255.251   b3s37
    example2-3163416953-pldp1   1/1       Running   0          15h       10.47.255.250   b3s37
    [root@a7s16 ~]# kubectl exec -it example3-3365988731-bvqx5 -n test-isolated-ns ping 10.47.255.251
    PING 10.47.255.251 (10.47.255.251): 56 data bytes
    64 bytes from 10.47.255.251: icmp_seq=0 ttl=63 time=1.467 ms
    64 bytes from 10.47.255.251: icmp_seq=1 ttl=63 time=0.137 ms
    ^C--- 10.47.255.251 ping statistics ---
    2 packets transmitted, 2 packets received, 0% packet loss
    round-trip min/avg/max/stddev = 0.137/0.802/1.467/0.665 ms

### 9.7 __How to check if a Kubernetes namespace is isolated.__

    Use the following command to look at annotations on the namespace:

    Kubectl describe namespace <namespace-name>

    If the annotations on the namespace has the following statement, then the namespace is isolated.

    'opencontrail.org/isolation' : 'true'

        [root@a7s16 ~]# kubectl describe namespace test-isolated-ns
        Name:       test-isolated-ns
        Labels:     <none>
        Annotations:    opencontrail.kubernetes.isolated : true     Namespace is isolated
        Status:     Active

## 10. __Testing__
### 10.1 __Unit tests__
### 10.2 __Dev tests__
### 10.3 __System tests__

## 11. __Installation__
Follow below steps to install Containerized contrail controller for Kubernetes.
* Re-image all hosts with Ubuntu 16.04.2. Setup passwordless access to all hosts.
* Install Docker on all the target nodes. Update your /etc/apt/sources.list accordingly so that it can download package from internet 
* Download contrail-ansible repo and create your inventory file. 

  Wiki: https://github.com/Juniper/contrail-ansible/wiki/Provision-Contrail-Kubernetes-Cluster

* Create a folder container_images inside contrail ansible playbook.
* Download contrail-kubernetes-docker_<contrail-version>.tgz. Untar tgz and copy all docker image to container_images folder. 
* Run the ansible playbook: ansible-playbook  -i inventory/my-inventory site.yml