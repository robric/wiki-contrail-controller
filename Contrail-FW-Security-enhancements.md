# Introduction
This document addresses security enhancements to contrail product.
* **FW security policy feature enhancements**
* **Decouple routing from security policy**
* _Integrate with third party NG FWs_
* _Support FWAASv2 API_

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

[Import picture from word document for configuration objects and their layout]

### Tags

#### Configuration Tag object

Tag object contains type, value, description, configuration_id.
type is one of the defined tag types, stored as string. 
value is a string. 
description is a string to describe the tag 
configuration_id is a 32bit value 
* 1 bit for global vs local 
* 5 bits for tag types 
* 26 bits for tag values 

As each value is entered by user will create a unique id and maintained and will be used in the configuration_id 
System can have up to 64 million tag values. On an average each tag can up to 2k values, but there are no restrictions per tag. 

Tags/labels can be attached to project, VN, VM, VMI and policy objects. These objects will have tag ref list to support multiple tags. RBAC will control the users to modify or remove the attached tags. Some tags will be attached by system by default or via introspection, these tags are typically facts.

Tag APIs
boolean add_tag_<tag_name> (value) 
boolean delete_tag_<tag_name>(value) 
The above allows us to give RBAC per tag in any object (VMI, VM, Project ….)

Configuration should support the following API also 
Tag query
tags(policy) 
tags (application tag) 
Object query
tags(object) 
tags (type, value) 

Label
label is special tag type, used to assign labels for objects. All the above tag constructs are valid, except that tag type will be ‘label'. 

#### Analytics
Given tag SQL where clause and select clause, analytics should give out objects. Query may contain labels also, whereas labels will have different operators. 
Examples: User might want to know ... 
list of VMIs where ’site == USA and deployment == Production' 
list of VMIs where ’site == USA and deployment == Production has <label name>’ 
Given tag SQL where clause and select clause, analytics should give out flows. 

#### Control node
Control node passes the tags along with route updates to agents and other control nodes.

#### Agent
Agent gets attached tags along with configuration objects. Agent also gets route updates containing tags associated with IP route. This process is similar to getting SG ids along with the route update.

### Address-group

There are multiple ways to add IP address to address-group. 
A) user can manually add IP prefixes to it via configuration. 
B) user can label a work load with address-group’s specified label. 
C) Introspect workloads and based on certain criteria add ip-address to address-group. [Needs discussion]

#### Configuration
address-group object contains label object, description and list of IP prefixes.
label - object is created using the tag APIs.

#### Agent
Agent gets address-group and label objects referenced in policy configuration. Agent uses this address group for matching policy rules.
#### Analytics
Given address group label, get all the objects associated with it. 
Given address group label, get all the flows associated with it. 

### Service-group

#### Configuration
service-group contains list of port list and protocol. 
Whereas, open stack service-group has list of service objects and service object contains the following attributes [1] [2].
Open stack service object contains the following parameters
id, name, service group id, protocol, source_port, destination_port, icmp_code, icmp_type, timeout, tenant id

#### Agent
Agent gets service-group object as it is referred in a policy/rule. Agent uses this service group during policy evaluation. 

### Application-policy

application-policy configuration object contains application_tag, list of NP, list of FWP. This object could be project or global scoped.
application_tag is tag object for application tag. It is allowed only application tags here as the name suggests. 
NP is existing contrail network policies 
FWP is new firewall policy object, which will be detailed in later section. 
Upon seeing application tag for any object, the associated policies will be send to agent. Agent will use this information to find out the list of policies to be applied and their sequence during flow evaluation. User could attach application tag to allowed objects (Project, VN, VM or VMI). 

### Policy-management

Policy-management is a global container object for all policy related configuration. Policy-management object contains - 
Network policies (NPs), Firewall policies (FWPs), Application-policy objects, global-policy objects, global-policy-apply objects 
NPs - List of contrail networking policy objects 
FWPs - List of new firewall policy objects 
Application-policies - List of Application-policy objects 
Global-policies - List of new firewall policy objects, that are defined for global access 
Global-policy-apply - List of global policies in a sequence, and these policies applied during flow evaluation. 

Network Policies (NP) references in the policy-management is long term plan, these changes are not required in the first release. NP policies will be available, as they are today.

### Firewall-policy
We discussed to use existing network policy, instead of creating new firewall-policy object. But, network policy supports both connectivity and policy.
We decided to keep a separate new object for firewall policy to do all firewall related features, and network policy could slowly move towards to connectivity and includes service chain connectivity. 
firewall-policy object will keep adding firewall related features like DDOS, filtering features and etc.…
Firewall-policy is a new policy object, which contains list of firewall-rule-objects and audited flag. Firewall-policy could be project or global scoped depends on the usage.

audited
boolean flag to indicate that, owner of the policy indicated that policy is audited. Default is False, and will have to explicitly set to True after review.

Generate a log event for audited with timestamp and user details.






