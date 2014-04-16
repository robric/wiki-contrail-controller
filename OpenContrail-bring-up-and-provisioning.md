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
* api-server
* schema-transformer
* discovery
* ifmap-server

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