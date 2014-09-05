Contrail Daemons' configuration files
=====================================
Most of the configuration files follow the stand ini file format, with
name=value pairs divided into various appropriate sections. These values can
be _overridden_ from command line as well.

Contrail Daemon                  Default-configuration file
===========================================================
/usr/bin/contrail-collector      /etc/contrail/contrail-collector.conf
                                 https://github.com/Juniper/contrail-controller/blob/master/src/analytics/contrail-collector.conf

/usr/bin/contrail-control        /etc/contrail/contrail-control.conf
                                 https://github.com/Juniper/contrail-controller/blob/master/src/control-node/contrail-control.conf

/usr/bin/contrail-vrouter-agent  /etc/contrail/contrail-vrouter-agent.conf
                                 /etc/contrail/rpm_agent.conf
                                 https://github.com/Juniper/contrail-controller/blob/master/src/vnsw/agent/contrail-vrouter-agent.conf

/usr/bin/contrail-query-engine   /etc/contrail/contrail-query-engine.conf
                                 https://github.com/Juniper/contrail-controller/blob/master/src/query_engine/contrail-query-engine.conf

/usr/bin/dnsd                    /etc/contrail/dns.conf
                                 https://github.com/Juniper/contrail-controller/blob/master/src/dns/dns.conf

/usr/bin/contrail-discovery      /etc/contrail/contrail-discovery.conf
                                 https://github.com/Juniper/contrail-controller/blob/master/src/discovery/contrail-discovery.conf

/usr/bin/contrail-schema         /etc/contrail/schema_transformer.conf
                                 https://github.com/Juniper/contrail-packaging/blob/master/common/control_files/schema_transformer.conf

/usr/bin/contrail-svc-monitor    /etc/contrail/svc_monitor.conf
                                 https://github.com/Juniper/contrail-controller/blob/master/src/config/svc-monitor/svc-monitor.conf

/usr/bin/contrail-analytics-api  /etc/contrail/contrail-analytics-api.conf
                                 https://github.com/Juniper/contrail-controller/blob/master/src/opserver/contrail-analytics-api.conf

/usr/bin/contrail-api            /etc/contrail/contrail-api.conf
                                 https://github.com/Juniper/contrail-controller/blob/master/src/config/api-server/contrail-api.conf

/usr/bin/contrail-nodemgr        /etc/contrail/contrail-nodemgr-database.conf

<UNKNOWN>                        /etc/contrail/contrail-schema.conf
                                 https://github.com/Juniper/contrail-controller/blob/master/src/config/schema-transformer/contrail-schema.conf
