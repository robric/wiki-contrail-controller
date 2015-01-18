From R2.1, OpenContrail supports extending a cluster to include baremetal servers and other virtual instances connected to a TOR switch supporting OVSDB protocol. The baremetal servers and other virtual instances can be configured to be part of any of the virtual networks configured in the contrail cluster, facilitating communication between them and the virtual instances running in the cluster. Contrail policy configurations can be used to control this communication.

The solution is achieved by using OVSDB protocol to extend contrail control plane into the TOR switch. VXLAN encapsulation will be used to extend contrail data plane to the TOR switch, to communicate with the baremetal servers and other virtual instances.

# TOR Services Node (TSN)

A node provisioned in a new role, called TOR Services Node (or TSN), is introduced to support this. TSN will act as the multicast controller for the TOR switches and will also provide DHCP and DNS services to the baremetal servers or virtual instances running behind TOR ports.

As the multicast controller, TSN will receive the broadcast packets from the TOR, which are then replicated to the required compute nodes and other EVPN nexthops. Broadcast packets from the virtual machines in the cluster are sent from the respective compute nodes to the TOR switch directly.

 