From R2.1, OpenContrail supports extending a cluster to include baremetal servers and other virtual instances connected to a TOR switch supporting OVSDB protocol. The baremetal servers and other virtual instances can be configured to be part of any of the virtual networks configured in the contrail cluster, facilitating communication between them and the virtual instances running in the cluster. Contrail policy configurations can be used to control this communication.

The solution is achieved by using OVSDB protocol to configure the TOR switch and to import dynamically learnt addresses from it. VXLAN encapsulation will be used in the data plane communication with the TOR switch.

## TOR Services Node (TSN)

A node provisioned in a new role, called TOR Services Node (or TSN), is introduced. TSN will act as the multicast controller for the TOR switches and will also provide DHCP and DNS services to the baremetal servers or virtual instances running behind TOR ports.

TSN will receive all the broadcast packets from the TOR, which are then replicated to the required compute nodes in the cluster and to other EVPN nodes. Broadcast packets from the virtual machines in the cluster are sent from the respective compute nodes to the TOR switch directly.

TSN can act as the DHCP server for the baremetal servers or virtual instances, leasing them IP address along with other DHCP options configured in the system. It also provides a DNS service for the baremetal servers.

Multiple TSN nodes can be configured in the system based on the scaling needs of the cluster.

## Contrail TOR Agent

A TOR agent provisioned in the OpenContrail cluster acts as the OVSDB client for the TOR switch - all the OVSDB interactions with the TOR are done via the TOR agent. It programs the different OVSDB tables on the TOR switch and receives the local unicast table entries from it.

While TOR agents can be run on any node in the cluster, typical practice is to run it on the TSN node.

## Configuration Model

The following figure depicts the configuration model used in the system.

> 
        Physical Router
              |
              | 
        Physical Interface  -----  Logical Interface
                                          |
                                          |
                                   Virtual Machine Interface  -----  Virtual Network
> 

TOR Agent receives the configuration relevant to the TOR switch. It translates the OpenContrail configuration to OVSDB and populates the relevant OVSDB table entries in the TOR switch.  The following table maps the OpenContrail configuration objects to the OVSDB tables.

> 
        Contrail Objects                OVSDB Tables

        Physical Device	              Physical Switch
        Physical Interface	          Physical Port
        Virtual Networks	          Logical Switch
        Logical Interface	          <Vlan, Physical Port> binding to Logical Switch
        L2 Unicast Route table        Unicast Remote and Local Table
        -	                          Multicast Remote Table
        -	                          Multicast Local Table
        -	                          Physical Locator Table
        -	                          Physical Locator Set Table
> 

## Control Plane

The TOR Agent receives the EVPN entries (routes) for the virtual networks in which the TOR switch ports are members. These are added to the unicast remote table in the OVSDB. 

MAC addresses learnt in the TOR switch in different logical switches (entries from local table in OVSDB) are propagated to the TOR Agent. TOR Agent exports them to the control node in the corresponding EVPN tables, which are further distributed to other controllers and subsequently to compute nodes and other EVPN nodes in the cluster.

The TSN node receives the replication tree for each virtual network from the control node. It adds the required TOR addresses to the same, forming its complete replication tree. The other compute nodes also receive the replication tree from the control node and this tree includes the TSN node.

## Data Plane

VXLAN is the data plane encapsulation used. The Virtual tunnel endpoint (VTEP) for the baremetal end will be on the TOR switch.

Unicast traffic from baremetal servers are VXLAN encapsulated by the TOR switch and forwarded, if the destination MAC is known within the virtual switch.

Unicast traffic from the virtual instances in the OpenContrail cluster are forwarded to the TOR switch, where VXLAN is terminated and the packet is forwarded to the baremetal server.

Broadcast traffic from baremetal servers are received by the TSN node. It uses the replication tree to flood the broadcast packets in the virtual network.

Broadcast traffic from the virtual instances in the OpenContrail cluster are sent to the TSN node, which replicates the packets to the TORs.

## Configuration

Contrail WebUI can be used to configure a TOR switch and interfaces on the switch.

In "Configure - Physical Devices - Physical Routers" page, create an entry for the TOR switch, providing the TOR's IP address, VTEP address. The TSN and TOR agent addresses for this TOR should also be configured here.

In "Configure - Physical Devices - Interfaces" page, add the logical interfaces to be configured on the TOR. Please note that the Name of the logical interface should match the name on the TOR (eg, ge-0/0/0.10). Other logical interface configurations like Vlan id, Mac address and IP address of the baremetal server and the virtual network it belongs to can be configured here.

## Provisioning

TSN can be provisioned using Fab scripts. The following changes are required in the testbed.py:
1. In env.roledefs, hosts for 'tsn' and 'toragent' roles have to be added. The TSN node should also be configured in 'compute' role as well.
1. The TOR agent configuration should be added as below.
    `

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
    `
Use the tasks add_tsn and add_tor_agent to provision the TSN and TOR Agents.

## Prior Configuration required on QFX5100

The following configuration has to be done on a QFX5100 beforehand.

