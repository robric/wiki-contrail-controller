# Overview
Contrail uses flows to implement following features (as of R3.0)
  - Network Policy
  - Security Group
  - ECMP
  - RPF Check
  - Different type of NAT used for
  - Floating-IP
  - Link-local services
  - Metadata services
  - BGP As a Service

# Flows in VRouter

Flows in VRouter have following fields,

## Flow-Key
Flow key is made of <nh-id, src-ip, dst-ip, protocol, src-port, dst-port>
  - nh-id: This is nexthop-id for the flow. The nexthop-id value is based on the
         type of packet
         1. Packet from vm-interface:
            The interface next-hop created for vm-interface is used as key
         2. MPLSoGRE or MPLSoUDP packet from fabric:
            The next-hop pointed to by the MPLS Label is used as key
         3. VxLan packet from fabric
            Bridge lookup is done for DMAC in the table pointed by vxlan-id.
            The nexthop in bridge route is used as the key
  - src-ip, dst-ip, protocol, src-port and dst-port are the 5-tuple from packet

## Flow-Data

The data part of flow contains following information,
  - Flow Action
    Values can be Forward(F), Deny (D) or Nat

  - Flags

  - RPF Nexthop
    Used to implement RPF checks for the packet.
    When a flow is being setup, agent will do route lookup for the source-ip
    and sets up the rpf-nh in the flow. The type of NH used will depend on
    matched route for the source-ip,
    1. source-ip is reachable on a local vm-interface
       The rpf-nh points to nexthop created for the interface
    2. source-ip is reachable thru a remote compute node (non-ECMP route)
       The rpf-nh points to tunnel-nh for the remote compute node
       VRouter checks if the soruce-ip in tunnel-header meatches the IP address
       in the tunnel-nh
    3. source-ip is reachable via ECMP nexthop
       If source-ip is in ECMP, the rpf-nh will point to ECMP-NH. VRouter
       should accept packet as long as it comes from any of the member NH in
       ECMP

  - ECMP Component Index
    Specifies the member to be picked in case of ECMP

  - TCP Eviction flags
    Tracks the tcp-state flags. Used to evict TCP flows on connection closure

  - Statistics
    Packet and byte counts for the flow

  - Reverse-flow index
    Index of the reverse flow
    The reverse flow is used in two scenarios,
    - When a packet is to be NAT-ed, NAT address are got from the reverse flow
    - To track TCP connection from both directions

  - VRF
    Changes VRF for the packet to value configured

  - Gen-Id
    This is an 8-bit number representing generation-id for the flow. Every time a hash entry is reused for a flow, the gen-id is incremented.

Contrail does not use flow to make forwarding decisions in the flows. However,
flows can influence forwarding decision (NAT rewrites, VRF Translation and
choosing ECMP members)

## Flow debug output
A flow in vrouter is shown as below,

    Entries: Created 86315592 Added 86305583 Processed 86315016 Used Overflow entries 127
    (Created Flows/CPU: 2 2 0 0 3 0 0 1 0 6 0 0 4 0 0 0 0 0 0 34389347 51926223 0 0 0 4 0 0 0 0 0 0 0)(oflows 0)

    Action:F=Forward, D=Drop N=NAT(S=SNAT, D=DNAT, Ps=SPAT, Pd=DPAT, L=Link Local Port)
    Other:K(nh)=Key_Nexthop, S(nh)=RPF_Nexthop
    Flags:E=Evicted, Ec=Evict Candidate, N=New Flow, M=Modified
    TCP(r=reverse):S=SYN, F=FIN, R=RST, C=HalfClose, E=Established, D=Dead

    Index                Source:Port/Destination:Port                      Proto(V)
    -----------------------------------------------------------------------------------
    25<=>2811240      1.1.1.3:63                                         17 (1)
                      8.0.75.57:63
    (Gen: 243, K(nh):12, Action:F, Flags:, S(nh):12,  Stats:1/88,  SPort 58395)

The fields used are,

***_Entries: Created 86315592 Added 86305583 Processed 86315016 Used Overflow entries 127_***

This line give summary statistics for the flow.


***_(Created Flows/CPU: 2 2 0 0 3 0 0 1 0 6 0 0 4 0 0 0 0 0 0 34389347 51926223 0 0 0 4 0 0 0 0 0 0 0)(oflows 0)_***

This line give number of flow created on per-cpu basis and also number of overflow entries currently in use.


***Gen*** : Represents generation-id for the flow

***K(nh)*** : Represents nh-id for the flow

***Action*** : Action for the flow

***S(nh)*** : This is nexthop used for RPF checks

***Stats*** : Packet and Bytes counts for the flow. Updated when a packet hits the flow.

***SPort*** : Specifies the source-port used when packet is encapsulated in MPLSoUDP or VxLAN tunnels.

## Organization of Flow-Table
Flow table is organised as a hash-table with 4 entries per-bucket. Hash collision are resolved by allocating entries from an overflow table. The default size of hash-table is 512K entries (128K buckets) and overflow table is 8K entries. When a bucket has no empty slots, an entry from overflow table is allocated and linked to the bucket. The overflow entry is freed when flow is deleted.

