
# 1. Introduction
Kubernetes (K8s) is an open source container management platform. It provides a portable platform across public and private clouds. K8s supports deployment, scaling and auto-healing of applications. More details can be found at: http://kubernetes.io/docs/whatisk8s/. 


# 2. Problem statement
There is a need to provide pod addressing, network isolation, policy based security, gateway, SNAT, loadalancer and service chaining capability in Kubernetes orchestratation. To this end K8s supports a framework for most of the basic network connectivity. This pluggable framework is called Container Network Interface (CNI). Opencontrail will support CNI for Kubernetes.


# 3. Proposed solution
Currently K8s provides a flat networking model wherein all pods can talk to each other. Network policy is the new feature added to provide isolation between pods/services. Opencontrail will add additional networking functionality to the solution - multi-tenancy, network isolation, micro-segmentation with network policies, load-balancing etc. 

Opencontrail maps the following k8s concepts into opencontrail resources:

|Kubernetes|Opencontrail|
|:---|:---|
|Namespace|Shared or single project based on configuration|
|Pod|Virtual-machine, Interface, Instance-ip|
|Service|ECMP based native Loadbalancer|
|Ingress|Haproxy based L7 Loadbalancer for URL routing|
|Network Policy|Security group based on namespace and pod selectors|

Opencontrail can be configured in the following mode in a K8s cluster:

