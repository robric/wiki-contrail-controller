The contrail software consists of multiple modules:
* configuration
* analytics
* control plane
* compute node
* web-ui

## Configuration

### Services
* zookeeper
* cassandra
* rabbitmq
* ntp

Zookeeper: recommend odd number of nodes.

Cassandra: recommend a multi-node cluster configuration.

If rabbitmq is being used for openstack, we recommend that one uses the same service with a "vhost" for open contrail.

Servers running control-node components should be time synchronized.

### Processes
#### api-server

Example: /etc/contrail/contrail-api.conf
```
[DEFAULTS]
log_file = /var/log/contrail/contrail-api.log
ifmap_username = api-server
ifmap_password = api-server
cassandra_server_list = x.x.x.x:9160
auth = keystone
multi_tenancy = True
disc_server_ip = x.x.x.x
zk_server_ip = x.x.x.x:2181
rabbit_server = x.x.x.x
rabbit_password = xxxxxxxxxxxxxxxxxxxx

[KEYSTONE]
auth_host = x.x.x.x
auth_port = 35357
auth_protocol = http
admin_user = neutron
admin_password = xxxxxxxxxxxxxxxxxxxx
admin_token = 
admin_tenant_name = service

```

- disc_server_ip should be the load balancer address. The LB should front-end port 5998 which is served by the discovery process. Only a single discovery server answers requires (master election via zookeeper); defaults to localhost.
- cassandra_server_list is a space separated list in the form: "x.x.x.x:9160 y.y.y.y:9160".
- zk_server_ip is a comma separated list in the form "x.x.x.x:2181,y.y.y.y:2181" and defaults to localhost.

#### schema-transformer

- Example: /etc/contrail/contrail-schema.conf
```
[DEFAULTS]
log_file = /var/log/contrail/contrail-schema.log
cassandra_server_list = x.x.x.x:9160
zk_server_ip = x.x.x.x
disc_server_ip = x.x.x.x

[KEYSTONE]
admin_user = neutron
admin_password = xxxxxxxxxxxxxxxxxxxx
admin_tenant_name = service
```

Parameters should be the same as api-server.conf.

- Example: /etc/contrail/vnc_api_lib.ini
```
[auth]
AUTHN_TYPE = keystone
AUTHN_SERVER=x.x.x.x
AUTHN_PORT = 35357
AUTHN_URL = /v2.0/tokens
```

vnc_api_lib.ini is required in the systems that run schema-transformer and neutron-server plugin. It is accessed from the neutron process.

#### discovery
- Example: /etc/contrail/contrail-discovery.conf
```
[DEFAULTS]
zk_server_ip = x.x.x.x
```
#### ifmap-server
- The ifmap-server works with default config when running on all the nodes that api-server runs; the config examples above assume that.

#### Load balanced services
- api-server (port 8082).
- discovery (port 5998).

#### Diagnostics
```
curl http://api-server-address:8082/projects | python -mjson.tool
```
When multi_tenancy is enabled the http request to the api server requires a keystone auth_token.
The command should return a list of several projects, including the project that contrail creates internally as well as all projects currently visible in keystone tenant-list.

```
http://x.x.x.x:5998/services
```

Displays the services registered in the discovery server. Only one of the discovery servers will answer API requests in a multi node configuration. The others are in standby mode.
The output should show one or more entries for: ApiServer, IfmapServer, Collector and xmpp-server.

## Analytics

### Services
* zookeeper
* cassandra
* redis

### Processes
* collector
* query-engine
* query-api (?)

## Control plane

### Processes
#### control-node
Example: /etc/contrail/control-node.conf
```
[DISCOVERY]
server = x.x.x.x

[IFMAP]
user=control-node-<N>
password=control-node-<N>
```

Where N should be the instance-id (e.g. 1, 2, ...)

* dns deamon

Recommendation: 2 control-nodes.

## Compute node
* vrouter agent
* vrouter kernel module
* nova vif driver

## Neutron
* neutron opencontrail plugin
Currently distributed as a neutron fork at github.com/Juniper/neutron