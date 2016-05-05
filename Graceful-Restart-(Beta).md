In Release 3.1, limited support to Graceful Restart (GR) and Long Lived Graceful Restart (LLGR) helper modes has been added to contrail-controller.

GracefulRestart for BGP (and XMPP) follows [RFC4724](https://tools.ietf.org/html/rfc4724) specifications and LongLivedGracefulRestart feature follows [draft-uttaro-idr-bgp-persistence](https://tools.ietf.org/html/draft-uttaro-idr-bgp-persistence-03) specifications.

Some of the highlights of this feature are listed below

* Support to advertise GR and LLGR capabilities in BGP (By configuring non-zero restart time)
* Support for GR and LLGR helper mode to retain routes even after sessions go down (By configuring helper mode)
* With GR is in effect, when ever a session down event is detected and close process is triggered, all routes (across all address families) are marked stale and remain eligible for best-path election for GracefulRestartTime duration (as exchanged)
* With LLGR is in effect, stale routes can be retains for much longer time than whats allowed by GR alone. In this phase, route preference is brought down and best paths are recomputed. Also LLGR_STALE community is tagged for stale paths. If NO_LLGR community is associated with any received stale route, then such routes are not kept (and deleted instead)
* GR/LLGR feature can be enabled for both BGP based and XMPP based peers
* BGP GR/LLGR configuration resides under BgpRouter and BgpSessionAttributes configuration sections
* XMPP GR/LLGR configuration resides under global-vrouter-config section