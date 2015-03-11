Contrail BGP implementation uses the following Extended Communities.

#### Route Target

A Two-Octet AS Specific route target extended community is generated for each routing-instance object.  The Global Administrator is set to the AS Number configured for the Contrail cluster.  The Local Administrator field is set to a unique number that is 8000000 (8 million) and higher.  This target is used as the export and import target for the routing-instance in question.

A Two-Octet AS Specific Route Target extended community is also generated for each logical-router object.  This route target is added as an import and export target to the default routing-instance of all virtual-networks that are connected to the logical-router.

The expectation is that the user can allocate and manage numbers smaller than 8000000 when they need to co-ordinate the allocation of targets between a Contrail cluster and DC GW Routers or between 2 or more Contrail clusters.

The Route Target extended community is defined in [RFC 4360](https://tools.ietf.org/html/rfc4360).

#### Encapsulation

This opaque extended community is used to indicate the list of encapsulation protocol that can be used for sending packets from an ingress router to an egress vRouter. The encapsulation extended community with one or more tunnel types is attached to routes advertised by a vRouter. It is assumed that all vRouters and DC GWs can send and receive MPLS in GRE. Other possible tunnel types are MPLS in UDP (can be used for both L2 and L3 routes) and VXLAN (only for L2 routes).

The Encapsulation extended community is defined in [RFC 5512](https://tools.ietf.org/html/rfc5512#page-9).

#### Security Group

A Two-Octet AS Specific route target extended community is generated for each security-group object.  The Global Administrator is set to the AS Number configured for the Contrail cluster.  The Local Administrator field is set to a unique number that is 8000000 (8 million) and higher. A security group extended community with one or more ids is attached to routes advertised by a vRouter as appropriate.  If a port/virtual-machine-interface has one or more security groups associated with it, this community is attached to all routes associated with the port.  This includes the routes for any addresses for the virtual-machine-interface as well as any interface-static-routes and allowed-address-pairs associated with the virtual-machine-interface.

The expectation is that the user can allocate and manage numbers smaller than 8000000 when they need to co-ordinate the allocation of security group ids between 2 or more Contrail clusters.

Advertising the list of security-group ids with routes in this manner allows for fully distributed enforcement of security-group rules.  There's no need for global re-compilation of IP based rules whenever a security-group is associated with a new port/virtual-machine-interface.

The Security Group extended community is currently not standardized and uses type/subtype 0x8004.

#### Origin VN

A Two-Octet AS Specific origin VN extended community is generated for each virtual-network object.  The Global Administrator is set to the AS Number configured for the Contrail cluster. The Local Administrator field is set to a per-VN unique number that is 1 and higher.

This extended community is used to convey the original virtual-network in which a route originated.  Note that the origin VN is preserved even when routes are re-originated with different route targets due to service chaining. The use of this extended community allows fully distributed policy enforcement for traffic between virtual-networks.

If a route is originated from a vRouter, it's straightforward to determine the origin VN for the route.  In case of routes received from a DC GW or from another Contrail cluster, the Control Nodes determine the origin VN based on the route targets in the route.

The Origin VN extended community is currently not standardized and uses type/subtype 0x8071.

#### MAC Mobility

This extended community is used to carry a sequence number for L2 and L3 routes originated by vRouters.  It helps determine the correct location of a MAC/IP when the virtual-machine moves from one vRouter to another.

The MAC Mobility extended community is defined in [RFC 7432](https://tools.ietf.org/html/rfc7432#page-18).