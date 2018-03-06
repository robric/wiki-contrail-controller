# Introduction

Contrail FW Security Policy allows decoupling of routing from security policies and provides multi dimension segmentation and policy portability, while significantly enhancing user visibility and analytics functions. 

Contrail FW Security Policy introduces the concept of tags to achieve multi-dimension traffic segmentation among various entities, with security features. Tags are key-value pairs associated with different entities in the deployment. Tags can be pre-defined or custom/user defined.

Kubernetes Network Policy is a specification of how groups of kubernetes workloads (hereafter referred to as "pods") are allowed to communicate with each other and other network endpoints. NetworkPolicy resources use labels to select pods and define rules which specify what traffic is allowed to the selected pods.

# Problem statement

This document discusses implementation of Kubernetes Network Policy in Contrail using Contrail FW Security Policy framework.

While Kubernetes network policy can be implemented using other security objects in Contrail like Security Groups, Contrail network policies etc, the support of tags by Contrail FW Security Policy aids in the simplification and abstraction of workloads.


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

# Proposed solution 
## Representing Kubernetes Network Policy as Contrail FW Securty Policy:

Kubernetes and Contrail FW Policy are different in terms of the semantics in which network policy is specified in each. The key to efficient implementation of kubernetes network policy via Contrail FW policy is in mapping of the corresponding configuration constructs between these two entities. 

We propose to mapping the constructs as follows:

| Kubernetes Network Policy Constructs | Contrail FW Policy Constructs |
| --- | --- |
| Label                                         |  Custom Tag (one for each label) |
| Namespace                                       | Custom Tag (one for each namespace) |
| Network Policy                                  | Firewall Policy (one FP per Network Policy) |
| Ingress Rule                                    | Firewall Rule (one FW rule per Ingress Rule) |
| Ingress CIDR Rules                              | Address Group |
| Cluster                                         | Default Application Policy Set |

**NOTE**

The Contrail FW Policy constructs will be created in the "global" scope, if the kubernetes cluster is a standalone cluster.

The Contrail FW Policy constructs will be created in the "project" scope, if the kubernetes cluster is a nested cluster. The project in 
which these constructs are created will be the one that houses the cluster.

## Naming Convention
### Contrail Firewall Policy
Contrail Firewall Policy create for a K8s network policy will named in the following format:

    < Namespace-name >-< Network Policy Name >

For example: Network policy "world" in namespace "Hello" will be named: Hello-world

### Contrail Firewall Rule
Contrail Firewall Rules create for a K8s network policy will named in the following format:

    < Namespace-name >-<PolicyType>-< Network Policy Name >-<Index of from/to blocks>-<selector type>-<rule-index>-<svc/port index>

For example:
```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: world
  namespace: hello
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          role: frontend
```
A Rule corresponding to the above policy would be named:

    hello-ingress-world-0-podSelector-0-0

## Illustration: 1  
### Sample Kubernetes Network Policy 

```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: test-network-policy
  namespace: default
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - ipBlock:
        cidr: 172.17.0.0/16
        except:
        - 172.17.1.0/24
    - namespaceSelector:
        matchLabels:
          project: myproject
    - podSelector:
        matchLabels:
          role: frontend
    ports:
    - protocol: TCP
      port: 6379
  egress:
  - to:
    - ipBlock:
        cidr: 10.0.0.0/24
    ports:
    - protocol: TCP
      port: 5978

```
### Sample Contrail FW Policy 
The test-network-policy defined in kubernetes will result in the following objects being created in Contrail.

**Tags**

The following tags will be created, if they do not exist.
In a regular workflow, these tags would have been created by the time the namespace and pods were created.

| Key | Value |
| --- | --- |
| role | db |
| namespace | default |

### Address Groups
| Name | Prefix |
| --- | --- |
| 172.17.1.0/24 | 172.17.1.0/24 |
| 172.17.0.0/16 | 172.17.0.0/16 |
| 10.0.0.0/24 | 10.0.0.0/24 |


### Firewall Rules
| Rule Name | Action | Services | Endpoint1 | Dir | Endpoint2 | Match Tags |
| --- | --- | --- | --- | --- | --- | --- |
| default-ingress-test-network-policy-0-ipBlock-0-172.17.1.0/24-0 | deny | tcp:6379 | Address Group: 172.17.1.0/24 | > | role=db && namespace=default| |
| default-ingress-test-network-policy-0-ipBlock-0-cidr-172.17.0.0/16-0	| pass | tcp:6379 | Address Group: 172.17.0.0/16 | > |role=db && namespace=default | |
| default-ingress-test-network-policy-0-namespaceSelector-1-0 | pass  | tcp:6379 | project=myproject | > | role=db && namespace=default | |
| default-ingress-test-network-policy-0-podSelector-2-0	| pass | tcp:6379 | namespace=default && role=frontend | > | role=db && namespace=default ||
| default-egress-test-network-policy-ipBlock-0-cidr-10.0.0.0/24-0 | pass | tcp:5978 | role=db && namespace=default | > | Address Group: 10.0.0.0/24 | |

