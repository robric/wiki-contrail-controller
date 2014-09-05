# Contrail Daemons' configuration files
Most of the configuration files follow the stand ini file format, with
name=value pairs divided into various appropriate sections. These values can
be _overridden_ from command line as well.

* /usr/bin/contrail-collector      [/etc/contrail/contrail-collector.conf](https://github.com/Juniper/contrail-controller/blob/master/src/analytics/contrail-collector.conf)
* /usr/bin/contrail-control        [/etc/contrail/contrail-control.conf](https://github.com/Juniper/contrail-controller/blob/master/src/control-node/contrail-control.conf)
* /usr/bin/contrail-vrouter-agent  [/etc/contrail/contrail-vrouter-agent.conf](https://github.com/Juniper/contrail-controller/blob/master/src/vnsw/agent/contrail-vrouter-agent.conf)
* /usr/bin/contrail-query-engine   [/etc/contrail/contrail-query-engine.conf](https://github.com/Juniper/contrail-controller/blob/master/src/query_engine/contrail-query-engine.conf)
* /usr/bin/dnsd                    [/etc/contrail/dns.conf](https://github.com/Juniper/contrail-controller/blob/master/src/dns/dns.conf)
* /usr/bin/contrail-discovery      [/etc/contrail/contrail-discovery.conf](https://github.com/Juniper/contrail-controller/blob/master/src/discovery/contrail-discovery.conf)
* /usr/bin/contrail-schema         [/etc/contrail/schema_transformer.conf](https://github.com/Juniper/contrail-packaging/blob/master/common/control_files/schema_transformer.conf)
* [/etc/contrail/contrail-schema.conf](https://github.com/Juniper/contrail-controller/blob/master/src/config/schema-transformer/contrail-schema.conf)
* /usr/bin/contrail-svc-monitor    [/etc/contrail/svc_monitor.conf](https://github.com/Juniper/contrail-controller/blob/master/src/config/svc-monitor/svc-monitor.conf)
* /usr/bin/contrail-analytics-api  [/etc/contrail/contrail-analytics-api.conf](https://github.com/Juniper/contrail-controller/blob/master/src/opserver/contrail-analytics-api.conf)
* /usr/bin/contrail-api            [/etc/contrail/contrail-api.conf](https://github.com/Juniper/contrail-controller/blob/master/src/config/api-server/contrail-api.conf)
* /usr/bin/contrail-nodemgr        /etc/contrail/contrail-nodemgr-database.conf

### Notes ###
* Use --help to see various options accepted
* A different configuration file can be provided using --conf_file=<config-file>
* Values from the config file can be overridden using command line option using
    --<SECTION>.<option>=<value> format. e.g. --DEFAULT.log_level=DEBUG
* Config values can be modified using the following command
  /usr/bin/openstack-config --set|--del config_file section [parameter] [value]
* When changes are made to the configuration file, the process must be
  _restarted_. (e.g. service supervisord-control restart).

When a contrail-package is installed via yum/dpkg, default configuration files are generated and placed under /etc/contrail/. unless one already exists.

###Precedence of configuration###
As mentioned before, configuration takes into affect (after daemon restart) in the following order of preference (highest to lowest)

* Command line
* Configuration file
* Default value (as shown in --help)

### Provisioning through fab ###
When fab is used to provision various roles such as controller, compute, etc. appropriate configuration files (in ini format) are generated and placed in /etc/contrail/. in appropriate nodes, based on the topology file specified.

### Transition from old configuration format to new ini based format ###
Following scripts can be used to convert old configuration files into new ones with the ini format.

* /opt/contrail/contrail_installer/contrail_config_templates/collector.conf.sh
* /opt/contrail/contrail_installer/contrail_config_templates/control-node.conf.sh
* /opt/contrail/contrail_installer/contrail_config_templates/dns.conf.sh
* /opt/contrail/contrail_installer/contrail_config_templates/query-engine.conf.sh
* /opt/contrail/contrail_installer/contrail_config_templates/vrouter-agent.conf.sh

```conf
e.g. old configuration file /etc/contrail/control_param
root@b6s23:~# cat /etc/contrail/control_param 
IFMAP_USER=192.168.68.1
IFMAP_PASWD=192.168.68.1
COLLECTOR=192.168.68.1
COLLECTOR_PORT=8086
DISCOVERY=192.168.69.2
HOSTNAME=b6s23
HOSTIP=192.168.68.1
BGP_PORT=179
CERT_OPTS=
LOGFILE=--log-file=/var/log/contrail/control.log
LOG_LOCAL=

> /opt/contrail/contrail_installer/contrail_config_templates/control-node.conf.sh
> cat /etc/contrail/contrail-control.conf
#
# Copyright (c) 2014 Juniper Networks, Inc. All rights reserved.
#
# Control-node configuration options, generated from /etc/contrail/control_param
#

[DEFAULT]
# bgp_config_file=bgp_config.xml
# bgp_port=179
# collectors= # Provided by discovery server
  hostip=192.168.68.1 # Resolved IP of single-node-253
  hostname=b6s23 # Retrieved as single-node-253
# http_server_port=8083
# log_category=
# log_disable=0
  log_file=/var/log/contrail/contrail-control.log
# log_files_count=10
# log_file_size=10485760 # 10MB
# log_level=SYS_NOTICE
# log_local=0
# test_mode=0
# xmpp_server_port=5269

[DISCOVERY]
# port=5998
  server=192.168.69.2 # discovery-server IP address

[IFMAP]
  certs_store=
  password=192.168.68.1
# server_url= # Provided by discovery server, e.g. https://127.0.0.1:8443
  user=192.168.68.1

```