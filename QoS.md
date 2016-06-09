
# Overview

QoS feature in networking provides the ability to control reliability, bandwidth and latency among other traffic management features. Network traffic can be marked with QoS bits (DSCP, 802.1p etc), which intermediate network switches and routers can use to provide service guarantees.

#High level QOS model in Contrail

1. All packet forwarding devices (for e.g.: vRouter and Gateway) together form a system.
2. Interfaces to the system are ports from where system sends and receives packets, such as tap interfaces and physical ports.
3. Fabric interfaces are interfaces where the overlay traffic is tunneled
4. QOS is applied at ingress to the system, i.e traffic from interfaces to fabric.
5. At egress, packets will be stripped off their tunnel headers and sent to interface queues based on forwarding class. No marking from the outer packet to innerpacket is considered at this time.

Unlike other interfaces that may not be shared, fabric interfaces are always shared, and thus, a common property. Hence, the traffic classes and the QOS marking on the fabric has to be controlled by the System Administrator. Administrator may provision different classes of service on the fabric.

This is achieved by two constructs:

* Queueing on the fabric interface, which involves queues, scheduling of queues and drop policy
  AND
* Forwarding class, which involves marking and identifying which queue to use and thus controls how packets are sent to fabric.

Tenants have the ability to define which forwarding class their traffic can use. So, in QOS config, they can decide which packets can use what forwarding class.

QOS config object has a mapping table from incoming DSCP or .1p to forwarding class mapping.

Additionally QOS config can be applied to a virtual-network, interface or network-policy.

#Release
1. QOS config and forwarding class will be implemented as part of 3.1
2. Queueing will be implemented as part 3.2
3. Egress marking and queueing is currently not planned.

# Implementation details

## Configuration objects.
1. Forwarding-class: specifies knobs for marking and queuing. 
    1. Specifies DSCP value, 802.1p and MPLS EXP values to be written on packet  
    2. Specifies queue index to be used for packet.
 
2. QoS config object: specifies a mapping from DSCP, 802.1p and MPLS EXP values to corresponding forwarding class. It also has a trusted mode to specify if QoS bits in packet should be honored or not.
    1. If packet is IP packet then DSCP map would be used to lookup and corresponding forwarding class will be applied.
    2. If packet is layer2 packet then 802.1p map would be used to lookup and corresponding forwarding class will be applied.
    3. If its a MPLS tunneled packet and its has MPLS EXP values specified, then EXP bit value would be used to lookup into MPLS EXP map and corresponding forwarding class will be applied.
    4. If QoS config is untrusted then DSCP, 802.1p and EXP bits in packet would be ignored, all packets would be marked with same forwarding-class properties.

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
                            |            +--------------------+ <------|     +---------------+
                            ---------->  +    QoS config      + <------------+  global qos   +   
                                         +--------------------+     vhost/   +    config     +
                                                                    fabric   +---------------+    
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
### Traffic originated by Virtual machine interface
1. If interface sends a IP packet to another VM in remote compute node, then this DSCP value in IP header value would be used to look up in cos-config table, and the tunnel header would be marked with DSCP, 802.1p and MPLS EXP bit as specified by forwarding-class.
2. If VM sends a layer 2 non IP packet with 802.1p value, then corresponding 802.1p value would be used to look into qos-config table and corresponding forwarding-class DSCP, 802.1p and MPLS EXP value would be written to tunnel header.
3. If VM sends an IP packet to VM in same compute node, then DSCP value in IP header would be matched in qos-config and corresponding forwarding class would be used to overwrite IP header with new DSCP value and 802.1p value.

###Traffic destined to Virtual machine interface
1. If a tunneled packet is received DSCP value in the tunnel IP header would be used to look in qos-config table and corresponding DSCP value would be written to inner payload IP header.
2. If VM receives an IP packet from VM in same compute node, then DSCP value in IP header would be matched in qos-config and corresponding forwarding class would be used to overwrite IP header with new DSCP value and 802.1p value.

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
Queuing and scheduling will not be supported in 3.1   