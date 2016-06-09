In Release 3.1, limited support to Graceful Restart (GR) and Long Lived Graceful Restart (LLGR) helper modes has been added to contrail-controller. This feature is deemed to be in "Beta" phase and not enabled by default.

### Reference
* GracefulRestart for BGP (and XMPP) follows [RFC4724](https://tools.ietf.org/html/rfc4724) specifications
* LongLivedGracefulRestart feature follows [draft-uttaro-idr-bgp-persistence](https://tools.ietf.org/html/draft-uttaro-idr-bgp-persistence-03) specifications

### Applicability 
When ever a bgp peer (or contrail-vrouter-agent) session down is detected, all routes learned from the peer are deleted and also withdrawn immediately from advertised peers. This causes instantaneous disruption to traffic flowing end-to-end even when routes are kept inside vrouter kernel module (in data plane) intact. GracefulRestart and LongLivedGracefulRestart features help to alleviate this problem. When sessions goes down, learned routes are not deleted and also not withdrawn from advertised peers for certain period. Instead, they are kept as is and just marked as 'stale'. Thus, if sessions come back up and routes are relearned, then overall impact to the network can be significantly contained.

### Feature highlights
* Support to advertise GR and LLGR capabilities in BGP (By configuring non-zero restart time)
* Support for GR and LLGR helper mode to retain routes even after sessions go down (By configuring helper mode)
* With GR is in effect, when ever a session down event is detected and close process is triggered, all routes (across all address families) are marked stale and remain eligible for best-path election for GracefulRestartTime duration (as exchanged)
* With LLGR is in effect, stale routes can be retained for much longer time than however long allowed by GR alone. In this phase, route preference is brought down and best paths are recomputed. Also LLGR_STALE community is tagged for stale paths and re-advertised. However, if NO_LLGR community is associated with any received stale route, then such routes are not kept and deleted instead
* After a certain time, if session comes back up, any remaining stale routes are deleted. If the session does not come back up, all retained stale routes are permanently deleted and withdrawn from advertised peers
* GR/LLGR feature can be enabled for both BGP based and XMPP based peers
* BGP GR/LLGR configuration resides under BgpRouter and BgpSessionAttributes configuration sections
* XMPP GR/LLGR configuration resides under global-vrouter-config section

### Caveats (3.1)
* GR support in contrail-vrouter-agent is not present. It is only in contrail-control, does this take into effect. In future releases, GR/LLGR support shall be extended to contrail-vrouter-agent as well thus keeping end-to-end traffic intact during agent restarts.
* GR/LLGR feature with a peer comes into effect either to all negotiated address-families or to none. i.e, if a peer signals support to GR/LLGR only for a subset of negotiated address families (Via bgp GR/LLGR capability advertisement), then GR helper mode does not come into effect for all of the negotiated address families