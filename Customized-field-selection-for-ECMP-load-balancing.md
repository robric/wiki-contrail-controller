In Release 3.0, ability to configure the set of fields one can use to hash during ECMP load balancing has been introduced.
This set of fields until 3.0 was always fixed to the standard 5 tuple (Source L3 address, Destination L3 address, L4 protocol, L4 SourcePort and L4 DestinationPort) fields. With this new feature, users can configure the exact sub-set of fields to hash upon when choosing the forwarding path among a set of eligible ECMP candidates.

The schema change for this feature can be found in [Hash Fields Schema](https://github.com/Juniper/contrail-controller/blob/master/src/schema/xmpp_unicast.xsd#L69)

Configuration can be applied on a global level, on a per virtual-network(VN) level and on a per virtual-network-interface(VMI) level. VMI's config takes precedence over VN config which takes precedence over global level configuration if present.



