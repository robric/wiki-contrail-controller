# Introduction

Every virtual-network has a routing-instance associated with it. The routing-instance defines network connectivity for the virtual-machines spawned in the corresponding virtual-network. By default, the routing-instance contains routes only for the virtual-machines spawned in the virtual-network. Network policies allow network connectivity between two virtual-networks. 
 
Virtual-Networks do not have access to "public" network. "public" network here can be the IP Fabric or external networks across the IP Fabric. A Gateway must be used to provide connectivity to "public" network from a virtual-network. In traditional deployments, a routing device such as Juniper MX can act as a gateway.
 
Simple Gateway is a restricted implementation of gateway which can be used for experimental purposes. Simple gateway provides access to "public" network to a single virtual-network.

# Configuration

Simple Gateway uses following configuration parameters in the devstack "localrc" file. The routes given below in example are derived from the configuration parameters given below.
 
## CONTRAIL_VGW_PUBLIC_NETWORK
A routing-instance is internally created for every virtual-network defined. CONTRAIL_VGW_PUBLIC_NETWORK must specify the routing-instance created for virtual-network that needs "public" access. The routing-instance must be specified as fully-qualified name (FQN).
 
The FQN is derived from tenant, project and virtual-network according to format tenant:project:virtual-network:virtual-network.
 
Example: If tenant=default-domain, project=admin and virtual-network=net1, FQN for routing-instance created for virtual-network "net1" is default-domain:admin:net1:net1
 
## CONTRAIL_VGW_PUBLIC_SUBNET
Configuration variable CONTRAIL_VGW_PUBLIC_SUBNET specifies subnet of virtual-network (net1) needing "public" access. This subnet must be routable in the "public" network.
Example: 192.168.1.0/24
 
## CONTRAIL_VGW_INTERFACE
Simple-gateway creates a tap interface with this name. The tap interface is added in vrouter in context of routing-instance for the virtual-network. The interface carries packets between vrouter and host-os belonging to routing-instance CONTRAIL_VGW_PUBLIC_NETWORK.
 
Packets from virtual-network destined to "public" network are sent over this interface from vrouter. Also, packets from public network destined to virtual-network are sent from host-os to vrouter on this interface.
 
Example: vgw

# Example Scenario

Example configuration :
If "public" access is needed for a virtual-network with following attributes 
tenant = default-domain
project = admin
network-name = net1
subnet = 192.168.1.0/24
 
## Devstack Configuration

Add following lines in the "localrc" file for stack.sh
 
CONTRAIL_VGW_INTERFACE=vgw
CONTRAIL_VGW_PUBLIC_SUBNET=192.168.1.0/24
CONTRAIL_VGW_PUBLIC_NETWORK=default-domain:admin:net1:net1

## Vrouter and host-os configuration

The diagram below shows a summary of configuration.

    +-------------------------------------------------------------------+
    |                  Host-OS Networking Stack                         |
    |                  0.0.0.0/24 => 10.1.1.254                         |
    |                  192.168.1.253/24 => vgw                          |
    |                                              10.1.1.1/24          |
    +---------+----------------------------------------+----------------+
              |                                        |
              |vgw                                     |vhost0
              |                                        |
    +---------+----------------------------------------+----------------+
    |         |                                        |                |
    |+--------+-------------------+         +----------+-------------+  |
    ||NET1 Routing-Instance       |         | FABRIC Routing-Instance|  |
    ||                            |         |                        |  |
    ||0.0.0.0/0 => vgw            |         |192.168.1.0/24 => vhost0|  |
    ||192.168.1.253/32 => tap0    |         |10.1.1.1/32=>vhost0     |  |
    ||                            |         |10.1.1.253/32=>eth0     |  |
    |+-------+- ------------------+         +------------------------+  |
    |        |                                                          |
    |        |                    VROUTER                               |
    +--------+-----------------------------------------+----------------+
             |                                         |
             |tap0                                     |eth0
    +--------+------------+                            |
    |  192.168.1.253/24   |                             
    |                     |
    |   VM (VN=net1)      |
    +---------------------+
 
*NET1  is short form for default-project:admin:net1:net1

*FABRIC is the routing-instance for traffic on fabric-network

1. Virtual-network with tenant=default-domain, project=admin and virtual-network=net1 needs "public" access. FQN for routing-instance for the virtual-network is default-domain:admin:net1:net1. The virtual-network is configued with subnet 192.168.1.0/24
1. A virtual-machine is spawned in virtual-network net1 with IP address 192.168.1.253
1. Host-OS is configured with following,
   1. The routing-instance FABRIC represents the IP-Fabric.
   1. vhost0 interface is configured with IP 10.1.1.1/24. 
   1. I nterface vgw in host-os does not have any IP address.
   1. The host-os is n ot aware of any routing-instances.
   1. Routing is enabled in host-os to route packets between vhost0 and vgw
   1. Route is added for 192.168.1.0/24 pointing to vgw interface. Any packet destined to 192.168.1.0/24 from host-os are transmitted on this interface.
