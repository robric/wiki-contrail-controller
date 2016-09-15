#(1) POST for port creation/updation
 
###http://localhost:9091/port (POST)
 
###Json for Single port creation/updation
 
{"port": {"uuid": "1a9b556b-e4a9-4236-a9ca-559deb5de5a9", "instance_uuid": "2434abce-73dd-48ca-8577-a5b83859383f", "ip_address": "11.0.0.6", "ip6_address":"::", "vn_uuid": "37e5d18d-601e-4215-b092-ba507924c14a", "display_name": "vn1vm4", "vm_project_id": "e15b0375-bb9d-4163-8636-76c52f68dd17", "mac_address": "02:1a:9b:55:6b:e4", "system_name": "tap1a9b556b-e4", "type": 0, "rx_vlan_id": 65535, "tx_vlan_id": 65535}}
 
####Example:
curl -X POST -H "Content-Type: application/json" -d '{"ip-address": "11.0.0.11", "rx-vlan-id": 1999, "display-name": "vm1", "id": "753a65ef-918a-4193-9266-dc23ed8a5982", "instance-id": "63bd1a8a-ba6a-4547-af8d-653dd8ff397e", "ip6-address": "None", "tx-vlan-id": 2000, "vn-id": "38ced7ff-fdfc-4f17-9547-17ff4fb45da9", "vm-project-id": "cb33cc2c-6fb8-4072-a112-77547c60d2b7", "type": 2, "mac-address": "02:75:3a:65:ef:91", "system-name": "753a65ef-918a-4193-9266-dc23ed8a5981"}'  http://localhost:9091/port

###Json for Multiple ports creation/updation
 
[{"uuid": "1a9b556b-e4a9-4236-a9ca-559deb5de5a9", "instance_uuid": "2434abce-73dd-48ca-8577-a5b83859383f", "ip_address": "11.0.0.6", "ip6_address":"::", "vn_uuid": "37e5d18d-601e-4215-b092-ba507924c14a", "display_name": "vn1vm4", "vm_project_id": "e15b0375-bb9d-4163-8636-76c52f68dd17", "mac_address": "02:1a:9b:55:6b:e4", "system_name": "tap1a9b556b-e4", "type": 0, "rx_vlan_id": 65535, "tx_vlan_id": 65535}, {}]
 
####Example:
curl -X POST -H "Content-Type: application/json" -d '[{"ip-address": "11.0.0.11", "rx-vlan-id": 1999, "display-name": "vm1", "id": "753a65ef-918a-4193-9266-dc23ed8a5982", "instance-id": "63bd1a8a-ba6a-4547-af8d-653dd8ff397e", "ip6-address": "None", "tx-vlan-id": 2000, "vn-id": "38ced7ff-fdfc-4f17-9547-17ff4fb45da9", "vm-project-id": "cb33cc2c-6fb8-4072-a112-77547c60d2b7", "type": 2, "mac-address": "02:75:3a:65:ef:91", "system-name": "753a65ef-918a-4193-9266-dc23ed8a5981"}, {"ip-address": "11.0.0.12", "rx-vlan-id": 1999, "display-name": "vm1", "id": "753a65ef-918a-4193-9266-dc23ed8a5983", "instance-id": "63bd1a8a-ba6a-4547-af8d-653dd8ff397f", "ip6-address": "None", "tx-vlan-id": 2000, "vn-id": "38ced7ff-fdfc-4f17-9547-17ff4fb45da9", "vm-project-id": "cb33cc2c-6fb8-4072-a112-77547c60d2b7", "type": 2, "mac-address": "02:75:3a:65:ef:92", "system-name": "753a65ef-918a-4193-9266-dc23ed8a5982"}]'  http://localhost:9091/port

#(2) POST for sync of ports

###http://localhost:9091/syncports (POST)
This is used for syncing of ports between plug-in (REST client) and agent across plug-in restarts. Typically, when plug-in (client) restarts, it issues this command and then issues adds for ports using the above (item number 1) API. Agent, upon receiving this command starts a timer. It expects plug-in to replay all ports before the expiry of this timer.  On expiry of timer, agent will remove all ports which have not been added by plug-in after issue of syncports command.
 
####Example:
curl -X POST http://localhost:9091/syncports
 
#(3) POST for GET of port
 
http://localhost:9091/port/uuid (GET)

####Example:
curl -X GET http://localhost:9091/port/753a65ef-918a-4193-9266-dc23ed8a5982
 
#(4) POST for DELETE of port
 
http://localhost:9091/port/uuid

####Example:
curl -X DELETE http://localhost:9091/port/753a65ef-918a-4193-9266-dc23ed8a5982

