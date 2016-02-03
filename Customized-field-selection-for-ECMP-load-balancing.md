In Release 3.0, ability to configure the set of fields one can use to hash during ECMP load balancing has been introduced.
This set of fields until 3.0 was always fixed to the standard 5 tuple (Source L3 address, Destination L3 address, L4 protocol, L4 SourcePort and L4 DestinationPort) fields. With this new feature, users can configure the exact sub-set of fields to hash upon when choosing the forwarding path among a set of eligible ECMP candidates.

The schema change for this feature can be found in [Hash Fields Schema](https://github.com/Juniper/contrail-controller/blob/master/src/schema/xmpp_unicast.xsd#L69)

Configuration can be applied on a global level, on a per virtual-network(VN) level and on a per virtual-network-interface(VMI) level. VMI's config takes precedence over VN config which takes precedence over global level configuration if present.

Typically this feature is useful whenever packets originated from a particular source and addressed a particular destination MUST go through a same set of service instances during the transit. This could be required if source/destination/transit nodes keep some state based on the flow and could be used for subsequent new flows as well, between the same pair of source and destination addresses.

Instead of providing a feature to select pre-configured subsets of the 5 tuples to use during ecmp forwarding, a general feature to select necessary fields directly by the users themselves has been provided in 3.0 release.

e.g. ECMP fields can be selected from contrail web UI at virtual-network level as shown in this picture ![ECMP Fields Selection configuration](https://raw.githubusercontent.com/wiki/rombie/contrail-controller/virtual_network_ecmp_fields_selection.png)