The size of tables can be modified by setting the vrouter module parameters vr_flow_entries and vr_oflow_entries.

It is recommended that overflow-table should be atleast twice the number of expected flows and the overflow entries must be atleast 25% of the flow-table size

# AgentHdr format

All packets exchanged between agent and vrouter will have a proprietary header given below.
 
     0                        16                         31
     +--------------------------+--------------------------+
     |    ifindex               | vrf                      |
     +--------------------------+--------------------------+
     |    command               | prameter-1               |
     +--------------------------+--------------------------+
     |    parameter-1           | parameter-2              |
     +--------------------------+--------------------------+
     |    parameter-2           | parameter-3              |
     +--------------------------+--------------------------+
     |    parameter-3           | parameter-4              |
     +--------------------------+--------------------------+
     |    parameter-4           | parameter-5              |
     +--------------------------+--------------------------+
     |    parameter-5           | Unused                   |
     +--------------------------+--------------------------+

- ifindex : Interface index for the ingress interface. Can be fabric interface when packet is received on fabric or vm-interface.
- vrf : vrf-index for the packet
- parameter-1 : Flow-handle allocated by vrouter for the flow
- parameter-2 : 
- parameter-3 : 
- parameter-4 : 
- parameter-5 : Gen-Id for the flow

The packet receive notification is received in ASIO context. Agent does not do any processing in the ASIO context. All packet processing is done in "Packet Handler" module.

#Flows in Agent

contrail-vrouter-agent is responsible to manage the flows in vrouter. Agent
applies policies rules and computes appropriate actions for the flows.

The diagram below gives summary of flow processing,

               +------------------+
               |                  |
               |     pkt0 Rx      |
               |                  |
               +--------+---------+
                        |
                        |
               +--------v---------+
               |                  |
               | Pkt Handler      |
               |                  |
               +--------+---------+
                        | 1:N
                        |
                        |   <-----------------------------------------------------------------------+
                        |   |                                                                       ^
                        |   |      <-----------------------------------------------------------+    |
                        |   |      |                                                           ^    |
        +---------------v---v------v----+                                                      |    |
        |                               |                                                      |    |
        |      +------------------+     |                                                      |    |
        |      |                  |     |                                                      |    |
        |      | Flow Setup       |     |                                                      |    |
        |      |                  |     |                                                      |    |
        |      +--------+---------+     |                                                      |    |
        |               |               |                                                      |    |
        |               |               |                                                      |    |
        |               |               |                                                      |    |
        |      +--------v---------+     |                                                      |    |
        |      |                  |     |                                                      |    |
        |      | Flow Table       +-----+----------------+-----------------------+             |    |
        |      |                  |     |                |                       |             |    |
        |      +--------+---------+     |                |                       |             |    |
        |               |               |                |                       |             |    |
        |               |               |                |                       |             |    |
        |               |               |                |                       |             |    |
        |      +--------v----------+    |      +---------v--------+     +--------v---------+   |    |
        |      |                   |    |      |                  |     |                  |   |    |
        |      | Index Management  |    |      |  Flow Management |     |  Flow Stats      |   |    |
        |      |                   |    |      |                  |     |  Collector       |   |    |
        |      +--------+----------+    |      +---------^---+----+     +--------+---------+   |    |
        |               |               |                |   |                   |             |    |
        |               |               |                |   |                   v------------>+    |
        |               |               |                |   v                                      |
        |               |               |                |   +------------------------------------->+
        |      +--------v----------+    |      +---------+---+----+
        |      |                   |    |      |                  |
        |      | Flow KSync        |    |      | DB Clients       |
        |      |                   |    |      |                  |
        |      +--------+----------+    |      +------------------+
        +---------------|---------------+
                        |
               +--------v----------+
               |                   |
               | KSync Socket      |
               |                   |
               +-------------------+
                        |
                        |
                        |
                        |
               +--------v----------+
               |                   |
               | VRouter           |
               |                   |
               +-------------------+



### Pkt0 Rx
This module manages pkt0 interface. All packets exchanged over pkt0 interface will be prepended with a proprietary agent-header. The format of agent header is, 

Agent uses Boost ASIO library for I/O on the pkt0 interface. 
### Packet Handler

### Flow Handler

### Flow Table

### Index Management

### Flow KSync

### KSync Socket

### VRouter

### Flow Management

### DB Client

### Flow Stats Collector

Agent receives flow setup notification from pkt0 interfaces. The packet handler
module registers pkt0 interface with ASIO to get notifications.

    2. Packet parse
        - Decapsulate packet if dmac matches vhost-mac and destination-ip
          matches agent ip-address on fabric
          Decapsulate the pacekt if it has tunnel headers
        - If packet is TCP/UDP/SCTP packet
            Pick port-numbers from L4 header
        - If packet is ICMP echo request
            Set sport = ICMP Echo request indentifier
        - If packet is ICMP error
            Parse inner payload to get the 5 tuple




# Flow setup events