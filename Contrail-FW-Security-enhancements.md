# Introduction
This document addresses security enhancements to contrail product.
* FW security policy feature enhancements
* Decouple routing from security policy
_* Integrate with third party NG FWs_
_* Support FWAASv2 API_

This feature development addresses ‘Decoupling routing from security policy’ and ‘FW security policy feature enhancements’. It also considers future support for FWAASv2 API, while developing this feature. 

# Problem statement

Network security is orthogonal to network topology. Network topology addresses some of the network security elements. OpenStack tenant networks are isolated, and can’t communicate by default. Virtual networks with in a tenant requires neutron router for connectivity, in a way traffic is isolated/segmented between networks. Tenant is isolated from other tenants.

Contrail network policy provides security between networks by allow/deny certain traffic. Contrail network policy also provides connectivity between virtual networks.
 
OpenStack security groups allow access between workloads/instances for specified traffic types and rest denied.

Security Policy model for a given customer needs to map to above OpenStack/Contrail network policy and security group constructs.
 
Customer deployments may contain multiple dimension entities (multiple deployments, multiple applications, multiple tiers). Security policy model might contain many ways of cross cutting those dimensions to control traffic among workloads.
 
User might want to segregate traffic on the following categories 
* Site: Site could be Country or City or Rack or Region or all together Country/City/Rack or it could be any other arbitrary ways of dividing place
* OS: User might want to communicate among same OS
* Environment: Modeling, Testing, production
* Application: HR, salesforce app, oracle ordering app
* Workload type: low sensitivity, financial, or personal identifiable information
* Application-Tier: web tier, database tier
* Many more ...

User wants to cross section between above categories, which makes it harder to express with existing network policy/security group constructs.  

Addressing this requirement with Security Groups will be tough, and have create combinatorial SGs to address it. As customer comes with different ways to segregate traffic, then number of SGs will explode and also it is not possible without changing the definition of SGs and their attachment at workloads. 

A customer scenario was to have multi-tier application, that supports both development and production environment workloads. Security requirement is to application shouldn’t have cross environment traffic, which is a simple ask. It became pretty hard to do with existing constructs. Customer started with a VN per tier (VNweb, VNapp, VNdb) and have SGs to support application isolation (SGhr, SGsalesforce, SGemail), now it is hard to environment isolation without exploding VNs or SGs.
 
Now image if you have 100 applications and 10 environments, somebody might have to create 1000 SGs to manage them and also any cross relation between environments won’t work. 
It might work some specific cases, but not all.

Workday like customer wants to segment traffic based on Application and Deployment, it was hard to solve using SGs, will produce too many rules in the SGs.  As number of applications grows, it is impossible to manage rule explosion.

# Proposal for FW security policy enhancements

This feature introduces decoupling of routing from security policies, multi dimension segmentation and policy portability. This feature also enhances user visibility and analytics functions.

## Routing and policy decoupling

Network Policy objects are tightly coupled with routing. Hence, we are introducing new firewall policy objects, which decouples policy from routing. 

## Multi dimension segmentation

As part of enhancing FW features, we are introducing multi dimension segmentation.
Multi dimension segmentation (Example dimensions: Application, Tier, Deployment, Site, UserGroup and etc...) is to segment traffic based on multiple dimensions of entities with security features.

## Policy portability

Customers are looking for portability of their security policies to different environments. Portability might be required ‘from development to production’, ‘from pci-complaint to production’, ‘to bare metal environment’ and ‘to container environment’. 

## Visibility and Analytics
[Anish]

# Proposed solution

This proposal is to address multi dimension traffic segmentation using tags in security policy definitions. High level idea is to use tag regular expressions in the source and destination fields of policy rules. These tag regular expressions will give cross sections of tag dimensions. 

Also today, policies suffer portability from one environment to other, this proposal also enhances with match condition to address this issue. Match tags will be added to policy rule to match tag values of source and destination workloads without mentioning tag values. Example is ‘allow protocol tcp source application-tier=web destination application-tier=application match application and site’. application and site values should match for this rule to take effect.

We would like to limit tags types to make it easier for customers to consume. The following tag types are defined/allowed, and additional tags will be added later as needed. Contrail supports up to 32 tag types.

Proposal also considers FWAASv2, while creating the policy objects.

## Predefined Tags

Predefined tags are chosen based on customer requirements and also look into existing deployment issues. User can pick relevant tag values for their environment. Tag values are unique.

Predefine tag types are -
* application 
* application-tier 
* deployment 
* site 
* user 
* compliance 
* label (is a special tag, allows the user to label objects) 

Example usage of tags, as follows
application = HRApp
application-tier = Web
site = USA

## Implicit tags (facts)
Implicit tags are facts of an existing environment. These tags could be added as part of provisioning or configured manually.
* compute node 
* rack 
* pod 
* cluster 
* dc 

## Tagging objects

User can tag objects Project, VN, VM, VMI with tags and values, to map their security requirements. Tags follow hierarchy of Project, VN, VM and VMI and inherited in that order. This gives an option for user to provide default settings for any tags at any level. As mentioned earlier, policies could specify their security in term of tagged end points. Policies also can express in terms of ip prefix, network and address groups end points. 

## Policy application

Policy application is via application tag. Policy application is new object. User can create list of policies per application, to be applied during the flow acceptance evaluation. Introducing global scoped policies and project scoped policies. Some policies can be defined/applied globally for all the projects, others project specific policies. 

## Configuration objects

Identified the following objects as starting point for new security features.
* firewall-policy 
* firewall-rule 
* policy-management 
* application-policy 
* service-group 
* address-group 
* tag 
* global-apply
