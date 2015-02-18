## Introduction
OpenContrail Analytics exposes a rich variety of comprehensive and in-depth statistics related to the operational state of virtual machines or instances, virtual networks, floating IPs and other objects. Cloud operators using OpenContrail as the networking solution in an Openstack deployment benefit from having these statistics exposed to them. The OpenContrail Analytics API server provides a very fast and scalable REST API implementation to query these statistics. However for cloud operators who prefer to query the Ceilometer APIs to extract these statistics, OpenContrail provides a Ceilometer driver / plugin to expose these statistics. 

One of the important statistics for a cloud operator from an accounting, billing, and planning perspective is the traffic statistics for a particular floating IP or elastic IP assigned to a customer's virtual machine or instance. Further, traffic statistics for total traffic flowing from a customer's network to the public network are also needed.

## Proposal
Ceilometer currently does not have meters for traffic statistics for floating IPs and hence the proposal is to add the following meters to ceilometer -

    ip.floating.receive.bytes
    ip.floating.receive.packets
    ip.floating.transmit.bytes
    ip.floating.transmit.packets

Similarly, ceilometer currently does not meters for total traffic statistics from one virtual network to another (this is a generalization of the requirement for finding total traffic flowing from a customer's network to the public network). The proposal is to add the following meters to ceilometer -

    network.destination.network.receive.bytes
    network.destination.network.receive.packets
    network.destination.network.transmit.bytes
    network.destination.network.transmit.packets 

## Existing OpenContrail Ceilometer Driver Implementation
The OpenContrail ceilometer driver populates the `switch.port.receive.bytes`, `switch.port.receive.packets`, `switch.port.transmit.bytes`,  and `switch.port.transmit.packets` meters in ceilometer. 

The driver can be configured to populate these meters with the following options:

1. The floating IP statistics or the interface statistics

2. For a particular virtual machine / instance or all virtual machines / instances

3. For a particular virtual network or all virtual networks. 

The driver is configured using the `resource` field in the ceilometer pipeline configuration file - `pipeline.yaml`. For example, the below configuration in `pipeline.yaml` will configure the driver to populate the meters with the floating IP statistics for all virtual machines with destination virtual network `default-domain:demo:vn3-public`

    sources:
        - name: network_source
          interval: 600
          meters:
            - "switch.port.receive.packets"
            - "switch.port.transmit.packets"
            - "switch.port.receive.bytes"
            - "switch.port.transmit.bytes"
          resources:
              - opencontrail://a6s23.contrail.juniper.net:8081/analytics/uves/virtual-machine/?resource=fip_stats_list&virtual_network=default-domain:demo:vn3-public
          sinks:
            - oc_network
    sinks:
        - name: oc_network
          publishers:
            - rpc://
          transformers:

## Enhancements to OpenContrail Ceilometer Driver Implementation
The driver will be enhanced to support populating the above proposed traffic statistics meters for floating IPs and virtual networks. For these meters, only the address of the analytics API server is needed from the `resources` URL above and rest of the URL parameters like `resource` will be ignored. For the floating IPs meters, the driver will query neutron to obtain the list of floating IPs and extract the virtual machines/instances associated with the floating IPs. It will then query the OpenContrail analytics REST API server to extract the floating IP statistics associated with those virtual machines and floating IPs and populate the meters. Similarly for the virtual network meters, the driver will query neutron/nova to obtain the list of networks and then query the OpenContrail analytics REST API server to extract the inter and intra virtual network statistics and populate the meters. 