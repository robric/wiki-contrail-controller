
Subnets can be created and deleted in a Virtual Network, each having its own attributes. The attributes that can be defined for each subnet, along with the subnet prefix and prefix length are listed below. Subnets are configured in virtual-network-network-ipam under the network-ipam(s) associated with the virtual-network, with as many 'ipam-subnets' as desired within each associated network-ipam.

<br>
1.  Default Gateway

***
* A default gateway for the subnet can be set explicitly, can be left to be allocated by Contrail, or can be disabled.
* 'default-gateway' is the corresponding field in ipam-subnets. When default gateway is to be disabled, set this field to 0.0.0.0. 
* Neutron commands <br>
`     neutron subnet-create test-vn 10.0.0.0/24 --gateway 10.0.0.1`<br>
`     neutron subnet-create test-vn 10.0.0.0/24 --no-gateway`

<br>
2. Allocation Pool

***
* Allocation pools with start and end addresses can be configured as 'allocation-pools'.
* VMs spawned are assigned addresses from within the allocation pool range.
* Neutron command <br>
`     neutron subnet-create test-vn 10.0.0.0/24 --allocation-pool start=10.0.0.100,end=10.0.0.200`

<br>
3. DHCP

***
* DHCP is enabled by default on each subnet.
* 'enable-dhcp' is the corresponding field in ipam-subnets (boolean).
* When enabled, the DHCP requests from the VMs are trapped to contrail-vrouter-agent, which responds with the IP address of the VM's interface along with other configured DHCP options.
* When disabled, the DHCP requests from the VMs are broadcast within the virtual network or are unicast for broadcast and unicast requests respectively.
* Neutron commands <br>
`     neutron subnet-create test-vn 10.0.0.0/24 —enable_dhcp` <br>
`     neutron subnet-create test-vn 10.0.0.0/24 —disable_dhcp`

<br>
4. DHCP options

***
* Contrail allows DHCP options to be configured at different levels - IPAM, Subnet, Interface
* Interface level has the highest precedence, followed by Subnet and IPAM in that order. Options defined at a higher precedence level override the options defined at lower precedence level. Thus, global DHCP options can be defined at IPAM or Subnet levels and specific options can be overridden at Subnet or Interface levels.
* In all these cases, dhcp-option-list can be configured, having a list of dhcp-optiontypes, each option having dhcp-option-name and dhcp-option-value. Complete list of options supported is here : https://github.com/Juniper/contrail-controller/wiki/Extra-DHCP-Options.
* Neutron command <br>
`     neutron port-create test-vn --extra-dhcp-opts list=true opt_name='dhcp_option_name',opt_value='value'`

<br>
5. Host Routes

***
* To update routes in the VM, host routes can be defined.
* Set of prefix and next-hop address for the prefix can be configured, which are then sent to the VM as classless static route option in DHCP response. 
* These can be configured as extra-dhcp-opts (see above) or in host-routes field in ipam-subnets. 
* Neutron command <br>
`     neutron subnet-create test-vn 10.0.0.0/24 --host_routes type=dict list=true destination=10.0.1.0/24,nexthop=10.0.0.3`

<br>
6. Subnet name

***
* Name for the subnet, 'subnet-name' is the corresponding field in ipam-subnets.
* Neutron command
`     neutron subnet-create test-vn 10.0.0.0/24 --name subnet-name`

<br>
7. DNS name servers

***
* Name servers for the subnet can be explicitly set.
* These are set using the DHCP option 'domain-name-servers' to the required address list (see DHCP options above).
* To disable DNS name servers in the subnet, set the DHCP option 'domain-name-servers' to 0.0.0.0.
* If this option is not set, the VM receives the service address as the DNS name server and contrail-vrouter-agent provides the DNS service for the subnet.
* Neutron command <br>
`     neutron subnet-create test-vn 10.0.0.0/24 --dns_nameservers list=true 8.8.8.7 8.8.8.8`

<br>
8. Service Address

***
* In every subnet, one address is reserved for Gateway (when enabled) and another is reserved for Services. This Services address is used as the DHCP server address and can be seen in all the DHCP responses.
* The same address is also used as the DNS server address for the subnet if DNS name server is not configured explicitly. In such a case, this address will be sent as domain-name-server in the DHCP response.
* 'dns-server-address' is the corresponding field in ipam-subnets. If not set, this address will be auto-allocated from the subnet (typically .2 address from the subnet).
