Add support for ecmp based loadbalancer in OpenContrail.

Currently OpenContrail supports L4-L7 proxy based loadbalancers. This is done by instantiating haproxy on a compute node. If there is no requirement for proxy based loadbalancers then ecmp based loadbalancing helps prevents traffic tromboning.

#Summary

OpenContrail supports ECMP based forwarding if the same IP address is announced by different endpoints. This is based on a hash of the five tuple:
* Source ip
* Source port
* Destination ip
* Desination port
* Protocol

Native loadbalancer will use the ECMP feature to support non-proxy based loadbalancing.

#API changes
New loadbalancer provider called "native" is being added to the loadbalancer object. The rest of the objects remain the same as before in the loadbalancer hierarchy.

* User creates a port for a virtual-ip.
* User provides the list of endpoint IPs where the VIP should be loadbalanced.

#Controller changes
Controller now aggregates the configuration based on the provider. In the case of native provider controller allocates a new floating-ip object as the child of the instance-ip. It copies the IP address from the instance-ip into the floating-ip. Controller also copies the port translations from VIP to members in the floating-ip object. There is a flag to identify that port-translations are enabled on the floating-ip.

In addition controller also attaches this floating IP to all the virtual machine interface objects of the endpoints/pool members.

#Agent changes
Agent acts on the new floating-ip object. Based on the port translation flag agent creates a port translation table for the floating ip. In addition agent traverses its parent instance-ip to get the corresponding routing table information.

#Example LB creation
- Set the configuration for provider as "native"

    neutron net-create private-net
    neutron subnet-create --name private-subnet private-net 30.30.30.0/24
    neutron lbaas-loadbalancer-create $(neutron subnet-list | awk '/ private-subnet / {print $2}') --name lb1
    neutron lbaas-listener-create --loadbalancer lb1 --protocol-port 80 --protocol HTTP --name listener1
    neutron lbaas-pool-create --name pool1 --protocol HTTP --listener listener1 --lb-algorithm ROUND_ROBIN
    neutron lbaas-member-create --subnet private-subnet --address 30.30.30.10 --protocol-port 8080 mypool
    neutron lbaas-member-create --subnet private-subnet --address 30.30.30.11 --protocol-port 8080 mypool

#Health check
For health check we will use the following blueprint.
https://blueprints.launchpad.net/opencontrail/+spec/service-health-check