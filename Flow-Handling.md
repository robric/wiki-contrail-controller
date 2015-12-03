When a virtual machine sends or receives IP traffic, forward and reverse flow entries are setup. When the first packet arrives, a flow key is used to hash into a flow table (identify a hash bucket). The flow key is based on five tuple consisting of source and destination IP addresses, ports and the IP protocol. A flow entry is created and the packet is sent to contrail-vrouter-agent. Agent applies the configured policies and updates the flow action. Agent also creates a flow entry for the reverse direction, where relevant. Subsequent packets match the established flow entries and are forwarded / dropped / NAT translated and so on based on the flow action.

When the hash bucket is full, entries are created in an overflow table. In releases earlier than Contrail Release 2.22, the overflow table was a global table, which is searched sequentially. In Contrail Release R2.22 and later, the overflow entries are maintained as a list against the hash bucket.

By default, the maximum number of flow table and overflow table entries are 512,000 and 8,000 respectively. These can be modified by configuring them as vrouter module parameters – vr_flow_entries and vr_oflow_entries (For more information about the vRouter module parameters, see https://github.com/Juniper/contrail-controller/wiki/Vrouter-Module-Parameters).

## Flow aging
Flows are aged out based on inactivity for a specific period of time. By default, the timeout value is 180 seconds. This can be modified by configuring flow_cache_timeout under the DEFAULT section in /etc/contrail/contrail-vrouter-agent.conf file.

## TCP State based flow handling
In Contrail Release 2.22 and later, Contrail vRouter monitors TCP flows to identify entries that can be reused without going through the standard aging cycle. All flow entries that correspond to TCP flows that have seen a connection tear down, be it through the standard TCP closure cycle (FIN/ACK-FIN/ACK) or the RST indicator, is torn down by vRouter and is available for use by other new qualified flows that needs new entries immediately. vRouter also keeps track of connection establishment cycle and exports necessary information to vRouter-agent (such as SYN/ACK and a digested Established flag) so that vRouter-agent can tear down flows that don't see a full connection cycle.

Flows that vRouter identifies as reuse candidates, more precisely 'Eviction Candidates', are marked so in the flow entry. Such flows will get to 'Evicted' state eventually when they become available for other new flows to be reused. Such a two step transition is followed so that the flow entry remains valid through the time the packet reaches the destination and not get remapped to another flow entry in the interim.


## Protocol based flow aging
While TCP flows are now deleted based on TCP state, it is sometimes required to age out specific protocol flows more aggressively. One example is when a DNS server is run in one VM. In this case multiple flows would are set up for DNS, a pair of flows to serve each query. Because the flows are no longer required once the query is served, the timeout could be lower for these flows. To handle these cases, protocol-based flow aging is used. With protocol-based flow aging, the aging timeout can be configured per protocol. All other protocols continue to use the default aging timeout.  

This configuration can be done in global-vrouter-config. For example, `protocol = udp, port = 53 can be set to an aging timeout of 5 seconds`, to have all DNS flows to be aged with that timeout.
