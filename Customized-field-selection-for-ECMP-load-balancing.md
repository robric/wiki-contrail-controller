In Release 3.0, ability to configure the set of fields one can use to hash during ECMP load balancing has been introduced.
This set of fields until 3.0 was always fixed to the standard 5 tuple (Source L3 address, Destination L3 address, L4 protocol, L4 SourcePort and L4 DestinationPort) fields. With this new feature, users can configure the exact sub-set of fields to hash upon when choosing the forwarding path among a set of eligible ECMP candidates.

The schema change for this feature can be found in [Hash Fields Schema](https://github.com/Juniper/contrail-controller/blob/master/src/schema/xmpp_unicast.xsd#L69)

Configuration can be applied on a global level, on a per virtual-network(VN) level and on a per virtual-network-interface(VMI) level. VMI's config takes precedence over VN config which takes precedence over global level configuration if present.

Typically this feature is useful whenever packets originated from a particular source and addressed a particular destination MUST go through a same set of service instances during the transit. This could be required if source/destination/transit nodes keep some state based on the flow and could be used for subsequent new flows as well, between the same pair of source and destination addresses. In such cases, subsequent flows must follow the same set of service nodes that the initial flow followed.

Instead of providing a feature to select pre-configured subsets of the 5 tuples to use during ECMP forwarding, a general feature to select necessary fields directly by the users themselves has been provided in 3.0 release.

e.g. ECMP fields can be selected from contrail web UI in virtual-network configuration section as shown in this picture ! 
[ECMP Fields Selection under VN](https://raw.githubusercontent.com/wiki/rombie/contrail-controller/virtual_network_ecmp_fields_selection.png)

If configured for the VirtualNetwork, all traffic destined to that VM will get the customized hash field selection during forwarding over ECMP paths (by VRouters).

Instead, in a more practical scenario in which, flows between a pair of source and destination must go through the same service-instance in between, one could configure customized ECMP fields for the ServiceInstances' VirtualMachineInterface (VMI). Then, all service-chain routes originated off that VMI would get the desired ECMP field selection applied as its path attribute, and eventually gets propagated to the ingress VRouter node.

! [ECMP Fields Selection under VMI](https://raw.githubusercontent.com/wiki/rombie/contrail-controller/virtual_network_interface_ecmp_fields_selection.png)


### Applicability
This feature is mainly applicable in scenarios where in multiple ECMP paths exist for a destination. Typically these paths point ingress service instance nodes, which could be running any where in the contrail cloud. Following steps can be followed to create a setup with ECMP over service chains

1. Create necessary virtual networks to interconnect using service-chaining (with ECMP load-balancing).
2. Create a service template ST and enable scaling.
3. Create service instance SI using ST, select desired number if instances for scale-out, select desired left and right virtual network to connect and also select "shared address" space to make sure that instantiated services come up with the same IP address for left and right respectively. This enables ECMP among all those service instances during forwarding
4. Create a policy, select service instance SI and apply it to the desired VMIs or VNs.
5. After service VMs are instantiated, left and right interfaces' ports shall be available for further configuration. Select the left port (VMI) of the service instances and apply appropriate ECMP field selection configuration.

Note: At the moment, from the contrail UI, one cannot apply ECMP field selection configuration directly over the service's left or right interface. Instead, one has go to the ports(VMIs) section under networking and configure ECMP fields selection for each of the instantiated service instances' VMIs explicitly. This must be done for interface of all service instances in the group to ensure correctness.

Once all setup done correctly, vrouters shall be programmed with appropriate routing table with ECMP paths towards various service instances. Also vrouters are programmed with the desired ECMP fields to be used to hash during load balancing the traffic.

