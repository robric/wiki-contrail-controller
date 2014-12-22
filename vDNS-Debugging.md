## Steps to debug virtual-DNS issues

If there is a specific host/VM not getting resolved,

* Find hypervisor on which requesting VM is running (via `nova show` or alternate)
* Access http://hypervisor-ip:8085/Snh_SandeshTraceRequest?x=DnsBind to get how vrouter-agent handled request for dns resolution
* Find hypervisor on which VM whose name is not resolving
* Access http://hypervisor-ip:8085/Snh_SandeshTraceRequest?x=XmppMessageTrace to see if vrouter-agent sent the VM details to the contrail-dns service
* Access http://controller-ip:8092/Snh_SandeshTraceRequest?x=XmppMessageTrace to see if contrail-dns received message from vrouter-agent about VM presence
* Access http://controller-ip:8092/Snh_SandeshTraceRequest?x=DnsBind to see whether contrail-dns programmed bind entries correctly on every controller
* grep <vm-name> in /etc/contrail/dns/* in every controller to check presence of entry
