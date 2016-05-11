
# Overview
QoS in networking guarantees reliability, bandwidth, latency and better management of network traffic upon congestion. Network traffic gets marked with QoS bits (DSCP, 802.1p etc) which intermediate network switches and routes use to provide priority.

QoS feature in contrail would be supported from 3.1

# Implementation details

## Configuration objects.
1. Forwarding-class: specifies knobs for marking and queuing. 
    1. If packet is IP packet, then it specifes DSCP value to be written on packet  
    2. If packet is non-IP layer-2 packet, then it specifies 802.1p value to be written on packet

2. Queue: specifies minimum and maximum bandwidth values and priority of the traffic.

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

## Example QoS config object on virtual-machine-interface, vhost or fabric interface 
Lets apply above example QoS config on virtual-machine-interface or vhost or fabric interface

1. If interface sends a IP packet with DSCP value *18*, then packet would be remarked with DSCP value *10* and enqueued to *queue 0* as specified by *FC1* with ID *1* 
2. If interface sends a IP packet with DSCP value *22*, then packet would map to default rule which specifies that DSCP value *38* to be written and queued to queue *1* as specified by *FC2* with ID *2*
3. If a layer 2 packet is received with 802.1p value of *5*, then packet would be remarked with 802.1p value of *7* and enqueue to *queue 0* as specified by *FC1*

## Example QoS config object on virtual-network.
If the above example qos config is applied on virtual-network, then all the intra-vn traffic would be 
subjected to the above translation rules.

## Example QOS config on network policy or SG.
If the above example qos config is applies in policy, the all the packet matching that policy
would be subjected to above translation rules.  

# Caveats
1. NIC queues would be used.
1. Built in or default scheduler in NIC would be used.        