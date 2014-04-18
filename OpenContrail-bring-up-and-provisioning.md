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
cassandra_server_list = 127.0.0.1:9160
auth = keystone
multi_tenancy = True
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

- cassandra_server_list is a space separated list in the form: "x.x.x.x:9160 y.y.y.y:9160".
- zk_server_ip is a comma separated list in the form "x.x.x.x:2181,y.y.y.y:2181" and defaults to localhost.

#### schema-transformer
#### discovery
#### ifmap-server

- Load balanced services: api-server (port 8082) and discovery.

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
* control-node
* dns deamon

Recommendation: 2 control-nodes.

## Compute node
* vrouter agent
* vrouter kernel module
* nova vif driver

## Neutron
* neutron opencontrail plugin
Currently distributed as a neutron fork at github.com/Juniper/neutron