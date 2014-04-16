The contrail software consists of multiple modules:
* configuration
* analytics
* control plane
* compute node
* web-ui

## Configuration

### Services
* cassandra
* rabbitmq
** If rabbitmq is being used for openstack, we recommend that one uses the same service with a "vhost" for open contrail.

### Processes
* api-server
* schema-transformer
* discovery
* ifmap-server

## Analytics

### Services
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

## Compute node
* vrouter agent
* vrouter kernel module
