
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

#Queueing

From 3.2 release onwards, vRouter will provide the infrastructure to make use of the queues supplied by the network interface, also called hardware queueing. NICs that implement hardware queueing also come with their own set of scheduling algorithms associated with the queues. While the implementation will work with most of the NICs, Intel based 10G NIC (also called 'Niantic') is what was used for internal testing.

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
If a tunneled packet is received, the tunnel headers are stripped off and sent to interface. No marking from the outer packet to inner packet is done.

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

### Queue selection in datapath

The queue to send a packet to is specified by the forwarding class. While in almost every component of contrail where queue number specified is a logical number, in vRouter (i.e in the data path) that number specifies the actual hardware queue to which the packet needs to be send to. To facilitate this logical to physical transition, there needs to be a mapping and this mapping is specified in the vRouter-agent's configuration file. vRouter-agent, when it programs the vRouter, will program this translated queue number from the logical queue number that it has in its configuration.

#### Hardware queueing in Linux kernel based vRouter

If XPS or Xmit-Packet-Steering is enabled, which is the common case in Ubuntu based kernels, kernel will choose the queue to send a packet to based on the affinity each cpu has i.e each cpu has a list of queues it can send a packet to and the kernel will select one from the available set. If kernel selects the queue, then packets will not be sent to the vRouter specified queue. Hence, this mapping needs to be disabled. One can disable this mapping either by having a kernel without CONFIG_XPS option or by writing zeros to the mapping file in /sys/class/net/<ifname>/queues/tx-X/xps_cpus. Once this mapping is disabled, kernel will start sending the packets to the specific hardware queue. One can verify that packets indeed make it to the specified hardware queue by looking at individual queue statistics in the output of 'ethtool -S <ifname>' command. 

### Bandwidth control and scheduling algorithms in hardware based queueing

In the case of Intel 10G NIC, the QOS features come as part of a feature called DataCenterBridging (DCB). DCB is now an IEEE standards based feature that provides end to end QOS. In Linux, DCB feature is provided by the tools associated with a package called lldpad. If one wishes to use DCB, DCB has to be enabled at both ends of the wire i.e both on the compute as well as on the switch to which the compute is connected to. The 'lldpad' package also provides tools to configure the bandwidth and the scheduling algorithms to be used under a feature called ExtendedTransmissionSelection or ETS. Please refer to lldptool-ets(8) for the various configuration options.

If one doesn't want to enable DCB on both ends of the wire, one has the option to program the NIC with available functionality/interfaces provided by the Linux kernel under the DCB feature. In vRouter-utils package, there is a utility called 'qosmap' that allows configuration of bandwidth groups and bandwidths allotted to each of the group. It also allows one to specify whether this is a strict allocation or not. Bandwidth that is left after allotment to strict priority groups is divided in a round-robin manner.


## QoS queue provisioning using Fab
The QoS queue configuration provided in the forwarding class is a logical queue. The logical queues used are mapped to the physical queues supported in the NIC, this mapping is done in the contrail-vrouter-agent.conf on each compute node. Fab setup supports updating this mapping in the corresponding contrail-vrouter-agent configuration files. In addition, the scheduling algorithm and bandwidth values that are used by respective priority groups (in IEEE mode, we have one traffic class per priority group) can also be updated in the same agent configuration. The scheduling algorithm and bandwidth values can be read by a script that runs on the compute node to configure the priority groups.

The logical to physical mapping and scheduling, bandwidth values can be added in testbed.py in the following format.

     env.roledefs = {
         // Add below section in roledefs    
         'qos': [host4, host5]   
     }

     env.qos = {    
           host4: [     
           {'hardware_q_id': '1', 'logical_queue':['1', '6-10', '12-15'], 'scheduling': 'strict','bandwidth':'0'},   
           {'hardware_q_id': '2', 'logical_queue':['2-5'], 'scheduling': 'rr', 'bandwidth': '60'},  
           {'hardware_q_id': '3', 'logical_queue':['7'], 'scheduling': 'rr', 'bandwidth': '40', 'default': 'True'}],

           host5: [ 
           {'hardware_q_id': '1', 'logical_queue':['1', '3-8', '10-15'], 'scheduling': 'rr', 'bandwidth': '15'},
           {'hardware_q_id': '2', 'logical_queue':['9'], 'scheduling': 'strict', 'bandwidth': '0', 'default': 'True'}]
      }

      hardware_q_id  : Hardware queue identifier.   
      logical_queue  : Defines the logical queues that map to the hardware queue.   
      scheduling     : Defines the scheduling algorithm to be used by the corresponding priority group (strict / rr).  
      bandwidth      : Bandwidth to be used by the corresponding priority group when scheduling is round-robin.   
      default        : When set to True, defines the default hardware queue for Qos. All unspecified logical queues map to this hardware queue. One of the queue must be defined default.

###Generated contrail-vrouter-agent.conf
The above parameters are updated in /etc/contrail/contrail-vrouter-agent.conf on host4 as follows:  

     [QOS]

     [QUEUE-1]
     # Logical nic queues for qos config
     logical_queue= ['1', '6-10', '12-15']

     # Nic queue scheduling algorithm used
     scheduling= strict

     # Percentage of the total bandwidth used by queue
     bandwidth= 0

     [QUEUE-2]
     # Logical nic queues for qos config
     logical_queue= [2-5]

     # Nic queue scheduling algorithm used
     scheduling= rr

     # Percentage of the total bandwidth used by queue
     bandwidth= 60

     [QUEUE-3]
     # This is the default hardware queue
     default_hw_queue= true

     # Logical nic queues for qos config
     logical_queue= [7]

     # Nic queue scheduling algorithm used
     scheduling= rr

     # Percentage of the total bandwidth used by queue
     bandwidth= 40

# Caveats
Queuing and scheduling will not be supported in 3.1   

# Guidelines and Limitations:
  1. DCB feature supports 2 modes. One is IEEE and other is CEE.
     We recommend and provide provision to configure Bandwidth and Scheduling values. 
     User can use CEE mode as well but some limitations are present in that mode which are documented in following bug:
     https://bugs.launchpad.net/juniperopenstack/+bug/1630865

  2. On causing congestion within a single instance of VM and verifying scheduling, results will not be as per expectations. This is because of the fact that VM has its own queue and congestion within the VM will be handled by the queue of the VM(Not by the parent NIC interface queues).
     For more information, please refer to the following bug:
     https://bugs.launchpad.net/juniperopenstack/+bug/1634762

  3. For "Intel based 10G NIC(Niantic)", you will observe 32 queues initially. As soon as you enable dcb on the interface, it shows all 64 queues. Using "qosmap" utility to configure Bandwidth and Scheduling, automatically enables DCB and creates 64 queues.

  4.  It is observed that even if the highest priority queue is carrying full rate(10G) data, some very minor leaks happen on lower priority queues as well.(Assuming highest priority queue is configured for strict priority)