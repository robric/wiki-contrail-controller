Disk usage by analytics db can be monitored from the contrail UI page at  
**Monitor -> Infrastructure -> Database Nodes**  
or
from database-node UVEs at  
_\<analytics-node-ip\>:8081/analytics/uves/database-node/*?cfilt=DatabaseUsageInfo_  

In an ideal scenario, the sizing should be done such that an **external** purging [as opposed to cassandra's own compaction] is not required.  

When the above [ideal sizing] does not happen, one can rely on the following purge methods  

_contrail-analytics-api_ process continuously monitors the disk usage and tries to purge the data when it exceeds certain value. 

If one needs to manually send a request to _contrail-analytics-api_, to purge beyond what it does automatically, it can be done from the UI using the grid's menu button at  
**Monitor -> Infrastructure -> Database Nodes**  
Another way to send a request to _contrail-analytics-api_ to purge is to do a POST request to    
_\<analytics-node-ip\>:8081/analytics/operation/database-purge_  
the POST data is the percentage to purge and is input in JSON format, as below,  
{ "purge_input" : 30 }

The above purge method is a slow process, as deleting the data from database does not immediately free up the disk space which depends on cassandra doing the compaction, which may depend on various factors including system load etc..  
In extreme cases, where one needs to manually purge analytics data, one of the following methods can be used  

**Method 1**: will save latest analytics data, but involves stopping cassandra on all nodes [this will disrupt config operations]  
a) stop cassandra on all nodes [_service contrail-database stop_]  
b) delete files /var/lib/cassandra/data/ContrailAnalytics that are older than **some time** on all database nodes  
c) restart cassandra  

**Method 2**: lose all analytics data, but cassandra can serve config operations  
a) stop collectors on all nodes  [_service contrail-collector stop_]
b) using cqlsh or cassandra-cli - drop ContrailAnalytics keyspace [this will take a bit of time]  
c) verify ContrailAnalytics keyspace is indeed not present in any node  
d) delete /var/lib/cassandra/data/ContrailAnalytics directory  
d) restart collectors

