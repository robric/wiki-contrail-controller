Contrail BGP implementation uses the following Extended Communities.

### Route Target

A Two-Octet AS Specific Route Target extended community is generated for each routing-instance object.  The Global Administrator is set to the AS Number configured for the Contrail cluster.  The Local Administrator field is set to a unique number that is 8000000 (8 million) and higher.  This target is used as the export and import target for the routing-instance in question.

A Two-Octet AS Specific Route Target extended community is also generated for each logical-router object.  This route target is added as an import and export target to the default routing-instance of all virtual-networks that are connected to the logical-router.

The expectation is that the user can allocate and manage numbers smaller than 8000000 when they need to co-ordinate the allocation of targets between a Contrail cluster and DC GW Routers or between 2 or more Contrail clusters.

### Encapsulation


### Security Group
### Origin VN
### MAC Mobility