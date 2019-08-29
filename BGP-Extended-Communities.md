Contrail BGP implementation uses the following Extended Communities.

#### Route Target

A Two-Octet AS Specific route target extended community is generated for each routing-instance object.  The Global Administrator is set to the AS Number configured for the Contrail cluster.  The Local Administrator field is set to a unique number that is 8000000 (8 million) and higher.  This target is used as the export and import target for the routing-instance in question.

A Two-Octet AS Specific Route Target extended community is also generated for each logical-router object.  This route target is added as an import and export target to the default routing-instance of all virtual-networks that are connected to the logical-router.

The expectation is that the user can allocate and manage numbers smaller than 8000000 when they need to co-ordinate the allocation of targets between a Contrail cluster and DC GW Routers or between 2 or more Contrail clusters. The user can configure one or more such manually allocated route targets for a virtual-network.

The Route Target extended community is defined in [RFC 4360](https://tools.ietf.org/html/rfc4360).

#### Encapsulation

This opaque extended community is used to indicate the list of encapsulation protocols that can be used to send packets from an ingress router to an egress vRouter. The encapsulation extended community with one or more tunnel types is attached to routes advertised by a vRouter. It is assumed that all vRouters and DC GWs can send and receive MPLS in GRE. Other possible tunnel types are MPLS-in-UDP (can be used for both L2 and L3 routes) and VXLAN (only for L2 routes).

The Encapsulation extended community is defined in [RFC 5512](https://tools.ietf.org/html/rfc5512#page-9). The [IANA registry for tunnel types](http://www.iana.org/assignments/bgp-parameters/bgp-parameters.xhtml#tunnel-types) contains the current list of defined types. Tunnel types supported by Contrail are GRE (2), MPLS-in-UDP (13) and VXLAN (8). Additionally, the value 37001 (0x9089) is used as a pre-standard code point for MPLS-in-UDP.

#### Security Group

A Two-Octet AS Specific route target extended community is generated for each security-group object.  The Global Administrator is set to the AS Number configured for the Contrail cluster.  The Local Administrator field is set to a unique number that is 8000000 (8 million) and higher. A security group extended community with one or more ids is attached to routes advertised by a vRouter as appropriate.  If a port/virtual-machine-interface has one or more security groups associated with it, this community is attached to all routes associated with the port.  This includes the routes for any addresses for the virtual-machine-interface as well as any interface-static-routes and allowed-address-pairs associated with the virtual-machine-interface.

The expectation is that the user can allocate and manage numbers smaller than 8000000 when they need to co-ordinate the allocation of security group ids between 2 or more Contrail clusters.

Advertising the list of security-group ids with routes in this manner allows for fully distributed enforcement of security-group rules.  There's no need for global re-compilation of IP based rules whenever a security-group is associated with a new port/virtual-machine-interface.

The Security Group extended community is currently not standardized and uses type/subtype **0x8004**.

#### Origin VN

A Two-Octet AS Specific origin VN extended community is generated for each virtual-network object.  The Global Administrator is set to the AS Number configured for the Contrail cluster. The Local Administrator field is set to a per-VN unique number that is 1 and higher.

This extended community is used to convey the original virtual-network in which a route originated.  Note that the origin VN is preserved even when routes are re-originated with different route targets due to service chaining. The use of this extended community allows fully distributed policy enforcement for traffic between virtual-networks.

If a route is originated from a vRouter, it's straightforward to determine the origin VN for the route.  In case of routes received from a DC GW or from another Contrail cluster, the Control Nodes determine the origin VN based on the route targets in the route.

The Origin VN extended community is currently not standardized and uses type/subtype **0x8071**.

#### MAC Mobility

This extended community is used to carry a sequence number for L2 and L3 routes originated by vRouters.  It helps determine the correct location of a MAC/IP when the virtual-machine moves from one vRouter to another.  Note that the EVPN specification describes the use of this extended community for L2 routes, but Contrail also uses it for L3 routes.

The MAC Mobility extended community is defined in [RFC 7432](https://tools.ietf.org/html/rfc7432#page-18).

#### Load Balance

This extended community is used to carry a list of packet fields to be used in the data plane while computing the hash during forwarding (over ECMP). It allows the originator (or re-originator) of a route to control the load balancing behavior at the traffic source(s).  By default, hashing is always based on the standard 5-tuple fields from the packet header (Src IP, Dest IP, L4 Protocol, Src Port and Dest Port). For further information, please refer to [format of this extended community] (https://github.com/Juniper/contrail-controller/blob/master/src/bgp/extended-community/load_balance.h#L24) and [Applicability](https://github.com/Juniper/contrail-controller/wiki/Customized-field-selection-for-ECMP-load-balancing). IETF Draft to standardize this has not been submitted yet.

#### Tag

This extended community is used to carry a opaque 4 bytes tag value with the routes. Format used to display is "tag:<4-byte-number>". Extended community type used to encode this is "Experimental (0x80)" with sub-type value "Tag (0x84)". One could use in firewall policies to match on the tags and perform various actions, and we would be using in various other contrail functions later.  Please refer to [fw_security_enhancements](https://github.com/Juniper/contrail-controller/blob/master/specs/fw_security_enhancements.md) for requirement specifications. Please refer to [tag.cc](https://github.com/Juniper/contrail-controller/blob/master/src/bgp/extended-community/tag.cc) for implementation details.

#### Origin Sub Cluster

This extended community is used to enforce match actions in routing policies (implicit or explicit). This permits to check wether routes are originated in a local site or a remote site, the semantic of "local" vs "remote site" being defined in conformance with the remote compute architecture. In the remote compute architecture, a unique contrail cluster spans multiple locations interconnected via an MPLS Backbone and site-specific DC gateways. Using Contrail terminology, a cluster is split into one or more sub-cluster that generally map to sites or pods.

In this regard, the allocation of the Origin Sub Cluster community maps with the definition of the sub-cluster ID object defined for this purpose. For further information, please refer to the [contrail sub-cluster specs](https://github.com/Juniper/contrail-specs/blob/master/subcluster.md). Hence, all routes orginiated in a given sub-cluster (i.e. site) will be tagged with a same Origin Sub Cluster community that is unique to this sub-cluster. Hence, as objects such as Virtual Networks or Routing Policies have a cluster-wide semantic (i.e. applying to any location), the Origin Sub Cluster community enhances the granularity of match/actions with a site-specific (i.e. sub-cluster) information. 

One direct application of this community is the extension of [Virtual Network Import Policies](https://github.com/Juniper/contrail-specs/blob/master/5.0/enhanced_routing_policies.md) to match all prefixes advertised by the local DC Gateways to the local Control Nodes. In this case, it is required to match only external prefixes and not local prefixes. This is achieved by enforcing a default tagging on local routes (XMPP, servicechain, aggregate, etc.) with the Origin Sub Cluster in order to exclude these routes from being matched by the routing policy. Note that this approach is compliant with HA: for example a local XMPP route redistributed to MP-BGP from Control Node to Control Node synchronisation is not processed by the routing policy. Additionally, this approach is compliant with the remote compute architecture because the value of the Origin Sub Cluster will differ for any sites so routes from a remote site (learnt via the local DC gateway) will be processed by the routing policy, which is expected.

The type/subtype for this community is TBD (IANA). The structure of this extended community is "ASN:Identifier", where Identifier is a 2-Byte field to comply with 4-byte AS length.
