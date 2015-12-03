When a virtual machine sends or receives IP traffic, forward and reverse flow entries are setup. When the first packet arrives, flow key, based on the 5-tuple consisting of source and destination IP addresses, ports and IP protocol is used to hash into a flow table (identify a hash bucket). A flow entry is created and the packet is sent to contrail-vrouter-agent, where configured policies are applied and flow action is updated. The agent also creates a flow entry for the reverse direction, where relevant. Subsequent packets will hit the established flow entries and are forwarded / dropped / NATed etc based on the flow action.

When the hash bucket is full, entries are created in an overflow table. In earlier releases, the overflow table was a global table, which will be searched sequentially. From R2.22, the overflow entries are maintained as a list against the hash bucket.

The default number of flow table and overflow table entries are 512K and 8K respectively. These can be modified by configuring them as vrouter module parameters – vr_flow_entries and vr_oflow_entries (see https://github.com/Juniper/contrail-controller/wiki/Vrouter-Module-Parameters).

## Flow aging
Flows are aged out based on inactivity for a specific period of time. By default, the timeout value is 180 seconds. This can be modified by configuring flow_cache_timeout under the DEFAULT section in contrail-vrouter-agent.conf file.

## TCP State based flow handling
From R2.22, vrouter handles flow setup and tear down for TCP connections based on the TCP state machine. Vrouter maintains the TCP state on the flows, based on the SYN and FIN packets. When connection close is identified, the flow is evicted.

## Protocol based flow aging
While TCP flows are now deleted based on TCP state, it is sometimes required to age out specific protocol flows more aggressively. One example could be when a DNS server is run in one VM, a number of flows would be set up for DNS, a pair of flows to serve each query. As the flows are no longer required once the query is served, timeout could be lower for these flows. To handle these cases, protocol based flow aging is supported from R2.22, where in the aging timeout can be configured per protocol. All other protocols continue to use the default aging timeout.  

This configuration can be done in global-vrouter-config. For example, `protocol = udp, port = 53 will be set an aging timeout of 5 seconds`, to have all DNS flows to be aged with that timeout.
