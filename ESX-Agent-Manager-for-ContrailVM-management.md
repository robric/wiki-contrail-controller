1. Introduction

Contrail integration with VMware runs contrail vrouter in a VM (ContrailVM) on 
each of the ESXi hosts in the vcenter cluster. Today, the ContrailVM provisioned 
and setup is like any other tenant VM running in the cluster.

2. Problem statement

The problems tried to address here are:
* automate the provisioning of ContrailVMs
* provide more privileges to ContrailVMs 
* manage and monitor the ContrailVMs

3. Proposed solution

VMware provides a standard vCenter solution called vSphere ESX Agent Manager (EAM),
this allows other solutions to deploy, monitor, and manage ESX agents (VMs) on ESXi hosts.
ESX Agent Manager performs the following functions:
* Provisions ESX agent virtual machines for solutions.
* Monitors changes to the ESX agent virtual machines and their scope in vCenter Server.
* Reports configuration issues in the ESX agents to the solution.
* Integrates agent virtual machines with vSphere features such as Distributed Resource Scheduler (DRS),
* Distributed Power Management (DPM), vSphere High Availability (HA), fault tolerance, maintenance mode, 
  and operations such as adding and removing hosts to and from clusters.
* Every vCenter Server instance contains a running ESX Agent Manager.

The proposal is to use ESX Agent Manager to provision, manage and monitor ContrailVMs that 
run on ESXi hosts. 

This involves the below:
* A solution (ContrailVM Manager), which is a vCenter extension that implements the Extension 
  data object and register with vCenter’s ExtensionManager and SolutionManager.
* ContrailVM Manager solution integrates with ESX Agent Manager and then the ESX 
  Agent Manager provisions and monitors ContrailVMs for the ContrailVM Manager.

To integrate a solution with ESX Agent Manager, the ContrailVM Manager solution must meet 
the following requirements:
* The solution must be a vCenter extension that implements the Extension data object 
  and registers  with ExtensionManager.
* Use Open Virtualization Format (OVF) to package ESX agent virtual machines or vApps. 
  ESX Agent Manager only supports the deployment of virtual machines using OVF.
* Use HTTP or HTTPS to publish OVF files to ESX Agent Manager.
* Use vSphere installation bundles (VIB) to add functions to ESXi hosts, for example to 
  add VMkernel modules or custom ESX Server applications to ESXi hosts.
* Use HTTP or HTTPS to publish VIB files to ESX Agent Manager.
* Use vCenter Server Compute Resources to define the ESX agent scope.

3.1 Alternatives considered

Describe pros and cons of alternatives considered.

3.2 API schema changes

Describe api schema changes and impact to the REST APIs.

3.3 User workflow impact

Describe how users will use the feature.

3.4 UI changes

Describe any UI changes

3.5 Notification impact

Describe any log, UVE, alarm changes

4. Implementation

4.1 Work items

**ContrailVM Manager** 
Develop a vCenter extension named “ContrailVM Manager” and register with ExtensionManager
and SolutionManager.

**Integrate with Esx Agent Manager**
Integrate the ContrailVM Manager solution with Esx Agent Manager.
Use Esx Agent Manager API to spawn ContrailVMs from the OVF file.

pyvmomi provides Eamobjects.py as an interface to the EAM API.

**Contrail vCenter Plugin**
contrail-vcenter-plugin should be distributed and run per ESXi host.

5. Performance and scaling impact

5.1 API and control plane

Scaling and performance for API and control plane

5.2 Forwarding performance

Scaling and performance for API and forwarding

6. Upgrade

Describe upgrade impact of the feature

Schema migration/transition

7. Deprecations

If this feature deprecates any older feature or API then list it here.

8. Dependencies

Describe dependent features or components.

9. Testing

9.1 Unit tests

9.2 Dev tests

9.3 System tests

10. Documentation Impact

11. References

https://pubs.vmware.com/vsphere-50/index.jsp?topic=%2Fcom.vmware.vsphere.ext_solutions.doc_50%2FGUID-A0B694CC-9B63-4147-9B4F-E585AC57DBDF.html

http://pubs.vmware.com/vsphere-6-5/index.jsp?topic=%2Fcom.vmware.vsphere.ext_solutions.doc%2FGUID-AC9DC022-2FBB-43FE-9CA6-5B3CD6AAD42D.html

http://pubs.vmware.com/vsphere-6-5/index.jsp?topic=%2Fcom.vmware.vsphere.ext_solutions.doc%2FGUID-BCF564C0-3CCA-4067-99D1-5C90D86914B1.html
