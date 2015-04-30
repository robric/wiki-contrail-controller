Welcome to the Opencontrail wiki!

Overview:
* [Project Overview](https://github.com/Juniper/contrail-controller/wiki/Contrail:-Project-Overview)
* [Installation and provisioning](https://github.com/Juniper/contrail-controller/wiki/OpenContrail-bring-up-and-provisioning)
* [Deploy OpenContrail 1.06](Install-and-Configure-OpenContrail-1.06)

Architecture:

* [Roles, Daemons and Ports](Roles-Daemons-Ports)
* [OpenContrail Internal Services](OpenContrail-Internal-Services)

Opencontrail Development:
* [Continuous Integration (CI)](OpenContrail-Continuous-Integration-(CI))
* [Code submission + review checklist](Code-Review-Checklist)

Provisioning:  
* [TTL configuration for Analytics](TTL-configuration-for-analytics-data)

Debug and Troubleshooting:
* [Checklist](Debug-Checklist)
* [Tips](Debug-Tips)
* [VRouter-Agent introspect](Contrail-Vrouter-Agent---Introspect)
* [Virtual DNS](vDNS-Debugging)
* [Scenarios](Scenario-Troubleshooting)

Maintenance Procedures:
* [Removing + Adding DB node](Removing_Adding_DB_Node)
* [Adding and Removing a hypervisor/compute node](Adding_Removing_Compute_Node)
* [Increasing Limits (nfiles/nprocs) for a service](Increasing_Service_Limits)
* [Analytics DB usage and purge](Contrail-Analytics-DB-data-purge)

Features:
* [Simple Gateway](Simple-Gateway)
* [Virtual DNS and IPAM](DNS-and-IPAM)
* [Layer2 EVPN](EVPN)
* [Link Local Services](Link-local-services)
* [Extra DHCP options configuration](Extra-DHCP-Options)
* [Network Rate Limiting Configuration] (https://techwiki.juniper.net/Documentation/Contrail/Contrail_Controller_Feature_Guide/Configuration/Configuring_Network_QoS_Parameters_in_VM)
* [VPC API support](VPC-API-support)
* [Support for Baremetal](Baremetal-Support)

Release specific Notes:
* R1.10
  + [Neutron API compatibility](Neutron-API-Support)
  + [Process Name Changes](Contrail-process-names'-changes-in-R1.10)

* R2.0
  + For build 22 and below, to enable sslv3 with ifmap, comment the following in `/etc/java-7-openjdk/security/java.security` and `service supervisor-config restart`

        ``
jdk.tls.disabledAlgorithms=SSLv3
        ``