* Enable OVSDB
* Set the connection protocol 
* Indicate the interfaces that will be managed via OVSDB
* Configure switch options with VTEP source (in the example below, lo0.0 is used – ensure that this address is reachable from TSN node)

> 
* set interfaces lo0 unit 0 family inet address <router-id-reachable-on-ip-fabric>
* set switch-options ovsdb-managed
* set switch-options vtep-source-interface lo0.0
* set protocols ovsdb passive-connection protocol tcp port <port-number>
* set protocols ovsdb interfaces <interfaces-to-be-managed-by-ovsdb>
> 

## Debug

On the QFX, the following commands show the OVSDB configuration.

* 	show ovsdb logical-switch
* 	show ovsdb interface
* 	show ovsdb mac
* 	show vlans

Introspect on TOR agent and TSN nodes show the configuration and operational state of these modules.

The TSN module is like any other contrail-vrouter-agent on a compute node, with Introspect access available on port 8085 by default. Operational data like interfaces, VN and VRF information along with the routes can be checked here.

The port on which TOR agent Introspect access is available will be present in the config file provided to contrail-tor-agent. This provides the OVSDB data available through the client interface, apart from the other data available in a Contrail Agent.

## Changes in the Configuration file to the Agent

In /etc/contrail/contrail-vrouter-agent.conf file for TSN, Agent mode option is now additionally available in DEBUG section to configure the agent in TSN mode.

> 
agent_mode = tsn
> 

The following are the typical configuration items in a Tor Agent config file.

> 
[DEFAULT]
> 
agent_name = noded2-1    # Name (formed with hostname and TOR id from below)
>
agent_mode = tor         # Agent mode 
>
http_server_port=9010    # Port on which Introspect access is available
>
[DISCOVERY]
>
server=<ip>              # IP address of discovery server
>
[TOR]
>
tor_ip=<ip>              # IP address of the TOR to manage
>
tor_id=1                 # Identifier for ToR. TOR Agent subscribes 
                         # to ifmap-configuration with this name
>
tor_type=ovs             # ToR management scheme - only “ovs” is supported
>
tor_ovs_protocol=tcp     # IP-Transport protocol used to connect to TOR
>
tor_ovs_port=port        # OVS server port number on the ToR
>
tsn_ip=<ip>              # IP address of the TSN
> 

## Contrail REST API

* Physical Router

    {
              u'physical-router': {
    u'physical_router_management_ip': u'100.100.100.100',
    u'virtual_router_refs': [],
    u'fq_name': [
      u'default-global-system-config',
      u'test-router'
    ],
    u'name': u'test-router',
    u'physical_router_vendor_name': u'juniper',
    u'parent_type': u'global-system-config',
    u'virtual_network_refs': [],
    'id_perms': {
      u'enable': True,
      u'uuid': None,
      u'creator': None,
      u'created': 0,
      u'user_visible': True,
      u'last_modified': 0,
      u'permissions': {
        u'owner': u'cloud-admin',
        u'owner_access': 7,
        u'other_access': 7,
        u'group': u'cloud-admin-group',
        u'group_access': 7
      },
      u'description': None
    },
    u'bgp_router_refs': [],
    u'physical_router_user_credentials': {
      u'username': u'',
      u'password': u''
    },
    'display_name': u'test-router',
    u'physical_router_dataplane_ip': u'101.1.1.1'
             }
    }

* Physical Interface

    {
      u'physical-interface': {
    u'parent_type': u'physical-router',
    'id_perms': {
      u'enable': True,
      u'uuid': None,
      u'creator': None,
      u'created': 0,
      u'user_visible': True,
      u'last_modified': 0,
      u'permissions': {
        u'owner': u'cloud-admin',
        u'owner_access': 7,
        u'other_access': 7,
        u'group': u'cloud-admin-group',
        u'group_access': 7
      },
      u'description': None
    },
    u'fq_name': [
      u'default-global-system-config',
      u'test-router',
      u'ge-0/0/1'
    ],
    u'name': u'ge-0/0/1',
    'display_name': u'ge-0/0/1'
      }
    }

* Logical Interface

    {
      u'logical-interface': {
    u'fq_name': [
      u'default-global-system-config',
      u'test-router',
      u'ge-0/0/1',
      u'ge-0/0/1.0'
    ],
    u'parent_uuid': u'6608b8ef-9704-489d-8cbc-fed4fb5677ca',
    u'logical_interface_vlan_tag': 0,
    u'parent_type': u'physical-interface',
    u'virtual_machine_interface_refs': [
      {
u'to': [
          u'default-domain',
          u'demo',
          u'4a2edbb8-b69e-48ce-96e3-7226c57e5241'
]
      }
    ],
    'id_perms': {
      u'enable': True,
      u'uuid': None,
      u'creator': None,
      u'created': 0,
      u'user_visible': True,
      u'last_modified': 0,
      u'permissions': {
        u'owner': u'cloud-admin',
        u'owner_access': 7,
        u'other_access': 7,
        u'group': u'cloud-admin-group',
        u'group_access': 7
      },
      u'description': None
    },
    u'logical_interface_type': u'l2',
    'display_name': u'ge-0/0/1.0',
    u'name': u'ge-0/0/1.0'
      }
    }
