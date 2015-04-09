With release 2.2 of Contrail Cloud, OpenStack Ceilometer is installed and enabled by default on the icehouse and juno OpenStack releases on Ubuntu 12.04.3 LTS, Ubuntu 14.04.1 LTS, and Redhat Enterprise Linux Server 7.0 when using Fabric. 

As per the OpenStack Wiki, Ceilometer is used to reliably collect measurements of the utilization of the physical and virtual resources comprising deployed clouds, persist these data for subsequent retrieval and analysis, and trigger actions when defined criteria are met. For more information about Ceilometer, please check the official [OpenStack Ceilometer Wiki](https://wiki.openstack.org/wiki/Ceilometer).

The Ceilometer architecture consists of:
1. Polling agent which is designed to poll OpenStack services and build Meters. 
2. Notification agent which is designed to listen to notifications on message queue and convert them to Events and Samples.
3. Collector which is designed to gather and record event and metering data created by notification and polling agents.
4. API server that provides a REST API to query and view data recorded by collector service.
5. Alarming - Daemons to evaluate and notify based on defined alarming rules.
For more information about Ceilometer architecture, please check the official [Ceilometer Architecture Docs](http://docs.openstack.org/developer/ceilometer/architecture.html)

The above services are named slightly differently on Ubuntu and RHEL Server 7.0.

On Ubuntu, the service names are:
1. Polling agent - ceilometer-agent-central
2. Notification agent - ceilometer-agent-notification
3. Collector - ceilometer-collector
4. API server - ceilometer-api
5. Alarming - ceilometer-alarm-evaluator and ceilometer-alarm-notifier

On RHEL Server 7.0, the service names are:



1. An API server (called ceilometer-api on Ubuntu and openstack-ceilometer-api on RHEL Server 7.0) providing REST API to access the metering data stored in the database. 
2. A central agent (called ceilometer-agent-central on Ubuntu and openstack-ceilometer-central on RHEL Server 7.0) that polls utilization statistics for resources not tied to instances or compute nodes. 
3. A compute agent (called ceilometer-agent-compute on Ubuntu and openstack-ceilometer-compute on RHEL Server 7.0) that polls metering data and instances statistics from the compute nodes.
4. A collector (called ceilometer-collector on Ubuntu and openstack-ceilometer-collector on RHEL Server 7.0) that monitors the message queues for notifications and statistics from the agents and stores them in the database.
5. A database for storing the metering data and 

These services communicate using the standard OpenStack messaging bus. Only the collector and API server have access to the data store. The supported databases are MongoDB, SQL-based databases compatible with SQLAlchemy, and HBase; however, Ceilometer developers recommend MongoDB due to its processing of concurrent read/writes. In addition, only the MongoDB backend has been thoroughly tested and deployed on a production scale

2.  running agents on the compute nodes and a collector and other server daemons on the OpenStack controller node.

To verify the Ceilometer installation, users can verify that the Ceilometer processes are up and running using the `openstack-status` command. For example, running `openstack-status` on an all-in-one node running Ubuntu 14.04.1 LTS with release 2.2 of Contrail Cloud installed will give the following output: 