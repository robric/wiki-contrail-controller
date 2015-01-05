The analytics db usage viewing and purging, if required, can be done from the contrail UI page.

The page is accessed from Setting -> Analytics DB tab. One can view the overall disk usage and the size of analytics data. If required, one can purge the analytics data using the menu item on the page.

Another way to purge analytics data is to do a POST request to  
\<analytics-node-ip\>:8081/analytics/operation/database-purge  
the POST data is the percentage to purge and is input in JSON format, as below,  
{ "purge_input" : 30 }

