
# Overview
QoS in networking guarantees reliability, bandwidth, latency and better management of network traffic upon congestion. Network traffic gets marked with QoS bits (DSCP, 802.1p etc) which intermediate network switches and routes use to provide priority.

QoS feature in contrail would be supported from 3.1

# Implementation details

## Configuration objects.
1. Forwarding-class: specifies knobs for marking and queuing. 
    1. If packet is IP packet, then it specifes DSCP value to be written on packet  
    2. If packet is non-IP layer-2 packet, then it specifies 802.1p value to be written on packet

2. Queue: Specifies priority of the traffic, in future bandwidth properties can be set on this entity.

3. QoS config object: specifies a mapping from DSCP, 802.1p and MPLS EXP values to corresponding forwarding class. It also has a trusted mode to specify if QoS bits in packet should be honored or not.
    1. If packet is IP packet then DSCP map would be used to lookup and corresponding forwarding class will be applied.
    2. If packet is layer2 packet then 802.1p map would be used to lookup and corresponding forwarding class will be applied.
    3. If its a MPLS tunneled packet and its has MPLS EXP values specified, then EXP bit value would be used to lookup into MPLS EXP map and corresponding forwarding class will be applied.    

>

            +--------------------------+       +---------------------+       +----------+ 
            +     QOS config object    + ----> +  Forwarding class   + ----> +  Queue   + 
            +--------------------------+       +---------------------+       + ---------+

>
            
                  +------------------------------------------------------------+
                  +   +--------------+    +--------------+   +--------------+  + 
                  +   + DSCP1 - FC-X +    + .1p 1 -- FC1 +   + EXP1 -- FC-Y +  +                                            
                  +   + DSCP2 - FC-Y +    + .1p 2 -- FC2 +   + EXP2 -- FC-Z +  + 
                  +   +   ....       +    + .....        +   + .....        +  +
                  +   +--------------+    +--------------+   +--------------+  +
                  +                    QOS config                              +
                  +------------------------------------------------------------+
                                           
Virtual-machine-interface, virtual-network, network policy and security-group can refer to QoS config object.
QOS config object can be specified on vhost and fabric interface so that underlay traffic can also be subjected to marking and queing  

>
            
                        +------------------+    +------------+      +-------------+  
                        + Virtual machine  +    +  Virtual   +      +  Policy/SG  + 
                        + interface        +    +  network   +      +             +  
                        +------------------+    +------------+      +-------------+
                                |                     |                    |
                                |                     V                    | 
                                |            +--------------------+        |          +---------------+
                                ---------->  +    QoS config      + <-----------------+  global qos   +   
                                             +--------------------+      vhost/       +    config     +
                                                                         fabric       +---------------+    
## Example Forwarding class

Name    |  ID | DSCP | 802.1p | MPLS EXP|Queue |
--------|-----|------|--------|---------|------| 
FC1     |  1  |  10  |   7    |   7     |  0   | 
FC2     |  2  |  38  |   0    |   0     |  1   |

In the above table there are two forwarding class objects defined.
FC1 marks the traffic with high priority values and queues to Queue 0.
FC2 marks the traffic a best effort and queues the traffic to Queue 1.

## Example QOS config object

DSCP | Forwarding-class ID | 802.1p  | Forwarding-class ID | MPLS EXP | Forwarding-class ID |
-----|---------------------|---------|---------------------|----------|---------------------|
  10 |        1            |   6     |           1         |    5     |        1            |
  18 |        1            |   7     |           1         |    7     |        1            |   
  26 |        1            |   *     |           2         |    *     |        1            |
   * |        2            |         |                     |          |                     |         

In the above example QOS config object DSCP values 10, 18 and 26 are mapped to forwarding class with ID 1 which is *FC1*,
all the other IP packets are mapped to forwarding class with ID 2 which is *FC2*.
Similarly all traffic with 802.1p value of 6 and 7 are mapped to forwarding class *FC1*, rest to *FC2*.

##QoS config object marking on packet
### Traffic originated by VMI
1. If interface sends a IP packet with DSCP value to another VM in remote compute node, then this DSCP value would be used to look up in cos-config table, and the tunnel header would be marked with DSCP, 802.1p and MPLS EXP bit as specified by forwarding-class.
2. If VM sends a layer 2 non IP packet with 802.1p value, then corresponding 802.1p value would be used to look into qos-config table and corresponding forwarding-class DSCP, 802.1p and MPLS EXP value would be written to tunnel header.
3. If VM sends a packet a IP packet with DSCP value to VM in same compute node, then DSCP value assigned forwarding class would be used to overwrite IP header with new DSCP value and 802.1p value.

###Traffic destined to VM
1. If a tunneled packet is received DSCP value in the tunnel IP header would be used to look in qos-config table and corresponding DSCP value would be written to inner payload IP header.

### Traffic from vhost interface
QoS config can be applied on IP traffic coming from vhost interface. DSCP value in packet would be used to lookup into cos-config object specified on vhost, and corresponding forwarding-class specified DSCP and 802.1p value would be rewritten on packet.

### Traffic from fabric interface
QoS config can be applied while receiving packet on ethernet interface of compute node. Based on DSCP or 802.1p value in packet corresponding forwarding-class DSCP, 802.1p values would be rewritten in packet header.

### Precedence of QoS bits in packet
1. IP packet would use DSCP value to lookup in DSCP table of qos-config.
2. Non IP layer 2 traffic would use 802.1p value and lookup in 802.1p table of cos-config.

### QoS config assignment on packet
QoS config can be specified at multiple levels, following is the order of priority
   1. QoS config in policy
   2. QoS config specified on virtual-network
   3. QoS config specified on virtual-machine-interface

# Caveats
1. NIC queues would be used.
1. Built in or default scheduler in NIC would be used.        