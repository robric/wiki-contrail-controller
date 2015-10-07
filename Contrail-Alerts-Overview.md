# Summary  

Alerts will be provided on a per-UVE basis.
Contrail Analytics will raise (or clear) alerts using python-coded “rules” that examine the contents of the UVE and the object’s config.
Some rules will be built-in. Others can be added using python stevedore plugins.  
  
The Contrail Analytics API will provide the following.  
* Read access to the Alerts as part of the UVE GET APIs   
* Alert acknowledgement using POST requests.  
* UVE and Alert streaming using Server-Sent Events :http://dev.w3.org/html5/eventsource/  
  
## Alert format:  
  
GET http://\<analytics-ip\>:8081/analytics/uves/control-node/a6s40?flat  


    {
        NodeStatus:  {…},
        ControlCpuState:  {…},
        UVEAlarms:  {
        alarms:  [
            {
                description:  [
                    {
                    value: "0 != 2",
                    rule: "BgpRouterState.num_up_bgp_peer != BgpRouterState.num_bgp_peer"
                    }
                ],
                ack: false,
                timestamp: 1442995349253178,
                token: "eyJ0aW1lc3RhbXAiOiAxNDQyOTk1MzQ5MjUzMTc4LCAiaHR0cF9wb3J0IjogNTk5NSwgImhvc3RfaXAiOiAiMTAuODQuMTMuNDAifQ==",
                type: "BgpConnectivity",
                severity: 4
            }
        ]
        },
        BgpRouterState:  {…}
    }

As can be seen from above, Alerts are raised on a per-UVE basis and can be retrieved by a GET on a UVE.  
“ack” indicates if the alert has been acknowledged or not.  
“token” is to be used by clients when requesting acknowledgements   
  
  
## Analytics APIs for Alerts:  
  
1. GET _http://\<analytics-ip\>:\<rest-api-port\>/analytics/uves/control-node/aXXsYY&cfilt=UVEAlarms_  
This will give a list of alerts raised against Control Node aXXsYY.
This is available for all UVE table-types.
 
2. GET _http://\<analytics-ip\>:\<rest-api-port\>/analytics/alarms_  
This will give a list of all alarms in the system.
 
3. POST _http://\<analytics-ip\>:\<rest-api-port\>/analytics/alarms/acknowledge_  
       Body: {“table”: \<object-type\>, “name”: \<key\>, “type”: \<alarm type\>, “token”: \<token\>}
This allows the user to acknowledge an alarm.
Acknowledged/un-acknowledged alarms can be queried specifically by using the URL Query Parameter “ackFilt=True” or “ackFilt=False” with APIs #1 and #2 above.  
 
4. GET _http://\<analytics-ip\>:\<rest-api-port\>/analytics/uve-stream?tablefilt=control-node_  
This provides a SSE-based stream of UVE updates for Control Node alarms.
This is available for all UVE table-types. If the “tablefilt” URL Query parameter is not provided, all UVEs will be seen.
 
5. GET _http://\<analytics-ip\>:\<rest-api-port\>/analytics/alarms-stream?tablefilt=control-node_  
This is similar to #4 above, but it provides only the Alerts portion of UVEs instead of providing the entire content of the UVEs.  
This is available for all UVE table-types. If the “tablefilt” URL Query parameter is not provided, alerts for all UVEs will be seen.
  
  
## Built-in Node Alerts:  
The following built-in node alerts are supported and can be retrieved using APIs listed in the previous section.  
  
    control-node:  {
        PartialSysinfoControl: "Basic System Information is absent for this node in BgpRouterState.build_info",
        ProcessStatus: "NodeMgr reports abnormal status for process(es) in NodeStatus.process_info",
        XmppConnectivity: "Not enough XMPP peers are up in BgpRouterState.num_up_bgp_peer",
        BgpConnectivity: "Not enough BGP peers are up in BgpRouterState.num_up_bgp_peer",
        AddressMismatch: “Mismatch between configured IP Address and operational IP Address",
        ProcessConnectivity: "Process(es) are reporting non-functional components in NodeStatus.process_status"
    },
 
    vrouter:  {
        PartialSysinfoCompute: "Basic System Information is absent for this node in VrouterAgent.build_info",
        ProcessStatus: "NodeMgr reports abnormal status for process(es) in NodeStatus.process_info",
        ProcessConnectivity: "Process(es) are reporting non-functional components in NodeStatus.process_status",
        VrouterInterface: "VrouterAgent has interfaces in error state in VrouterAgent.error_intf_list”,
        VrouterConfigAbsent: “Vrouter is not present in Configuration”,
    },
 
    config-node:  {
        PartialSysinfoConfig: "Basic System Information is absent for this node in ModuleCpuState.build_info",
        ProcessStatus: "NodeMgr reports abnormal status for process(es) in NodeStatus.process_info",
        ProcessConnectivity: "Process(es) are reporting non-functional components in NodeStatus.process_status"
    },
 
    analytics-node:  {
        ProcessStatus: "NodeMgr reports abnormal status for process(es) in NodeStatus.process_info",
        PartialSysinfoAnalytics: "Basic System Information is absent for this node in CollectorState.build_info",
        ProcessConnectivity: "Process(es) are reporting non-functional components in NodeStatus.process_status"
    },