**Firewall Policy**

| Name | Rules |
| --- | --- |
| default-test-network-policy | default-ingress-test-network-policy-0-ipBlock-0-172.17.1.0/24-0, default-ingress-test-network-policy-0-ipBlock-0-cidr-172.17.0.0/16-0, default-ingress-test-network-policy-0-namespaceSelector-1-0, default-ingress-test-network-policy-0-podSelector-2-0, default-egress-test-network-policy-ipBlock-0-cidr-10.0.0.0/24-0	|

## Illustration: 2  
### Default allow all ingress traffic

**Kubernetes Network Policy**

The below policy  explicitly allows all traffic for all pods in that namespace.
```
	apiVersion: networking.k8s.io/v1
	kind: NetworkPolicy
	metadata:
	  name: allow-all-ingress
	spec:
	  podSelector:
	  ingress:
	  - {}
```

**Contrail Firewall Security Policy:**

**Tags**
The following tags will be created, if they do not exist.
In a regular workflow, these tags would have been created by the time the namespace and pods were created.

| Key | Value |
| --- | --- |
| namespace | default |

**Address Groups**

None

**Firewall Rules**

| Rule Name | Action | Services | Endpoint1 | Dir | Endpoint2 | Match Tags |
| --- | --- | --- | --- | --- | --- | --- |
| default-ingress-allow-all-ingress-0-allow-all-0 | pass | any | any | > | namespace=default | |

**Firewall Policy**

| Name | Rules |
| --- | --- |
| default-allow-all-ingress | default-ingress-allow-all-ingress-0-allow-all-0 |

## Illustration: 3
### Default deny all ingress traffic.

**Kubernetes Network Policy**

The below policy  explicitly denies all traffic to all pods in that namespace.
```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-ingress
spec:
  podSelector:
  policyTypes:
  - Ingress
```

**Contrail FW Security Policy**

**Tags**

The following tags will be created, if they do not exist.
In a regular workflow, these tags would have been created by the time the namespace and pods were created.

| Key | Value |
| --- | --- |
| namespace | default |

**Address Groups**

None

**Firewall Rules**

| Rule Name | Action | Services | Endpoint1 | Dir | Endpoint2 | Match Tags |
| --- | --- | --- | --- | --- | --- | --- |

    NOTE: The implicit behavior of any network policy is to deny traffic not matching explicit allow flows. 
          However in this policy, there are no explicit allow rules.
          Hence there are no firewall rules created for this policy.

**Firewall Policy**

| Name | Rules |
| --- | --- |
| default-deny-ingress |  |

## Illustration: 4
### Default allow all egress traffic.

**Kubernetes Network Policy**

The below policy explicitly allows all traffic from all pods in that namespace.
```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-all-egress
spec:
  podSelector:
  egress:
  - {}
```
**Contrail Firewall Security Policy**

**Tags**

The following tags will be created, if they do not exist.
In a regular workflow, these tags would have been created by the time the namespace and pods were created.

| Key | Value |
| --- | --- |
| namespace | default |

**Address Groups**

None

**Firewall Rules**

| Rule Name | Action | Services | Endpoint1 | Dir | Endpoint2 | Match Tags |
| --- | --- | --- | --- | --- | --- | --- |
| default-egress-allow-all-egress-allow-all-0 | pass | any | namespace=default | > | any | |

**Firewall Policy**

| Name | Rules |
| --- | --- |
| default-allow-all-egress | default-egress-allow-all-egress-allow-all-0 |

## Illustration: 5 
### Default deny all egress traffic.

**Kubernetes Network Policy**

The below policy  explicitly denies all traffic from all pods in that namespace.
```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all-egress
spec:
  podSelector: {}
  policyTypes:
  - Egress
```

**Contrail Firewall Security Policy**

**Tags**

The following tags will be created, if they do not exist.
In a regular workflow, these tags would have been created by the time the namespace and pods were created.

| Key | Value |
| --- | --- |
| namespace | default |

**Address Groups**

None

**Firewall Rules**

| Rule Name | Action | Services | Endpoint1 | Dir | Endpoint2 | Match Tags |
| --- | --- | --- | --- | --- | --- | --- |

    NOTE: The implicit behavior of any network policy with egress policy type
          is to deny egress traffic not matching explicit egress allow flows.
          In this policy, there are no explicit egress allow rules.
          Hence there are no firewall rules created for this policy.

**Firewall Policy**

| Name | Rules |
| --- | --- |
| default-deny-all-egress ||

## Illustration: 6
### Default deny all ingress and egress traffic.

**Kubernetes Network Policy**

The below policy explicitly denies all traffic from all pods in that namespace.
```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all-ingress-egress
spec:
  podSelector:
  policyTypes:
  - Ingress
  - Egress
```

**Contrail Firewall Security Policy**

**Tags**

The following tags will be created, if they do not exist.
In a regular workflow, these tags would have been created by the time the namespace and pods were created.

| Key | Value |
| --- | --- |
| namespace | default |