|Deployment modes|
|:---|
|Default|
|Namespace isolation (with or without service isolation)|
|Custom (define network for a pod)|
|Nested (k8s cluster in openstack virtual-machines|

# 3.1 Default

Kubernetes imposes the following fundamental requirement on any networking implementation:

All Pods can communicate with all other containers without NAT

This is the default mode, and no action is required from the admin or app developer. It provides the same isolation level as kube-proxy. OpenContrail will create a cluster network shared by all namespaces, from where service IP addresses will be allocated.

This in essence is the "default" mode in Contrail networking model in Kubernetes.
In this mode, ALL pods in ALL namespaces that are spawned in the Kubernetes cluster will
be able to communicate with each other. The IP addresses for all the pods will be allocated
from a pod subnet that the Contrail Kubernetes manager is configured with.

NOTE:
System pods spawned in Kube-system namespace are NOT run in the Kubernetes Cluster. Rather they run in the underlay. Networking for these pods is not handled by Contrail.

# 3.1.1 Implementation

Contrail achieves this inter-pod network connectivity by configuring all the pods in a single Virtual-network. When the cluster is initialized, Contrail creates a virtual-network called "cluster-network".

In the absence of any network segmentation/isolation configured, ALL pods in ALL namespaces get assigned to "cluster-network" virtual-network.

# 3.1.2   Pods

In Contrail, each POD is represented as a Virtual-Machine-Interface/Port.

When a pod is created, a vmi/port is allocated for that POD. This port is made a member of the default virtual-network of that Kubernetes cluster.

# 3.1.3   Pod subnet:

The CIDR to be used for IP address allocation for pods is provisioned as a configuration to
contrail-kube-manger. To view this subnet info:

Login to contrail-kube-manager docker running on the Master node and see the "pod_subnets" in configuration file:  /etc/contrail/contrail-kubernetes.conf

# 3.2 Namespace isolation mode

In addition to default networking model mandated by Kubernetes, Contrail support additional, custom networking models that makes available the many rich features of Contrail to the users of the Kubernetes cluster. One such feature is network isolation for Kubernetes namespaces.

A Kubernetes namespace can be configured as “Isolated” by annotating the Kubernetes namespace metadata with following annotation:

“opencontrail.kubernetes.isolated” : “true”

Namespace isolation is intended to provide network isolation to pods.
The pods in isolated namespaces are not reachable to pods in other namespaces in the cluster.

Kubernetes Services are considered cluster resources and they remain reachable to all pods, even those that belong to isolated namespaces. Thus Kubernetes service-ip remains reachable to pods in isolated namespaces.

If any Kubernetes Service is implemented by pods in isolated namespace, these pods are reachable to pods from other namespaces through the Kubernetes Service-ip.

A namespace annotated as “isolated” has the following network behavior:

a.  All pods that are created in an isolated namespace have network reachability with each other.
b.  Pods in other namespaces in the Kubernetes cluster will NOT be able to reach pods in the isolated namespace.
c.  Pods created in isolated namespace can reach pods in other namespaces.
d.  Pods in isolated namespace will be able to reach ALL Services created in any namespace in the kubernetes cluster.
e.  Pods in isolated namespace can be reached from pods in other namespaces through Kubernetes Service-ip.

# 3.2.1 Implementation

For each namespace that is annotated as isolated, Contrail will create a Virtual-network with name:  “<Namespace-name>-vn”.

# 3.2.2 Pods

A Kubernetes pod is represented as vmi/port in Contrail. These ports are mapped to the virtual-network created for the corresponding isolated-namespace.

# 3.2.3   Kubernetes Service Reachability:

Pods from an isolated namespace should be able to reach all Kubernetes in the cluster.

Contrail achieves this reachability by the following:

1.  All Service-IP for the cluster is allocated from a Service Ipam associated with the default cluster virtual-network.
2.  Pods in isolated namespaces are associated with a floating-ip from the default cluster-network. This floating-ip makes is possible for the pods in isolated-namespaces to be able to reach Services and Pods in the non-isolated namespaces.

# 3.3 App isolation mode

In this finer-grain isolation mode, the admin or app developer can add the label "opencontrail.org/name" to the pod, replication controller and/or service specification, to enable micro-segmentation. As a result, virtual networks will be created for each pod/app tagged with the label. Network policies or security groups will need to be configured to define the rules for service accessibility.

# 3.4 Services
A Kubernetes service is an abstraction which defines a logical set of Pods and policy by which to access them. The set of Pods frontend'ing a Service are selected based on **LabelSelector** field in **Service** definition.
In OpenContrail, Kubernetes Service is implemented as **ECMP Native LoadBalancer**. The OpenContrail Kubernetes integration supports following Service types:
    * 'clusterIP': This is the default 'ServiceType'. Choosing this 'serviceType' makes it reachable with cluster-network.
    * 'LoadBalancer': Designating a ServiceType as LoadBalancer, exposes the service to external world. The LoadBalancer 
       service is assigned both a CluserIP and ExternalIP addresses. This assumes that user has configured public network
       with a floating-ip pool.
OpenContrail K8S `Services` integration supports `TCP` and `UDP` for protocols. Also 'Service' can expose more than one port where 'port' and 'targetPort' are different. For example:
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
K8S user can specify `spec.clusterIP` and 'spec.externalIPs' for both 'LoadBalancer' and 'ClusterIP' serviceType.
If ServiceType is 'LoadBalancer' and no externalIP is specified by user, then contrail-kube-manager allocates a
floating-ip from public pool and associates it with externalIP. 

## __K8S Ingress Introduction__
k8s services can rbe exposed to external (outside of the cluster) in many
ways. Popular ways are <https://kubernetes.io/docs/concepts/services-networking/ingress/#alternatives>. Ingress is another way to expose the service to external. Ingress provides layer 7 load balancing whereas the others provide layer 4 load balancing.

### __Types of ingress__
1. Single Service:<br/>
    This type is as equivalent to exposing the services with other ways.<br/>

    Sample yaml to create single-service:<br/>
    &nbsp;&nbsp;&nbsp;&nbsp;apiVersion: extensions/v1beta1<br/>
    &nbsp;&nbsp;&nbsp;&nbsp;kind: Ingress<br/>
    &nbsp;&nbsp;&nbsp;&nbsp;metadata:<br/>
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;name: single-service<br/>
    &nbsp;&nbsp;&nbsp;&nbsp;spec:<br/>
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;backend:<br/> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;serviceName: nginx<br/>    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;servicePort: 80<br/>

    All the traffic would be routed to nginx service.<br/>

2.  Simple fanout:<br/>
    Based on the url path, the traffic would be routed to desired service.<br/>

    Sample yaml to create simple-fanout:<br/>
    &nbsp;&nbsp;&nbsp;&nbsp;apiVersion: extensions/v1beta1<br/>
    &nbsp;&nbsp;&nbsp;&nbsp;kind: ingress<br/>
    &nbsp;&nbsp;&nbsp;&nbsp;metadata:<br/>
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;name: simple-fanout<br/>
    &nbsp;&nbsp;&nbsp;&nbsp;spec:<br/>
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;rules:<br/>
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;- &nbsp;&nbsp;http:<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;paths:<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;-&nbsp;&nbsp;path: /qa.html<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;backend:<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;serviceName: qa-webserver<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;servicePort: 80<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;-&nbsp;&nbsp;path: /dev.html<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;backend:<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;serviceName: dev-webserver<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;servicePort: 80<br/>

    If the incoming traffic's url path matches to qa.html, the traffic would be routed to qa-webserver service. If the incoming traffic ul matches to dev.html, the traffic's would be touted to dev-webserver.<br/>

3.  Name based Virtual Hosting:<br/>
    Multiple host names use the same ip address. Based on the Host, traffic would be routed to desired service.<br/>

    Sample yaml to create name based virtual-hosting:<br/>
&nbsp;&nbsp;&nbsp;&nbsp;apiVersion: extensions/v1beta1<br/>
&nbsp;&nbsp;&nbsp;&nbsp;kind: ingress<br/>
&nbsp;&nbsp;&nbsp;&nbsp;metadata:<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;name: virtual-host<br/>
&nbsp;&nbsp;&nbsp;&nbsp;spec:<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;rules:<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;-  host: maps.google.com<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;http<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;paths:<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;-  backend:<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;serviceName: google-maps-webserver<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;servicePort: 80<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;-  host: images.google.com<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;http<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;paths:<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;-  backend:<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;serviceName: google-images-webserver<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;servicePort: 80<br/>

    If the incoming traffic's host header matches to maps.google.com, the traffic would be routed to google-maps-webserver. If the incoming traffic's host header matches to images.google.com, the traffic would be routed to google-images-webserver.<br/>

For more information please refer
<https://kubernetes.io/docs/concepts/services-networking/ingress/>

## **Ingress in Contrail**
Ingress is implemented through load balancer feature in contrail.
[Please refer contrail feature guide for more information on contrail
load balancer]. Whenever ingress is configured in k8s, contrail-kube-manager creates the load balancer object in contrail-controller. Contrail service contrail-svc-monitor listens for the load balancer objects and launches the haproxy with appropriate configuration based on the ingress spec rules in active-standby mode in two active compute nodes.

### __Detailed Explanation with simple-fanout example:__
#### __Simple-fanout yaml:__<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;apiVersion: extensions/v1beta1<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;kind: ingress<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;metadata:<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;name: simple-fanout<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;spec:<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;rules:<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;- http:<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;paths:<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;-  path: /qa.html<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;backend:<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;serviceName: qa-webserver<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;servicePort: 80<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;paths:<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;-  path: /dev.html<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;backend:<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;serviceName: dev-webserver<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;servicePort: 80<br/>

#### __Creating simple-fanout ingress in k8s:__
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Kubectl create –f simple-fanout.yaml –n &lt;namespace\_name&gt;
#### __Simple-fanout in k8s with contrail:__
![Image of k8s-ingress-simple-fanout](images/k8s-ingress-simple-fanout.png)
#### __K8s-events:__
&nbsp;&nbsp;&nbsp;&nbsp;Kubectl connects to kube-api-server and creates the simps-fanout ingress in k8s.
#### __Contrail-events:__
##### __Contrail-kube-manager:__
Contrail-kube-manager register for event notification for interested resources with kube-api-server which includes ingress. When ingress object is created/modified/deleted in kube-api-server,  It will send an event notification to contrail-kibe-manager. Contrail-kube-manager creates load balancer, virtual-machine-interface for load balancer and instance-ip for virtual-machine-interface from pod-ipam in cluster-network. If the external(public) pool available, it allocates external-ip and associate to the virtual-machine-interface. The fq\_name of the external pool is \[&lt;public\_network_project&gt;,\_\_public\_\_,\_\_fip\_pool\_public\_\_\].
Finally it updates the cluseter-ip and external-ip in k8s.

##### __Contrail-svc-monitor:__
Contrail-svc-monitor listens for event notifications for contrail load balancer objects. When contrail-svc-monitor gets the create load balancer event from contrail-api-server, it generates the haproxy configuration file with simple-fanout ingress spec rules, finds two
active compute nodes for active-standby mode and sends to contrail-vrouter-agents as service-instances through control-node to launch haproxy with generated haproxy config

##### __Contrail-vrouter-agent:__
Contrail-vrouter-agent uses the opencontrail\_vrouter\_netns script to launch the haproxy inside the network namespace with generated haproxy configuration.

#### __Viewing the ingress information:__
##### __k8s:__

###### __To View the http rules:__
&nbsp;&nbsp;&nbsp;&nbsp;kubectl describe ing/simple-fanout –n &lt;namespace\_name&gt;

&nbsp;&nbsp;&nbsp;&nbsp;Name: simple-fanout<br/>
&nbsp;&nbsp;&nbsp;&nbsp;Namespace: default<br/>
&nbsp;&nbsp;&nbsp;&nbsp;Address:<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Default backend: default-http-backend:80 (&lt;none&gt;)<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Rules:<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Host&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Path&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Backends<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;------&nbsp;&nbsp;&nbsp;&nbsp;---------&nbsp;&nbsp;&nbsp;---------------<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;\*<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;/qa.html&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;qa-webserver:80 (&lt;none&gt;)<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;/dev.html&nbsp;&nbsp;&nbsp;&nbsp;dev-webserver:80 (&lt;none&gt;)<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Annotations:<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;No events.<br/>

###### __To View the ip information:__
&nbsp;&nbsp;&nbsp;&nbsp;kubectl get ing/simple-fanout -o=custom-columns=<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;NAME:.metadata.name,<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;CLUSTER-IP:.metadata.annotations.clusterIP,<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;EXTERNAL-IP:.metadata.annotations.externalIP -n &lt;namespace_name&gt;<br/>
&nbsp;&nbsp;&nbsp;&nbsp;NAME&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;CLUSTER-IP&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;EXTERNAL-IP<br/>
&nbsp;&nbsp;&nbsp;&nbsp;simple-fanout&nbsp;&nbsp;&nbsp;&nbsp;10.47.255.248&nbsp;&nbsp;&nbsp;&nbsp;10.84.59.22

##### __Contrail:__
##### __Contrail-api:__
http://&lt;contrail-controller-node&gt;:8082/loadbalancer/&lt;ingress\_uuid&gt; | python -m json.tool
##### __contrail-control:__
http://&lt;contrail-controller-node&gt;:8083/Snh\_IFMapTableShowReq?table\_name=service\_instance&search\_string=
##### __contrail-vrouter-agent:__
###### __ifmap\_agent:__
http://&lt;conril-compute-node&gt;:8085/Snh\_ShowIFMapAgentReq?table\_name=service\_instance&node\_sub\_string=&link\_type\_sub\_string=&link\_node\_sub\_string=
###### __opencontrail\_vrouter\_netns:__
http://&lt;contrail-compute-node&gt;:8085/Snh\_ServiceInstanceReq?uuid=
###### __haproxy configuration:__
Haproxy configuration is stored in /var/lib/contrail/loadbalancer/
haproxy/&lt;ingress\_uuid&gt;/haproxy.conf

###### __Haproxy configuration for simple-fanout:__
> global
>
> daemon
>
> user haproxy
>
> group haproxy
>
> log /dev/log local0
>
> log /dev/log local1 notice
>
> tune.ssl.default-dh-param 2048
>
> ssl-default-bind-ciphers
> ECDH+AESGCM:DH+AESGCM:ECDH+AES256:DH+AES256:ECDH+AES128:DH+AES:ECDH+3DES:DH+3DES:RSA+AESGCM:RSA+AES:RSA+3DES:!aNULL:!MD5:!DSS
>
> ulimit-n 200000
>
> maxconn 65000
>
> stats socket
> /var/lib/contrail/loadbalancer/haproxy/aa13aa8d-1faf-11e7-a423-00259030b0fe/haproxy.sock
> mode 0666 level user
>
> defaults
>
> log global
>
> retries 3
>
> option redispatch
>
> timeout connect 5000
>
> timeout client 300000
>
> timeout server 300000
>
> frontend e602faab-00e2-4ee2-b618-a420d8bb5128
>
> option tcplog
>
> bind 10.47.255.248:80
>
> mode http
>
> option forwardfor
>
> acl 8dc82a38-d61b-4853-96f2-e82c0ff66801\_path path /qa.html
>
> use\_backend 8dc82a38-d61b-4853-96f2-e82c0ff66801 if
> 8dc82a38-d61b-4853-96f2-e82c0ff66801\_path
>
> acl 56c180d1-9205-47d2-8eb7-4dba6ed5a6ae\_path path /dev.html
>
> use\_backend 56c180d1-9205-47d2-8eb7-4dba6ed5a6ae if
> 56c180d1-9205-47d2-8eb7-4dba6ed5a6ae\_path
>
> backend 8dc82a38-d61b-4853-96f2-e82c0ff66801
>
> mode http
>
> balance roundrobin
>
> option forwardfor
>
> server c3ee3971-6207-4239-8112-615c1741a3d5 10.96.13.170:80 weight 1
>
> backend 56c180d1-9205-47d2-8eb7-4dba6ed5a6ae
>
> mode http
>
> balance roundrobin
>
> option forwardfor
>
> server 671990da-51c5-46ea-b995-e3648f1bf2b7 10.98.199.61:80 weight 1

#### __Specifying the external-ip in yaml:__
An option is given to provide the external-ip in the yaml file. If the external-ip is provided, contrail-kube-manager uses the external-ip
otherwise it allocates from the external (public) pool if it is available

##### __Simple-fanout with external(public) ip:__
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;apiVersion: extensions/v1beta1<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;kind: ingress<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;metadata:<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;name: simple-fanout<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;annotations:<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;externalIP: 10.84.59.22<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;spec:<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;rules:<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;- &nbsp;&nbsp;http:<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;paths:<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;-&nbsp;&nbsp;path: /qa.html<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;backend:<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;serviceName: qa-webserver<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;servicePort: 80<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;-&nbsp;&nbsp;path: /dev.html<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;backend:<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;serviceName: dev-webserver<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;servicePort: 80<br/>


# 4. Implementation
## 4.1 Contrail kubernetes manager
Opencontrail implementation requires listening to K8s API messages and create corresponding resources in the Opencontrail API database. New module called "contrail-kube-manager" will run in a container to listen to the messages from the K8s api-server. A new project will be created in Opencontrail for each of the namespaces in K8s.

## 4.2 Contrail CNI plugin

## 4.3 ECMP Loadbalancer for K8s service
Each service in K8s will be represented by a loadbalancer object. The service IP allocated by K8s will be used as the VIP for the loadbalancer. Listeners will be created for the port on which service is listening. Each pod will be added as a member for the listener pool. contrail-kube-manager will listen for any changes based on service labels or pod labels to update the member pool list with the add/updated/delete pods.

Loadbalancing for services will be L4 non-proxy loadbalancing based on ECMP. The instance-ip (service-ip) will be linked to the ports of each of the pods in the service. This will create an ECMP next-hop in Opencontrail and traffic will be loadbalanced directly from the source pod.

## 4.4 Haproxy Loadbalancer for K8s ingress
K8s ingress is represented as a haproxy loadbalancer in contrail. For more information please refer k8s-ingress.md

## 4.5 Security groups for K8s network policy

Network policies can be applied in a cluster configured in isolation mode, to define which pods can communicate with each other or with other endpoints.
The cluster admin will create a Kubernetes API NetworkPolicy object. This is an ingress policy, it applies to a set of pods, and defines which set of pods is allowed access. Both source and destination pods are selected based on labels. The app developer and the cluster admin can add labels to pods, for instance “frontend” / “backend”,  and “development” / “test” / “production”. Full specification of the Network Policy can be found here:

http://kubernetes.io/docs/user-guide/networkpolicies/

Contrail-kube-manager will listen to Kubernetes NetworkPolicy create/update/delete events, and will translate the Network Policy to Contrail Security Group objects applied to Virtual Machine Interfaces. The algorithm will dynamically update the set of Virtual Machine Interfaces as pods and labels are added/deleted.

## 4.6 DNS
Kubernetes(K8S) implements DNS using SkyDNS, a small DNS application that responds to DNS requests for service name resolution from Pods. On K8S, SkyDNS runs as a Pod.


# 5. Performance and scaling impact

## 5.2 Forwarding performance

# 6. Upgrade

# 7. Deprecations

# 8. Dependencies
* Native loadbalancer implementation is needed to support service loadbalancing. https://blueprints.launchpad.net/juniperopenstack/+spec/native-ecmp-loadbalancer
* Health check implementation

# 9. Debugging

# 9.1 Pod IP Address Info:

    The following command can be used to determine the ip address assigned to a pod:

    kubectl get pods --all-namespaces -o wide

    Example:

    [root@k8s-master ~]# kubectl get pods --all-namespaces -o wide
    NAMESPACE     NAME                                 READY     STATUS    RESTARTS   AGE       IP              NODE
    default       client-1                             1/1       Running   0          19d       10.47.255.247   k8s-minion-1-3
    default       client-2                             1/1       Running   0          19d       10.47.255.246   k8s-minion-1-1
    default       client-x                             1/1       Running   0          19d       10.84.31.72     k8s-minion-1-1

# 9.2 Check Pods reachability:

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

# 9.3 Verify that default virtual-network for a cluster is created:

    In the Contrail GUI, verify that a virtual-network named “cluster-network” is created in your project.

# 9.4 Verify a virtual-network is created for an isolated namespace:

    In the Contrail-GUI, verify that a virtual-network with the name format: “<namespace-name>-
    vn” is created.

# 9.5 Verify that Pods from non-isolated namespace CANNOT reach Pods in isolated namespace.

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

# 9.6 Verify that Pods in isolated namespace can reach Pods in in non-isolated namespaces.

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

# 9.7 How to check if a Kubernetes namespace is isolated.

    Use the following command to look at annotations on the namespace:

    Kubectl describe namespace <namespace-name>

    If the annotations on the namespace has the following statement, then the namespace is isolated.

    “opencontrail.kubernetes.isolated” : “true”

        [root@a7s16 ~]# kubectl describe namespace test-isolated-ns
        Name:       test-isolated-ns
        Labels:     <none>
        Annotations:    opencontrail.kubernetes.isolated : true     Namespace is isolated
        Status:     Active

# 10. Testing
## 10.1 Unit tests
## 10.2 Dev tests
## 10.3 System tests

# 11. Documentation Impact

# 12. References