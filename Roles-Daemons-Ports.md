# Openstack Role
## Nova
### API
### Scheduler
### Conductor

## Glance
### API
### Registry

## Keystone
+ **ports**
  * 5000 - tenant port
  * 35357 - administrative port

## QPID
+ **ports**
  * 5672

# Configuration Role
## Quantum Server
+ **Service Name** - quantum-server
+ **Ports**
  * 9696 - public port

## API server
+ **Service Name** - contrail-api
+ **Ports**
  * 8082 - public port (accessible with credentials in auth mode)
  * 8095 - debug port (accessible without credentials in auth mode)
  * 8084 - sandesh port

## IFMAP server
+ **Ports**
  * 8443 - basic auth port

## Schema Transformer
+ **Service Name** - contrail-schema

## Service Monitor
+ **Service Name** - contrail-svc-monitor

## Discovery Server
+ **Ports**
  * 5998 - public port

# Analytics Role
## OP server
+ **Service Name** -
+ **Ports**
  * 8081 - public port
## Collector
## Query Engine

# WEBUI Role
## webui
+ **Service Name** - contrail-webui
+ **ports**
  * 8080 - http port redirects to https
  * 8143 - https port
## webui-middleware
+ **Service Name** - contrail-webui-middleware

# Database Role
## cassandra
+ **Service Name** - contrail-database
+ **ports**
  * 9160 - thrift port

# Control Role
## control-node
+ **Service Name** - contrail-control
+ **ports**
  * 8083
## dnsd
+ **Service Name** - contrail-dns

# VRouter Role
## VRouter agent
+ **Service Name** - contrail-vrouter
+ **Ports**
  * 8085