1. vrouter is configured with following,
   1. A routing-instance with name FABRIC is created for IP Fabric.
       1. Interface vhost0 is added to routing-instance FABRIC
       1. Interface eth0 is added to routing-instance FABRIC. The interface connected to IP Fabric.
       1. Default route is added pointing to interface vgw. This route will ensure packets destined to "public" network are sent to host-os
       1. The default route is exported in routing-instance NET1 to other compute nodes using BGP.
   1. A routing-instance for default-domain:admin:net1:net1 corresponding to virtual-network net1. The routing-instance is shown in abbrevated form as NET1 in diagram below.
      1. Interface vgw is added to routing-instance NET1
      1. Routing-instance NET1 has default-route pointing to interface tap0.
      1. Routing-instance NET1 contains routes to virtual-machines in the virtual-network

# Packet flow

## Packet from virtual-network (net1) to public network

1. Packet with source-ip=192.168.1.253 and destination-ip=10.1.1.253 from virtual-machine is received by vrouter on interface tap0
1. Interface tap0 is in routing-instance default-domain:admin:net1:net1
1. Route lookup for 10.1.1.253 in routing-instance default-domain:admin:net1:net1 hits default-route pointing to tap interface 'vgw'
1. vrouter transmits packet on 'vgw' and is received by networking-stack of host-os
1. Host-os does forwarding based on its routing table and forwards packet on vhost0 interface
1. Packets transmitted on vhost0 are received by vrouter
1. vhost0 interface is added in routing-instance FABRIC
1. Route for 10.1.1.253 in routing-instance FABRIC points to eth0 interface
1. vrouter transmits packet on eth0 interface
1. Host 10.1.1.253 on fabric receives the packet

## Packet from public network to virtual-network (net1)

1. Packet with source-ip=10.1.1.253 and destination-ip=192.168.1.253 from public network is received on interface eth0
1. vrouter has routing-instance FABRIC configured for fabric-network. Interface eth0 is added to routing-instance FABRIC
1. vrouter receives packet from eth0 in routing-instance FABRIC
1. Route lookup for 192.168.1.253 in FABRIC points to interface 'vhost0'
1. vrouter transmits packet on 'vhost0' and is received by networking-stack of host-os
1. Host-os does forwarding based on its routing table and forwards packet on 'vgw' interface
1. vrouter receives pacekt on 'vgw' interfcace into routing-instance default-domain:admin:net1:net1
1. Route lookup for 192.168.1.253 in routing-instance default-domain:admin:net1:net1 points to 'tap0' interface
1. vrouter transmits packet on tap0 interface
1. virtual-machine receives packet destined to 192.168.1.253

# Dynamic Virtual Gateway
From R1.1, Virtual Gateway can be created & deleted dynamically by sending thrift messages to the vrouter agent. The following thrift messages are defined:

    1. AddVirtualGateway to add a virtual gateway
    2. DeleteVirtualGateway to delete a virtual gateway
    3. ConnectForVirtualGateway can be used by stateful clients, which allows audit of virtual gateway configuration. Upon a new ConnectForVirtualGateway request, one minute is given for the configuration to be redone. Any older virtual gateway configuration remaining after this time is deleted.

## To create a virtual gateway
* Run the following script on the compute node where the virtual gateway will be created. This script enables forwarding on the node, creates the required interface, adds it to vrouter, adds required routes in the host OS and sends thrift message to the vrouter agent to create the virtual gateway.

    For example, to create interface vgw1 with subnets 20.30.40.0/24 and 30.40.50.0/24 in vrf default-domain:admin:vn1:vn1, run

    `python /opt/contrail/utils/provision_vgw_interface.py --oper create --interface vgw1 --subnets 20.30.40.0/24 30.40.50.0/24 --routes 8.8.8.0/24 9.9.9.0/24 --vrf default-domain:admin:vn1:vn1`

## To delete a virtual gateway
*  Run the following script on the compute node where the virtual gateway was created. This sends the DeleteVirtualGateway thrift message to the vrouter agent to delete the virtual gateway, deletes the interface from vrouter and deletes the routes added in the host OS.

    `python /opt/contrail/utils/provision_vgw_interface.py --oper delete --interface vgw1 --subnets 20.30.40.0/24 30.40.50.0/24`

If using a stateful client, send the ConnectForVirtualGateway thrift message to the vrouter agent when the client starts.

**Note:**
If the vrouter agent restarts or if  the compute node reboots, it is expected that the client will reconfigure again

# FAQ's

### Packets from external network are not reaching the compute node.

The devices in fabric network must be configured with static routes for 192.168.1.0/24 to reach the compute node running as simple gateway.
 
### Packets are reaching compute node, but are not routed from host-os to VM.

Check if the firewall_driver in /etc/nova/nova.conf file is set to nova.virt.libvirt.firewall.IptablesFirewallDriver. This will enable IPTables which can discard packet. Either disable IPTables during runtime or alternatively set firewall_driver to nova.virt.firewall.NoopFirewallDriver by setting LIBVIRT_FIREWALL_DRIVER=nova.virt.firewall.NoopFirewallDriver in localrc