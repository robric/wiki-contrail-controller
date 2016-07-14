# Remote Instances

Extending virtual instances running in non Openstack clusters or extending baremetal servers into Contrail virtual networks can be achieved using [OVSDB protocol](https://github.com/Juniper/contrail-controller/wiki/Baremetal-Support) or by configuring a contrail compute node to run in gateway mode to support remote instances. This wiki summarizes the latter.

Traffic from each external virtual instance is tagged with a unique VLAN, which is then mapped to a virtual machine interface in the contrail cluster. A contrail compute node can be configured to map VLAN tagged traffic coming on a physical port (other than the cluster's underlay IP fabric port) to a virtual machine interface configured in the contrail cluster. The virtual machine interface corresponds to the interface of the remote instance. For the vrouter on the gateway compute node, it works similar to a local virtual machine interface, with the traffic being subjected to the similar forwarding decisions and policies.

## Provisioning

In /etc/contrail/contrail-vrouter-agent.conf, add the following in DEFAULT section and restart contrail-vrouter-agent.

`gateway_mode = remote-vm`

## Configuration

1. Configure a physical router with the hostname of the compute node that acts as the gateway.
2. Create a physical interface on the physical router, with the name of the interface on the compute node that will be used for this traffic.
3. Create a logical interface on the physical interface, with a unique VLAN id and type set to L2.
4. Create a virtual machine interface in the required virtual network along with the MAC address of the remote virtual instance and IP address and link it to the logical interface.

## External configuration

The traffic from the remote virtual instance should come with the required VLAN tag to the gateway port.

