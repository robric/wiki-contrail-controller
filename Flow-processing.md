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

  Flow-Key
  --------
  Flow key is made of <nh-id, src-ip, dst-ip, protocol, src-port, dst-port>
  nh-id: This is nexthop-id for the flow. The nexthop-id value is based on the
         type of packet
         1. Packet from vm-interface:
            The interface next-hop created for vm-interface is used as key
         2. MPLSoGRE or MPLSoUDP packet from fabric:
            The next-hop pointed to by the MPLS Label is used as key
         3. VxLan packet from fabric
            Bridge lookup is done for DMAC in the table pointed by vxlan-id.
            The nexthop in bridge route is used as the key
  src-ip, dst-ip, protocol, src-port and dst-port are the 5-tuple from packet

  Flow-Data
  ---------
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

Contrail does not use flow to make forwarding decisions in the flows. However,
flows can influence forwarding decision (NAT rewrites, VRF Translation and
choosing ECMP members)

Flows in contrail-vrouter-agent
-------------------------------
contrail-vrouter-agent is responsible to manage the flows in vrouter. Agent
applies policies rules and computes appropriate actions for the flows.

The diagram below gives summary of flow processing,


       +------------------+
       |                  |
       | Pkt Receive      |
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
                |   <-------------------------------------------------------------------------+
                |   |                                                                         ^
                |   |      <-------------------------------------------------------------+    |
                |   |      |                                                             ^    |
+---------------v---v------v----+                                                        |    |
|                               |                                                        |    |
|      +------------------+     |                                                        |    |
|      |                  |     |                                                        |    |
|      | Flow Setup       |     |                                                        |    |
|      |                  |     |                                                        |    |
|      +--------+---------+     |                                                        |    |
|               |               |                                                        |    |
|               |               |                                                        |    |
|               |               |                                                        |    |
|      +--------v---------+     |                                                        |    |
|      |                  |     |                                                        |    |
|      | Flow Table       +-----+----------------+-----------------------+               |    |
|      |                  |     |                |                       |               |    |
|      +--------+---------+     |                |                       |               |    |
|               |               |                |                       |               |    |
|               |               |                |                       |               |    |
|               |               |                |                       |               |    |
|      +--------v----------+    |      +---------v--------+     +--------v---------+     |    |
|      |                   |    |      |                  |     |                  |     |    |
|      | Index Management  |    |      |  Flow Management |     |  Flow Stats      |     |    |
|      |                   |    |      |                  |     |  Collector       |     |    |
|      +--------+----------+    |      +---------^---+----+     +--------+---------+     |    |
|               |               |                |   |                   |               |    |
|               |               |                |   |                   v-------------->+    |
|               |               |                |   v                                        |
|               |               |                |   +--------------------------------------->+
|      +--------v----------+    |      +---------+---+----+
|      |                   |    |      |                  |
|      | Flow KSync        |    |      |                  |
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

Agent receives flow setup notification from pkt0 interfaces. The packet handler
module registers pkt0 interface with ASIO to get notifications.

    1. Packet validation
        - Validate the incoming interface
        - Validate the incoming VRF
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




Flow setup event sequences
--------------------------

   VRouter                                Agent

   Allocate a HOLD entry
   Cache the packet
   Send message to agent for flow setup