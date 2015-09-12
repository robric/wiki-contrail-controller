Much analytics documenation and information can be obtained from the REST API
of the contrail-analytics-api server.

_http://\<contrail-analytics-api-IP\>:8081/documentation/index.html_  
Provides information regarding the data available from analytics-api server
and how to query for the same.

# UVE [User Visible Entities]
_http://\<contrail-analytics-api-IP\>:8081/analytics/uves_  
Provides list of all UVE-types available in the system, examples include
virtual-networks, analytics-nodes, service-chains, bgp-peers, prouters etc.
The type names should be self explanatory.  
And on drilling down each set for e.g  
_http://\<contrail-analytics-api-IP\>:8081/analytics/uves/virtual-networks_  
provides all instances of this UVE type.  
And drilling down further for e.g.  
_http://\<contrail-analytics-api-IP\>:8081/analytics/uves/virtual-network/default-domain:admin:vn3?flat_  
provides the operating data for a specific UVE, in this case, for virtual network _default-domain:admin:vn3_

Same goes for other UVE types, for e.g. the following give operating data for
prouters - Physical Routers.  
_http://\<contrail-analytics-api-IP\>:8081/analytics/uves/prouters_  
_http://\<contrail-analytics-api-IP\>:8081/analytics/uves/prouter/a7-ex1?flat_  

# Analytics Table Queries
Most analytics data is queried using SQL-like queries into tables whose schema
are exposed via REST API - exact mechanism is in contrail-analytics-api server
documentation.  
_http://\<contrail-analytics-api-IP\>:8081/analytics/tables_  
Provides the list of all queryable tables and drilling down will give schema
for each of the tables, for e.g.  
_http://\<contrail-analytics-api-IP\>:8081/analytics/table/StatTable.VirtualMachineStats.cpu_stats/schema_  
provides schema for a table named StatTable.VirtualMachineStats.cpu_stats

There are a few types of tables - their type returned in the return list of tables  
* LOG - Only MessageTable is of this type and is used to query system logs  
* OBJECT - There are multipe tables of this type each corresponding to a particular object  
* FLOW - FlowRecordTable and FlowSeriesTable fall into category and are used to get flow information  
* STAT - There are many generic statistics tables  

To reiterate the query will be of the same format, SQL-like query in JSON, for all
tables. An example would be  

{  
start_time: "now-10m" ,   
end_time: "now" ,  
table: "StatTable.UveVirtualNetworkAgent.vn_stats" ,   
select_fields: ["SUM(vn_stats.in_bytes)", "vn_stats.other_vn"] ,   
where: [[{"value": "default-domain:admin:vn1", "op": 1, "suffix": null, "name": "name", "value2": null}]] }  
  
# Data Collection Frequency

Tables of most interest and their corresponding frequency are as below

The following info is collected every 60 seconds  
_http://\<contrail-analytics-api-IP\>:8081/analytics/table/StatTable.AnalyticsCpuState.cpu_info_  
_http://\<contrail-analytics-api-IP\>:8081/analytics/table/StatTable.ConfigCpuState.cpu_info_  
_http://\<contrail-analytics-api-IP\>:8081/analytics/table/StatTable.ControlCpuState.cpu_info_  
_http://\<contrail-analytics-api-IP\>:8081/analytics/table/StatTable.ComputeCpuState.cpu_info_  

The below stats are collected every 30 seconds  
_http://\<contrail-analytics-api-IP\>:8081/analytics/table/StatTable.UveVirtualNetworkAgent.vn_stats_  
_http://\<contrail-analytics-api-IP\>:8081/analytics/table/StatTable.VirtualMachineStats.cpu_stats_  
_http://\<contrail-analytics-api-IP\>:8081/analytics/table/StatTable.UveVMInterfaceAgent.fip_diff_stats_  
_http://\<contrail-analytics-api-IP\>:8081/analytics/table/StatTable.UveVMInterfaceAgent.if_stats_  
