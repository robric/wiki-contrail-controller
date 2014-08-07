# VRouter management webpage

You can use access the VRouter management webpage on port 8085

http://localhost:8085/

Here is some useful webpages.

List of VRFs. 

http://localhost:8085/Snh_VrfListReq?name=

You can click link in ucindex to see routes

http://localhost:8085/Snh_Inet4UcRouteReq?x=8

Sandish message stats

http://localhost:8085/Snh_SandeshMessageStatsReq?

# Check Cassandra

You can use this script

https://gist.github.com/nati/8064561

```
ubuntu@contrail-01:~$ python cas.py virtual-network
default-domain:default-project:__link_local__ 3e072e3c-59a9-402e-b61f-3697b81a7274
default-domain:default-project:default-virtual-network 3795cb36-1338-4292-936a-5541fbd06ee6
default-domain:default-project:ip-fabric 8adc0fdd-674f-44cd-ac1b-87854dd0ec87
default-domain:demo:right 6884a3b2-cfbe-4fff-88bb-52ef36146e0e
```

# Discovery Service

Check ip for clients

curl "http://node_ip:5998/clients.json" | python -mjson.tool
