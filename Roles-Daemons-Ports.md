## The following logical roles/nodes exist in Contrail Architecture:
* Openstack Node

  This node runs the non-neutron Openstack services (eg. nova-api, nova-scheduler, nova-conductor, glance-api etc.)

* Database Node

  Every instance of a database node runs the following processes:
  + cassandra
  + zookeeper (in future)

* Config Node

  Every instance of a config node runs the following processes:
  + contrail-discovery
  + neutron-server
  + contrail-api
  + ifmap
  + contrail-schema
  + contrail-svc-monitor
  + rabbitmq-server (this can optionally be located on an external server)
  + zookeeper (will move to database node)

* Control Node

  Every instance of a control node runs the following processes:
  + control-node
  + contrail-dns
  + contrail-named

* Compute Node

  Every instance of a compute node runs the following processes:
  + nova-compute
  + contrail-vrouter

* Analytics Node

  Every instance of a analytics node runs the following processes:
  + contrail-collector
  + contrail-analytics-api
  + contrail-query-engine (query engine)

* WebUI Node

  Every instance of a Web UI node runs the following processes:
  + contrail-webui
  + contrail-webui-middleware
  + redis-webui

The sections below mention the functionality of the processes in each node and how their connectivity.

### Openstack Role
#### Nova
##### API
##### Scheduler
##### Conductor

#### Glance
##### API
##### Registry

#### Keystone
+ **ports**
  * 5000 - tenant port
  * 35357 - administrative port

#### RabbitMQ
+ **Ports**
  * 5672

#### MySQL
+ **Ports**
  * 3306

### Configuration Node
#### Neutron Server

This process serves the Openstack Networking(neutron) API. It connects to contrail-api server and uses that as the backend for serving neutron API. In works in Active/Active mode in multi-node deployments. 

+ **Service Name** - neutron-server
+ **Ports**
  * 9696 - public port

#### API server

This process exposes a REST-based interface for Contrail API. It also:

  + connects to Cassandra on database node(s) for persistence.
  + publishes to IFMAP server for consumption of config information by other nodes.
  + uses AMQP/rabbitmq to communicate with other API servers in multi-node deployment.

+ **Service Name** - contrail-api
+ **Ports**
  * 8082 - public port (accessible with credentials in auth mode)
  * 8095 - debug port (accessible only on localhost in auth mode)
  * 8084 - Introspect port

#### IFMAP server

This process acts as a bulletin-board for pubsub purposes. contrail-control, contrail-schema, contrail-svc-monitor connect to this server and act on configuration change. All the publish to this happens from API server.

+ **Ports**
  * 8443 - basic auth port

#### Schema Transformer

This process listens to configuration changes done by user (higher-level constructs eg. VirtualNetwork) and generates appropriate system configuration objects (lower-level construct eg. RoutingInstance). It uses the REST interface of API server to manipulate the system objects. In multi-node deployments it works in Active/Backup mode.

+ **Service Name** - contrail-schema
+ **Ports**
  * 8087 - Introspect port

#### Service Monitor

This process listens to configuration changes of service-template and service-instance and spawns and monitors virtual machines for services like firewall, analyzer etc. It uses nova API to spawn the virtual machines. In multi-node deployments it works in Active/Backup mode.

+ **Service Name** - contrail-svc-monitor
+ **Ports**
  * 8088 - Introspect port

#### Discovery Server

This process acts a registry for all contrail services. It exposes a REST API that contrail processes use to publish the service they are offering and query for services they need. For eg. control-node publishes to discovery and queries for collectors to connect to.

+ **Service Name** - contrail-discovery
+ **Ports**
  * 5998 - public port

### Analytics Node
#### Analytics REST API Server
+ **Service Name** - contrail-analytics-api
+ **Ports**
  * 8081 - public port

### Collector
+ **Service Name** - contrail-collector
+ **Ports**
  * 8086 - public port

### Query Engine

## WEBUI Role
### webui
+ **Service Name** - contrail-webui
+ **ports**
  * 8080 - http port redirects to https
  * 8143 - https port
### webui-middleware
+ **Service Name** - contrail-webui-middleware

## Database Role
### cassandra
+ **Service Name** - contrail-database
+ **ports**
  * 9160 - thrift port

## Control Role
### control-node
+ **Service Name** - contrail-control
+ **ports**
  * 8083 - introspect
  * 5269 - XMPP port
### dnsd
+ **Service Name** - contrail-dns
+ **ports**
  * 8093 - DNS service

## VRouter Role
### VRouter agent
+ **Service Name** - contrail-vrouter
+ **Ports**
  * 8085
