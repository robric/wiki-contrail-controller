Four DNS modes are supported in the system.

## None
No DNS support is required for the VMs.

## Default DNS server
DNS resolution for the VMs is done based on the name server configuration in the server infrastructure. When a VM gets a DHCP response, the subnet default gateway is configured as the DNS server for the VM. DNS requests that the VM sends sent to this default gateway are resolved via the name servers configured on the respective compute nodes and the responses are sent back to the VM.

## Tenant DNS server
Tenants can use their own DNS servers using this mode. A list of servers can be configured in the IPAM, which are then sent in the DHCP response to the VM as DNS server(s). DNS requests that the VM sends are routed as any other data packet based on the available routing information.

## Virtual DNS server
In this mode, the system supports virtual DNS servers, providing DNS servers that resolve the DNS requests from the VMs. Each IPAM can configure a virtual DNS server. 
