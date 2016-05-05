In Release 3.1, limited support to Graceful Restart (GR) and Long Lived Graceful Restart (LLGR) helper modes has been added to contrail-controller. This feature is deemed to be in "Beta" phase and not enabled by default.

## Reference
GracefulRestart for BGP (and XMPP) follows [RFC4724](https://tools.ietf.org/html/rfc4724) specifications and LongLivedGracefulRestart feature follows [draft-uttaro-idr-bgp-persistence](https://tools.ietf.org/html/draft-uttaro-idr-bgp-persistence-03) specifications.

## Applicability 
When ever a bgp peer (or contrail-vrouter-agent) session down is detected, all routes learned from the peer are deleted and also withdrawn immediately from advertised peers. This causes instantaneous disruption to traffic flowing end-to-end even if routes are kept inside vrouter kernel module (in data plane) intact. GracefulRestart and LongLivedGracefulRestart feature helps to alleviate this problem. When the session goes down, learned routes are not deleted and also not withdrawn from advertised peers for certain period. If session comes back up and routes are relearned, then impact to the network can be significantly contained.

Note: In 3.1, GR support in contrail-vrouter-agent is not present. It is only in contrail-control, does this take into effect. In future releases, GR/LLGR support shall be extended to contrail-vrouter-agent as well thus keeping end-to-end traffic intact during agent restarts.

## Feature highlights

* Support to advertise GR and LLGR capabilities in BGP (By configuring non-zero restart time)
* Support for GR and LLGR helper mode to retain routes even after sessions go down (By configuring helper mode)
* With GR is in effect, when ever a session down event is detected and close process is triggered, all routes (across all address families) are marked stale and remain eligible for best-path election for GracefulRestartTime duration (as exchanged)
* With LLGR is in effect, stale routes can be retains for much longer time than whats allowed by GR alone. In this phase, route preference is brought down and best paths are recomputed. Also LLGR_STALE community is tagged for stale paths. If NO_LLGR community is associated with any received stale route, then such routes are not kept (and deleted instead)
* GR/LLGR feature can be enabled for both BGP based and XMPP based peers
* BGP GR/LLGR configuration resides under BgpRouter and BgpSessionAttributes configuration sections
* XMPP GR/LLGR configuration resides under global-vrouter-config section