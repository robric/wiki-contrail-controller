From R2.1, OpenContrail supports extending a cluster to include baremetal servers and other virtual instances connected to a TOR switch supporting OVSDB protocol. The baremetal servers and other virtual instances can be configured to be part of any of the virtual networks configured in the contrail cluster, facilitating communication between them and the virtual instances running in the cluster. Contrail policy configurations can be used to control this communication.

The solution is achieved by using OVSDB protocol to extend contrail control plane into the TOR switch. VXLAN encapsulation will be used to extend contrail data plane to the TOR switch, to communicate with the baremetal servers and other virtual instances.

## TOR Services Node (TSN)

A node provisioned in a new role, called TOR Services Node (or TSN), is introduced to support this. TSN will act as the multicast controller for the TOR switches and will also provide DHCP and DNS services to the baremetal servers or virtual instances running behind TOR ports.

As the multicast controller, TSN will receive the broadcast packets from the TOR, which are then replicated to the required compute nodes and other EVPN nexthops. Broadcast packets from the virtual machines in the cluster are sent from the respective compute nodes to the TOR switch directly.

Multiple TSN nodes can be configured in the system based on the scaling needs of the cluster.

## Contrail TOR Agent

For each TOR switch to be integrated in the OpenContrail cluster, one TOR agent is provisioned. A TOR agent acts as the OVSDB client - all the OVSDB interactions with the TOR are done via this. It programs the different OVSDB tables on the TOR switch and receives the local unicast table entries from it.

While TOR agents can be run on any node in the cluster, typical practice is to run it on the TSN node.

## Configuration

Contrail WebUI can be used to configure a TOR switch and interfaces on the switch.

In "Configure - Physical Devices - Physical Routers" page, create an entry for the TOR switch, providing the TOR's IP address, VTEP address. The TSN and TOR agent addresses for this TOR should also be configured here.

In "Configure - Physical Devices - Interfaces" page, add the logical interfaces to be configured on the TOR. Please note that the Name of the logical interface should match the name on the TOR (eg, ge-0/0/0.10). Other logical interface configurations like Vlan id, Mac address and IP address of the baremetal server and the virtual network it belongs to can be configured here.

## Provisioning

Fab scripts are provided to provision a TSN. The following changes are required in your testbed.py:
1. In env.roledefs, add hosts for 'tsn' and 'toragent' roles. The TSN node should also be configured in 'compute' role as well.
1. Add the TOR agent configuration as follows:
    env.tor_agent = {                                                                  
                     host2: [{                                                                      
                              'tor_ip':'<ip address>',      # IP address of the TOR                                    
                              'tor_id':'<1>',               # Numeric value to uniquely identify the TOR                                                
                              'tor_type':'ovs',             # always ovs                                     
                              'tor_ovs_port':'9999',        # the TCP port to connect on the TOR                                     
                              'tor_ovs_protocol':'tcp',     # always tcp, for now                                     
                              'tor_tsn_ip':'<ip address>'   # IP address of the TSN for this TOR                                    
                      }]                                                                     
    }

Use the tasks add_tsn and add_tor_agent to provision the TSN and TOR Agents.

## Prior Configuration required on QFX5100

* Enable OVSDB
* Set the connection protocol 
* Indicate the interfaces that will be managed via OVSDB
* Configure switch options with VTEP source (in the example below, we configured it to be lo0.0 – need to ensure that this address is reachable from TSN node)

set interfaces lo0 unit 0 family inet address <router-id-reachable-on-ip-fabric>
set switch-options ovsdb-managed
set switch-options vtep-source-interface lo0.0
set protocols ovsdb passive-connection protocol tcp port <port-number>
set protocols ovsdb interfaces <interfaces-to-be-managed-by-ovsdb>