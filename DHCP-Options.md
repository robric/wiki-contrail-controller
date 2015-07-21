Contrail allows DHCP options to be configured at different levels:
* IPAM
* Subnet
* Interface

Interface level has the highest precedence, followed by Subnet and IPAM in that order. Options defined at a higher precedence level override the options defined at lower precedence level. Thus, global DHCP options can be defined at IPAM or Subnet levels and specific options can be overridden at Subnet or Interface levels.

In all these cases, `dhcp-option-list` can be configured, having a list of `dhcp-option` types, each option having `dhcp-option-name` and `dhcp-option-value`. Complete list of options supported is listed here : https://github.com/Juniper/contrail-controller/wiki/Extra-DHCP-Options.

In every subnet, one address is reserved for Gateway and another is reserved for Services. This address (typically `.2` address from the subnet) is used as the DHCP server address and can be seen in all the DHCP responses. The same address is also used as the DNS server address for the subnet and will be sent in the DHCP response as `domain-name-server`, unless the same option is configured or disabled (see below).

## Disable DNS

To disable DNS service, set `domain-name-servers` option (DHCP code = `6`) to `0.0.0.0`. In this case, the DHCP response sent from contrail-vrouter-agent will not include any domain name server.