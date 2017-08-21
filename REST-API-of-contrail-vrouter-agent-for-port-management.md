# 1 POST for port creation/updation

### http://localhost:9091/port (POST)
 
### Json for Single port creation/updation
```
{
  "type": 2,
  "id": "753a65ef-918a-4193-9266-dc23ed8a5982",
  "instance-id": "63bd1a8a-ba6a-4547-af8d-653dd8ff397e",
  "display-name": "vm1",
  "ip-address": "11.0.0.11",
  "ip6-address": "None",
  "rx-vlan-id": 1999,
  "tx-vlan-id": 2000,
  "vn-id": "38ced7ff-fdfc-4f17-9547-17ff4fb45da9",
  "vm-project-id": "cb33cc2c-6fb8-4072-a112-77547c60d2b7",
  "mac-address": "02:75:3a:65:ef:91",
  "system-name": "753a65ef-918a-4193-9266-dc23ed8a5981"
}
```
#### Example
```
curl -X POST -H "Content-Type: application/json" -d '{"ip-address": "11.0.0.11", "rx-vlan-id": 1999, "display-name": "vm1", "id": "753a65ef-918a-4193-9266-dc23ed8a5982", "instance-id": "63bd1a8a-ba6a-4547-af8d-653dd8ff397e", "ip6-address": "None", "tx-vlan-id": 2000, "vn-id": "38ced7ff-fdfc-4f17-9547-17ff4fb45da9", "vm-project-id": "cb33cc2c-6fb8-4072-a112-77547c60d2b7", "type": 2, "mac-address": "02:75:3a:65:ef:91", "system-name": "753a65ef-918a-4193-9266-dc23ed8a5981"}'  http://localhost:9091/port
```
### Json for Multiple ports creation/updation
```
[{"ip-address": "11.0.0.11", "rx-vlan-id": 1999, "display-name": "vm1", "id": "753a65ef-918a-4193-9266-dc23ed8a5982", "instance-id": "63bd1a8a-ba6a-4547-af8d-653dd8ff397e", "ip6-address": "None", "tx-vlan-id": 2000, "vn-id": "38ced7ff-fdfc-4f17-9547-17ff4fb45da9", "vm-project-id": "cb33cc2c-6fb8-4072-a112-77547c60d2b7", "type": 2, "mac-address": "02:75:3a:65:ef:91", "system-name": "753a65ef-918a-4193-9266-dc23ed8a5981"}, {}]
```
#### Example
```
curl -X POST -H "Content-Type: application/json" -d '[{"ip-address": "11.0.0.11", "rx-vlan-id": 1999, "display-name": "vm1", "id": "753a65ef-918a-4193-9266-dc23ed8a5982", "instance-id": "63bd1a8a-ba6a-4547-af8d-653dd8ff397e", "ip6-address": "None", "tx-vlan-id": 2000, "vn-id": "38ced7ff-fdfc-4f17-9547-17ff4fb45da9", "vm-project-id": "cb33cc2c-6fb8-4072-a112-77547c60d2b7", "type": 2, "mac-address": "02:75:3a:65:ef:91", "system-name": "753a65ef-918a-4193-9266-dc23ed8a5981"}, {"ip-address": "11.0.0.12", "rx-vlan-id": 1999, "display-name": "vm1", "id": "753a65ef-918a-4193-9266-dc23ed8a5983", "instance-id": "63bd1a8a-ba6a-4547-af8d-653dd8ff397f", "ip6-address": "None", "tx-vlan-id": 2000, "vn-id": "38ced7ff-fdfc-4f17-9547-17ff4fb45da9", "vm-project-id": "cb33cc2c-6fb8-4072-a112-77547c60d2b7", "type": 2, "mac-address": "02:75:3a:65:ef:92", "system-name": "753a65ef-918a-4193-9266-dc23ed8a5982"}]'  http://localhost:9091/port
```
# 2 POST for sync of ports

### http://localhost:9091/syncports (POST)
This is used for syncing of ports between plug-in (REST client) and agent across plug-in restarts. Typically, when plug-in (client) restarts, it issues this command and then issues adds for ports using the above (item number 1) API. Agent, upon receiving this command starts a timer. It expects plug-in to replay all ports before the expiry of this timer.  On expiry of timer, agent will remove all ports which have not been added by plug-in after issue of syncports command.

The timeout value of this timer is configurable in contrail-vrouter-agent config file. The configration parameter is stale_interface_cleanup_timeout under DEFAULT section. The value is set in seconds.

#### Example
```
curl -X POST http://localhost:9091/syncports
```
# 3 POST for GET of port <br>
 
#### http://localhost:9091/port/uuid (GET)
#### Example:
```
curl -X GET http://localhost:9091/port/753a65ef-918a-4193-9266-dc23ed8a5982
```

# 4 POST for DELETE of port

#### http://localhost:9091/port/uuid
#### Example
```
curl -X DELETE http://localhost:9091/port/753a65ef-918a-4193-9266-dc23ed8a5982
```
