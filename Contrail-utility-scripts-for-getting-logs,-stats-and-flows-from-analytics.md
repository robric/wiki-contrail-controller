This document gives details of the utility scripts that allow you to query and get data from contrail analytics. The collection and storage of the data is not discussed here.

* _contrail-logs_: contrail-logs can be used to get system logs and object logs in the contrail system
* _contrail-stats_: contrail-stats can be used to get generic stats using the virtual stat tables
* _contrail-flows_: contrail-flows can be used to get flow information  
  
Each of the commands allow you filter, aggregate data in any slice and dice manner and are powerful debug tools. All of these queries can be done through Contrail UI as well.

# contrail-logs
--help option provides all the information with respect to the command's options as below.  

> root@a7s30:~# contrail-logs --help  
> usage: contrail-logs [-h] [--analytics-api-ip ANALYTICS_API_IP]  
>                      [--analytics-api-port ANALYTICS_API_PORT]  
>                      [--start-time START_TIME] [--end-time END_TIME]  
>                      [--last LAST] [--source SOURCE]  
>                      [--node-type {Invalid,Config,Control,Analytics,Compute,WebUI,Database,OpenStack,ServerMgr}]  
>                      [--module {contrail-control,contrail-vrouter-agent,contrail-api,contrail-schema,contrail-analytics-api,contrail-collector,contrail-query-engine,contrail-svc-monitor,DeviceManager,contrail-dns,contrail-discovery,IfmapServer,XmppServer,contrail-analytics-nodemgr,contrail-control-nodemgr,contrail-config-nodemgr,contrail-database-nodemgr,Contrail-WebUI-Nodemgr,contrail-vrouter-nodemgr,Storage-Stats-mgr,Ipmi-Stats-mgr,contrail-snmp-collector,contrail-topology,InventoryAgent,contrail-alarm-gen,contrail-tor-agent}]  
>                      [--instance-id INSTANCE_ID] [--category CATEGORY]  
>                      [--level LEVEL] [--message-type MESSAGE_TYPE] [--reverse]  
>                      [--verbose] [--all] [--raw]  
>                      [--object-type {service-chain,database-node,routing-instance,xmpp-connection,analytics-query,virtual-machine-interface,config-user,analytics-query-id,None,logical-interface,xmpp-peer,generator,virtual-network,analytics-node,prouter,bgp-peer,config,dns-node,control-node,physical-interface,None,virtual-machine,vrouter,None,None,service-instance,config-node}]  
>                      [--object-values] [--object-id OBJECT_ID]  
>                      [--object-select-field {ObjectLog,SystemLog}]  
>                      [--trace TRACE] [--limit LIMIT] [--send-syslog]  
>                      [--syslog-server SYSLOG_SERVER]  
>                      [--syslog-port SYSLOG_PORT] [--f] [--keywords KEYWORDS]  
>   
>   
> optional arguments:  
>   -h, --help            show this help message and exit  
>   --analytics-api-ip ANALYTICS_API_IP  
>                         IP address of Analytics API Server (default:  
>                         127.0.0.1)  
>   --analytics-api-port ANALYTICS_API_PORT  
>                         Port of Analytics API Server (default: 8081)  
>   --start-time START_TIME  
>                         Logs start time (format now-10m, now-1h) (default:  
> ...  
>   --syslog-port SYSLOG_PORT  
>                         Port to send syslog to (default: 514)  
>   --f                   Tail logs from now (default: False)  
>   --keywords KEYWORDS   comma seperated list of keywords (default: None)  
>   --message-types       Display list of message type (default: False)  

## contrail-logs systemlog examples:

* View all logs from all the boxes for the default time [which is last 10 minutes]:

`/usr/bin/contrail-logs`

* View all logs from all the boxes for the last hour:

`/usr/bin/contrail-logs --last 1h`

* View all logs from all boxes for 1 hour between 11 hours ago and 10 hours ago:

`/usr/bin/contrail-logs --start-time now-11h --end-time now-10h`

* View logs in a specific time interval specificed in date format:

`/usr/bin/contrail-logs --start-time "2013 May 12 18:30:27.0" --end-time "2013 May 12 18:31:27.0"`

* View all logs from all boxes for the last 10 minutes in reverse chronological order:

`/usr/bin/contrail-logs --reverse`

* module filtering: View the contrail-control daemon logs from all the boxes for the last 10 minutes:

`/usr/bin/contrail-logs --module contrail-control`

* module and source filtering: View the contrail-control daemon logs from source a6s23.contrail.juniper.net for the last 10 minutes:

`/usr/bin/contrail-logs --module contrail-control --source a6s23.contrail.juniper.net`

## contrail-logs objectlog examples:
This is used to get all the logs relevant for a particular object in the system

* View logs relevant to a virtual network object by name demo:admin:vn1 from all the boxes for the last 10 minutes:

`/usr/bin/contrail-logs --object-type virtual-network --object-id demo:admin:vn1`

* View logs relevant to an analytics-node object with name a7s30 for the last 10 minutes:

`contrail-logs --object-type analytics-node --object-id a7s30`

## object-logs object-values example
Typical work flow would be to find out all the object-values [keys] of a particular object-type and then get detailed logs of a particular object-id. The following example shows how to get object-values of type control-node, and then subsequently get logs of a particular object-id.

* List all the objects of type control-node for which messages have been received in the last 10 minutes:

`/usr/bin/contrail-logs --object-type control-node --object-values`

* View logs of a particular object of control-node type

`/usr/bin/contrail-logs --object-type control-node --object-id a7s30`

## contrail config audit example
Auditing of config is of very particular importance as it tells the sequence of operations on the config objects.
The object-type used in this case is config.

* List all the objects of type config for which messages have been received in the last 10 minutes:
`contrail-logs --object-type config --object-values`

* List all the logs of object routing_instance:default-domain:admin:vn5:vn5 of type config in the last 10 minutes:
`contrail-logs --object-type config --object-id routing_instance:default-domain:admin:vn5:vn5`

Please use the `--help` option to list the arguments supported by contrail-logs.