
#1. Introduction
Kubernetes (K8s) is an open source container management platform. It provides a portable platform across public and private clouds. K8s supports deployment, scaling and auto-healing of applications. More details can be found at: http://kubernetes.io/docs/whatisk8s/




#2. Problem statement
There is a need to provide pod addressing, network isolation, policy based security, gateway, SNAT, loadalancer and service chaining capability in Kubernetes orchestratation. To this end K8s supports a framework for most of the basic network connectivity. This pluggable framework is called Container Network Interface (CNI). Opencontrail will support CNI for Kubernetes.
 

#3. Proposed solution
Currently K8s provides a flat networking model wherein all pods can talk to each other. Network policy is the new feature added to provide security between the pods. Opencontrail will add additional networking functionality to the solution. There are multiple modes wherein Opencontrail can be configured in a K8s cluster:

##* Cluster mode
##* Namespace isolation mode
##* App isolation mode
##* User defined mode


##3.1 Alternatives considered
####Describe pros and cons of alternatives considered.

##3.2 API schema changes
####Describe api schema changes and impact to the REST APIs.

##3.3 User workflow impact
####Describe how users will use the feature.

##3.4 UI changes
####Describe any UI changes

##3.5 Notification impact
####Describe any log, UVE, alarm changes

#4. Implementation


##4.2 Work items
####Describe changes needed for different components such as Controller, Analytics, Agent, UI. 
####Add subsections as needed.

#5. Performance and scaling impact
##5.1 API and control plane
####Scaling and performance for API and control plane

##5.2 Forwarding performance
####Scaling and performance for API and forwarding

#6. Upgrade
####Describe upgrade impact of the feature
####Schema migration/transition

#7. Deprecations
####If this feature deprecates any older feature or API then list it here.

#8. Dependencies
####Describe dependent features or components.

#9. Testing
##9.1 Unit tests
##9.2 Dev tests
##9.3 System tests

#10. Documentation Impact

#11. References