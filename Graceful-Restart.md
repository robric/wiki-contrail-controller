In Release 3.2, support to Graceful Restart (GR) and Long Lived Graceful Restart (LLGR) helper modes have been added to contrail-controller. This feature is not enabled by default.

### Reference
* GracefulRestart for BGP (and XMPP) follows [RFC4724](https://tools.ietf.org/html/rfc4724) specifications
* LongLivedGracefulRestart feature follows [draft-uttaro-idr-bgp-persistence](https://tools.ietf.org/html/draft-uttaro-idr-bgp-persistence-03) specifications
* [SystemTest plan](https://github.com/Juniper/contrail-test/wiki/Graceful-Restart-(Beta))
* [Unit Test](https://github.com/Juniper/contrail-controller/blob/master/src/bgp/test/graceful_restart_test.cc#L1180)

### Applicability 
When ever a bgp peer (or contrail-vrouter-agent) session down is detected, all routes learned from the peer are deleted and also withdrawn immediately from advertised peers. This causes instantaneous disruption to traffic flowing end-to-end even when routes are kept inside vrouter kernel module (in data plane) intact. GracefulRestart and LongLivedGracefulRestart features help to alleviate this problem.

When sessions goes down, learned routes are not deleted and also not withdrawn from advertised peers for certain period. Instead, they are kept as is and just marked as 'stale'. Thus, if sessions come back up and routes are relearned, the overall impact to the network is significantly contained.

### Feature highlights
* Support to advertise GR and LLGR capabilities in BGP (By configuring non-zero restart time)
* Support for GR and LLGR helper mode to retain routes even after sessions go down (By configuring helper mode)
* With GR is in effect, when ever a session down event is detected and close process is triggered, all routes (across all address families) are marked stale and remain eligible for best-path election for GracefulRestartTime duration (as exchanged)
* With LLGR is in effect, stale routes can be retained for much longer time than however long allowed by GR alone. In this phase, route preference is brought down and best paths are recomputed. Also LLGR_STALE community is tagged for stale paths and re-advertised. However, if NO_LLGR community is associated with any received stale route, then such routes are not kept and deleted instead
* After a certain time, if session comes back up, any remaining stale routes are deleted. If the session does not come back up, all retained stale routes are permanently deleted and withdrawn from advertised peers
* GR/LLGR feature can be enabled for both BGP based and XMPP based peers
* GR/LLGR configuration resides under global-system-config configuration section
***[Configuration parameters](https://github.com/Juniper/contrail-controller/blob/master/src/schema/vnc_cfg.xsd#L885)***
* GR timers can be configured by UI or via provision script. e.g.
```
/opt/contrail/utils/provision_control.py --api_server_ip 10.84.13.20 --api_server_port 8082 --router_asn 64512 --admin_user admin --admin_password c0ntrail123 --admin_tenant_name admin --host_name a6s20 --host_ip 10.84.13.20  --graceful_restart_time 300 --long_lived_graceful_restart_time 60000
```

When BGP Peering with JUNOS, JUNOS must also be explicitly configured for gr/llgr. e.g.
```
set protocols bgp group a6s20 type internal
set protocols bgp group a6s20 local-address 10.87.140.181
set protocols bgp group a6s20 keep all
set protocols bgp group a6s20 family inet-vpn unicast graceful-restart long-lived restarter stale-time 20
set protocols bgp group a6s20 family route-target
set protocols bgp group a6s20 graceful-restart restart-time 600
set protocols bgp group a6s20 neighbor 10.84.13.20 peer-as 64512

```

GR helper mode can be enabled for BGP and/or XMPP sessions by configuring gr_helper_enable in /etc/contrail/contrail-control.conf configuration file. For BGP, restart time shall be advertised in GR capability, as configured (in schema). end-of-rib receive and send timeout values can be tuned by configuring DEFAULT.bgp_end_of_rib_timeout and DEFAULT.xmpp_end_of_rib_timeout (in seconds) values

e.g.
```
/usr/bin/openstack-config /etc/contrail/contrail-control.conf DEFAULT gr_helper_bgp_enable 1
/usr/bin/openstack-config /etc/contrail/contrail-control.conf DEFAULT gr_helper_xmpp_enable 1
/usr/bin/openstack-config /etc/contrail/contrail-control.conf DEFAULT bgp_end_of_rib_timeout 30
/usr/bin/openstack-config /etc/contrail/contrail-control.conf DEFAULT xmpp_end_of_rib_timeout 30
service contrail-control restart
```

### Caveats
* GR/LLGR feature with a peer comes into effect either to all negotiated address-families or to none. i.e, if a peer signals support to GR/LLGR only for a subset of negotiated address families (Via bgp GR/LLGR capability advertisement), then GR helper mode does not come into effect for any family among the set of negotiated address families
* end-of-rib notification exchanged between control-node and xmpp agents is afi independent and signals eor for all address families