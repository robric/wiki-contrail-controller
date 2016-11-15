
#1. Introduction
Kubernetes (K8s) is an open source container management platform. It provides a portable platform across public and private clouds. K8s supports deployment, scaling and auto-healing of applications. More details can be found at: http://kubernetes.io/docs/whatisk8s/




#2. Problem statement
There is a need to provide pod addressing, network isolation, policy based security, gateway, SNAT, loadalancer and service chaining capability in Kubernetes orchestratation. To this end K8s supports a framework for most of the basic network connectivity. This pluggable framework is called Container Network Interface (CNI). Opencontrail will support CNI for Kubernetes.
 

#3. Proposed solution
Currently K8s provides a flat networking model wherein all pods can talk to each other. Network policy is the new feature added to provide security between the pods. Opencontrail will add additional networking functionality to the solution - multi-tenancy, network isolation, micro-segmentation with network policies, load-balancing etc. Opencontrail can be configured in the following mode in a K8s cluster:

* Cluster isolation
**This is the default mode, and no action is required from the admin or app developer. It provides the same isolation level as kube-proxy. OpenContrail will create a cluster network shared by all namespaces, from where service IP addresses will be allocated.

* Namespace isolation mode
**The cluster admin can configure a namespace annotation to turn on isolation. As a result, services in that namespace will not be accessible from other namespaces, unless security groups or network policies are defined explicitly.

* App isolation mode
**In this finer-grain isolation mode, the admin or app developer can add the label "opencontrail.org/name" to the pod, replication controller and/or service specification, to enable micro-segmentation. As a result, virtual networks will be created for each pod/app tagged with the label. Network policies or security groups will need to be configured to define the rules for service accessibility.


#4. Implementation


#5. Performance and scaling impact

##5.2 Forwarding performance

#6. Upgrade

#7. Deprecations

#8. Dependencies
* Native loadbalancer implementation is needed to support service loadbalancing. https://blueprints.launchpad.net/juniperopenstack/+spec/native-ecmp-loadbalancer
* Health check implementation

#9. Testing
##9.1 Unit tests
##9.2 Dev tests
##9.3 System tests

#10. Documentation Impact

#11. References