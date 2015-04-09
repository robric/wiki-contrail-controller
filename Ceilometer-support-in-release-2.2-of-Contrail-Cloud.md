# Overview
With release 2.2 of Contrail Cloud, OpenStack Ceilometer is installed and enabled by default on the following OpenStack release and OS combination when the Contrail Cloud is installed and provisioned using Fabric:

1. OpenStack release icehouse on Ubuntu 12.04.3 LTS, Ubuntu 14.04.1 LTS, and Redhat Enterprise Linux Server 7.0
2. OpenStack release juno on Ubuntu 14.04.1 LTS

# Ceilometer Details 
As per the OpenStack Wiki, Ceilometer is used to reliably collect measurements of the utilization of the physical and virtual resources comprising deployed clouds, persist these data for subsequent retrieval and analysis, and trigger actions when defined criteria are met. For more information about Ceilometer, please check the official [OpenStack Ceilometer Wiki](https://wiki.openstack.org/wiki/Ceilometer).

The Ceilometer architecture consists of:

1. **Polling agent** which is designed to poll OpenStack services and build Meters. 
2. **Notification agent** which is designed to listen to notifications on message queue and convert them to Events and Samples.
3. **Collector** which is designed to gather and record event and metering data created by notification and polling agents.
4. **API server** that provides a REST API to query and view data recorded by collector service.
5. **Alarming** - Daemons to evaluate and notify based on defined alarming rules.
6. **Database** to store the metering data, notifications, and alarms. The supported databases are MongoDB, SQL-based databases compatible with SQLAlchemy, HBase. However, MongoDB is recommended and it is the backend that has been thoroughly tested and deployed on a production scale.

For more information about Ceilometer architecture, please check the official [Ceilometer Architecture Docs](http://docs.openstack.org/developer/ceilometer/architecture.html)

# Verification of Ceilometer operation
The Ceilometer services are named slightly differently on Ubuntu and RHEL Server 7.0.

On Ubuntu, the service names are:

1. _Polling agent_ - `ceilometer-agent-central`
2. _Notification agent_ - `ceilometer-agent-notification`
3. _Collector_ - `ceilometer-collector`
4. _API server_ - `ceilometer-api`
5. _Alarming_ - `ceilometer-alarm-evaluator` and `ceilometer-alarm-notifier`

On RHEL Server 7.0, the service names are:

1. _Polling agent_ - `openstack-ceilometer-central`
2. _Notification agent_ - `openstack-ceilometer-notification`
3. _Collector_ - `openstack-ceilometer-collector`
4. _API server_ - `openstack-ceilometer-api`
5. _Alarming_ - `openstack-ceilometer-alarm-evaluator` and `openstack-ceilometer-alarm-notifier`


To verify the Ceilometer installation, users can verify that the Ceilometer processes are up and running using the `openstack-status` command. For example, running `openstack-status` on an all-in-one node running Ubuntu 14.04.1 LTS with release 2.2 of Contrail Cloud installed will give the following output: 

