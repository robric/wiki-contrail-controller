Vrouter takes the following parameters:

* vr_flow_entries (uint)    : maximum flow entries (default is 512K)
* vr_oflow_entries (uint)   : maximum overflow entries (default is 8K)
* vr_bridge_entries (uint)  : maximum bridge entries (default is 256K)
* vr_bridge_oentries (uint) : maximum bridge overflow entries
* vr_mpls_labels (uint)     : maximum MPLS labels used in the node (default is 5K)
* vr_nexthops (uint)        : maximum nexthops in the node
* vr_vrfs (uint)            : maximum VRFs supported in the node
* vrouter_dbg (int)         : 1 to dump packets, 0 to disable (disabled by default)

The currently configured limits can be seen using `vrouter --info`. These can be updated by editing /etc/modprobe.d/vrouter.conf with the desired options and by following it up with a reboot of the node.

`options vrouter vr_mpls_labels=196000 vr_nexthops=521000 vr_vrfs=65536 vr_bridge_entries=1000000`

The following could be used to derive the values:
* vrfs = Max number of VNs
* mpls_labels = (max number of VNs * 3) + 4000
* nexthops = (max number of VNs * 4) + (max number of interfaces on the node * 5) + number of TORs + number of compute nodes + 100
* bridge entries / macs = Maximum number of MACs in a VN
* Flow table is hashed based on flow tuple. When flow bucket is full, new entries are added to overflow table. It is recommended not to increase the overflow table size as it could impact performance.

Some of these parameters can be provisioned during setup via fab, using the following testbed configuration.

> env.vrouter_module_params = {
>
>     host4:{'mpls_labels':'196000', 'nexthops':'521000', 'vrfs':'65536', 'macs':'1000000'},
>
>     host5:{'mpls_labels':'196000', 'nexthops':'521000', 'vrfs':'65536', 'macs':'1000000'}
>
>}
