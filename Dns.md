Four DNS modes are supported, IPAM configuration can select the DNS mode required.

## 1. None
No DNS support for the VMs.

## 2. Default DNS server
DNS resolution for the VMs is done based on the name server configuration in the server infrastructure. When a VM gets a DHCP response, the subnet default gateway is configured as the DNS server for the VM. DNS requests that the VM sends sent to this default gateway are resolved via the name servers configured on the respective compute nodes and the responses are sent back to the VM.

## 3. Tenant DNS server
Tenants can use their own DNS servers using this mode. A list of servers can be configured in the IPAM, which are then sent in the DHCP response to the VM as DNS server(s). DNS requests that the VM sends are routed as any other data packet based on the available routing information.

## 4. Virtual DNS server
In this mode, the system supports virtual DNS servers, providing DNS servers that resolve the DNS requests from the VMs. We can define multiple virtual domain name servers under each domain in the system. Each virtual domain name server is an authoritative server for the DNS domain configured. 

The following properties can be configured for each virtual DNS server:

1. Domain name (e.g. juniper.net)

2. Record order : when a name has multiple records matching, this determines the order in which the records are sent in the response. 
    * fixed - the records are sent in the order of creation
    * round-robin - the record order is cycled for each request to the record
    * random - the records are sent in random order

3. TTL in seconds : default time to live for the records in the domain

4. Next DNS server : indicates the next DNS server to send the request to when the request cannot be served in the context of the current virtual DNS server. 
    * When configured, the next DNS server can either be another virtual DNS server defined in the system or the IP address of an external DNS server which is reachable from the server infrastructure
    * Using the next DNS servers, we can configure a hierarchy of servers thru which the DNS requests are iterated

Each IPAM in the system can refer to one of the virtual DNS servers configured (when DNS mode is chosen as _Virtual DNS Server_). The virtual networks and virtual machines spawned will fall under the DNS domain of the virtual DNS server specified in the corresponding IPAM. The VMs will receive this domain in the DHCP DOMAIN-NAME option.