**Address Groups**

None

**Firewall Rules**

| Rule Name | Action | Services | Endpoint1 | Dir | Endpoint2 | Match Tags |
| --- | --- | --- | --- | --- | --- | --- |

    NOTE: The implicit behavior of any network policy with ingress/egress policy
          type is to deny corresponding traffic not matching explicit allow flows.
          In this policy, there are no explicit allow rules.
          Hence there are no firewall rules created for this policy.

**Firewall Policy**

| Name | Rules |
| --- | --- |
| default-deny-all-ingress-egress | |

# Cluster-wide Action Enforcement

The spec and the syntax of network policy allow for maximum flexibility and varied combinations. The framework
provides the user with all possible bells and whistles so as to not limit the user's imagination. With such great power, it is quite possible to shoot onself at the foot.

Lets assume a case where two network policies are created:

         Policy 1: Pod A can send to Pod B.
         Policy 2: Pod B can only recieve from Pod C.

These policies make sense when looking at source and destination endpoints independently.

However from a networking flow perspective, there is an inherent contradiction between the above policies. 
Policy1 states that a flow from PodA to PodB is allowed.
Subsequent policy implies that flow from PodA to PodB is not allowed.

From a networking narrative, Contrail prioritizes flow behavior as more critical.
In the event of inherent contradiction in network policies, Contrail will honor the flow perspective.

One of the core aspects of this notion is:

    If a policy matches a flow, the action is honored cluster-wide.
    
i.e If a flow matches a policy at the source, the flow will match the same policy in the  destination as well. 

So going back to our prior example:

         Policy 1: Pod A can send to Pod B.
         Policy 2: Pod B can only recieve from Pod C.

The flow behavior in a Contrail managed kubernetes cluster will be as follows:

    1. Flow from PodA to PodB will be allowed. (due to Policy 1)
    2. Flow from PodC to PodB will be allowed. (due to Policy 2)
    3. Any other flow to PodB will be disallowed. (due to Policy 2)

**Illustrations:**

1. Egress allow all and Ingress deny all

   Setup:

   Namespace NS1 has two pods, PodA and PodB.

   Policy:

   A network policy applied on namespace NS1 states:
   
   Rule 1. Allow all egress traffic from all pods in NS1.
   Rule 2. Deny all ingress traffic to all pods in NS1.

   Behavior:

   * PodA can send traffic to PodB. (due to rule 1)
   * PodB can send traffic to PodA. (due to rule 1)
   * PodX from a different namespace cannot send traffic to PodA or PodB(Due to rule 2)

2. Ingress allow all and Egress deny all

   Setup:

   Namespace NS1 has two pods, PodA and PodB.

   Policy:

   A network policy applied on namespace NS1 states:

   Rule 1. Allow all ingress traffic to all pods in NS1.
   Rule 2. Deny all egress traffic from all pods in NS1.

   Behavior:

   * PodA can send traffic to PodB. (due to rule 1)
   * PodB can send traffic to PodA. (due to rule 1)
   * PodA and PodB cannot send traffic to pods in any other namespace.

3. Egress CIDR rule.
   
   Setup:

   Namespace NS1 has two pods, PodA and PodB.

   Policy:

   A network policy applied on namespace NS1 states:

   Policy 1: Allow PodA to send traffic to CIDR of PodB.
   Policy 2: Deny all ingress traffic to all pods in NS1.

   Behavior:

   * PodA can send traffic to PodB. (due to Policy 1)
   * All other traffic to PodA and PodB will be dropped. (due to policy 2)

# Implementation

Contrail-kube-manager is the glue that binds the kubernetes and Contrail world together.
This daemon connects to the api server of Kubernetes cluster/s and receives all events from them.
It coverts kubernetes events, including Network policy events, into appropriate Contrail objects.

With respect to Kuberneter Network Policy, contrail-kube-manager will implement the following behavior:

1. Will create on Contrail Tag for each Kubernetes Label.
2. Will create one FW policy per Kubernetes Network Policy. 
3. Will create one Default Application Set to represent the cluster. All FW policies created in that cluster
   will be attached to this default application set.
4. Modifications to exiting Kubernetes networking policy will result in the corresponding FW policy being updated.
5. New network policies will always be added to the front of the list of FW policies in the default application set.
   This is so that, the latest rules that may potentially overlap existing policy behavior is always honored. 

# Limitation / Errata

1. Pod label with key as "Application" should not be used
  
   Label with key "Application" is reserved keyword in Contrail and has special meaning and semantics.
   If a pod has a label with key "Application", the enforcement of network policy specified for the Pod is
   not guaranteed/undefined. So the recommendation in release 5.0 is to not created Pod's with label
   key "Application".
   
   https://bugs.launchpad.net/juniperopenstack/+bug/1749902

# References

Contrail FW Security Policy                                                    https://github.com/Juniper/contrail-controller/wiki/Contrail-FW-Security-enhancements

Kubernetes Network Policy                 https://github.com/mironov/kubernetes/blob/master/docs/proposals/network-policy.md