Contrail FW Security Policy allows decoupling of routing from security policies and provides multi dimension segmentation and policy portability, which significantly enhancing user visibility and analytics functions. 

Contrail FW Security Policy introduces the concept of tags to achieve multi-dimension traffic segnmentation.
Multi dimension segmentation is to segment traffic based on multiple dimensions of entities with security features. Tags are key-value pairs associated with different entities in the deployment. Tags can be predefined or custom defined.

Kubernetes Network Policy is a specification of how groups of kubernetes workloads (hereafter referred to as "pods") are allowed to communicate with each other and other network endpoints. NetworkPolicy resources use labels to select pods and define rules which specify what traffic is allowed to the selected pods.

This wiki discusses implementation of Kubernetes Network Policy in Contrail using Contrail FW Security Policy framework.

# Kubernetes Network Policy
Kubernetes Network Policy specification has the following requirements.

1. Network policy is pod specific i.e the policy specification applies to a pod or a group of pods.
   If a specified network policy applies to a pod, the traffic to the pod is dictated by rules of the network policy.
2. In the absence of network policy specification that applies to it, a pod should accept all traffic from all sources.
3. A network policy may define traffic rules for a pod at the ingress, egress or both. 
   If no direction is specified, then the specified policy will be applied to ingress by default.
4. When network policy is applied to a pod, the policy should have explicit rules to specify a whitelist of permitted
  traffic in the egress and egress. All traffic that does not match the whitelist rules are to be denied and dropped.
5. Mutiple network policies can be applied on any pod. Traffic matching ANY of the network policies should be permitted.
6. NetworkPolicy act on connections rather than individual packets. That is to say that if traffic from pod A to
   pod B is allowed by the configured policy, then the return packets for that connection from B -> A are also allowed, even if the policy in place would not allow B to initiate a connection to A. 
7. If new network policy is applied that would block an existing connection between two endpoints, then the existing connection should
   terminated and blocked as soon as can be expected by the implementation.
8. Ingress Policy:
   An ingress rule consists of the identity of the source and the type(i.e. protocol/port) of traffic from the source that is allowed to be forwarded to a pod.
   The identify of source can be of type:
   	 * CIDR block

   	   If the source IP is from the CIDR and the traffice matches the protocol:port, then traffic will be forwarded to the pod.
   	 * Kubernetes Namespace

   	   Namespace selectors identify namespaces, whose pods can send the defined protocol:port traffic to the ingress pod.
   	 * Pods

   	   Pod selectors identify the pods in the namespace corresponding to the network policy, that can send matching protocol:port traffic to the ingress pods.
9. Egress Policy:
    This specifies a whitelist CIDR to which a partificular protocol:port traffic is permitted, from the pods targeted by this network policy
