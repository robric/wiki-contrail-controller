## Overview
With release 2.2 of Contrail Cloud, OpenStack Ceilometer is installed and enabled by default on the following OpenStack release and OS combination when the Contrail Cloud is installed and provisioned using Fabric:

1. OpenStack release icehouse on Ubuntu 12.04.3 LTS, Ubuntu 14.04.1 LTS, and Redhat Enterprise Linux (RHEL) Server 7.0
2. OpenStack release juno on Ubuntu 14.04.1 LTS, and Redhat Enterprise Linux (RHEL) Server 7.0

## Ceilometer Details 
As per the OpenStack Wiki, Ceilometer is used to reliably collect measurements of the utilization of the physical and virtual resources comprising deployed clouds, persist these data for subsequent retrieval and analysis, and trigger actions when defined criteria are met. For more information about Ceilometer, please check the official [OpenStack Ceilometer Wiki](https://wiki.openstack.org/wiki/Ceilometer).

The Ceilometer architecture consists of:

1. **Polling agent** which is designed to poll OpenStack services and build Meters. 
2. **Notification agent** which is designed to listen to notifications on message queue and convert them to Events and Samples.
3. **Collector** which is designed to gather and record event and metering data created by notification and polling agents.
4. **API server** that provides a REST API to query and view data recorded by collector service.
5. **Alarming** - Daemons to evaluate and notify based on defined alarming rules.
6. **Database** to store the metering data, notifications, and alarms. The supported databases are MongoDB, SQL-based databases compatible with SQLAlchemy, HBase. However, MongoDB is recommended and it is the backend that has been thoroughly tested and deployed on a production scale.

For more information about Ceilometer architecture, please check the official [Ceilometer Architecture Docs](http://docs.openstack.org/developer/ceilometer/architecture.html)

## Verification of Ceilometer operation
The Ceilometer services are named slightly differently on Ubuntu and RHEL Server 7.0.

On Ubuntu, the service names are:

1. **Polling agent** - `ceilometer-agent-central`
2. **Notification agent** - `ceilometer-agent-notification`
3. **Collector** - `ceilometer-collector`
4. **API server** - `ceilometer-api`
5. **Alarming** - `ceilometer-alarm-evaluator` and `ceilometer-alarm-notifier`

On RHEL Server 7.0, the service names are:

1. **Polling agent** - `openstack-ceilometer-central`
2. **Notification agent** - `openstack-ceilometer-notification`
3. **Collector** - `openstack-ceilometer-collector`
4. **API server** - `openstack-ceilometer-api`
5. **Alarming** - `openstack-ceilometer-alarm-evaluator` and `openstack-ceilometer-alarm-notifier`


To verify the Ceilometer installation, users can verify that the Ceilometer services are up and running using the `openstack-status` command. For example, running `openstack-status` on an all-in-one node running Ubuntu 14.04.1 LTS with release 2.2 of Contrail Cloud installed will show the Ceilometer services as active: 

    == Ceilometer services ==
    ceilometer-api:               active
    ceilometer-agent-central:     active
    ceilometer-agent-compute:     active
    ceilometer-collector:         active
    ceilometer-alarm-notifier:    active
    ceilometer-alarm-evaluator:   active
    ceilometer-agent-notification:active



Further, users can issue the ceilometer meter-list command on the OpenStack controller node to verify that meters are being collected, stored, and reported via the REST API. 

    root@a7s37:~# (source /etc/contrail/openstackrc; ceilometer meter-list)
    +------------------------------+------------+---------+--------------------------------------+----------------------------------+----------------------------------+
    | Name                         | Type       | Unit    | Resource ID                          | User ID                          | Project ID                       |
    +------------------------------+------------+---------+--------------------------------------+----------------------------------+----------------------------------+
    | ip.floating.receive.bytes    | cumulative | B       | a726f93a-65fa-4cad-828b-54dbfcf4a119 | None                             | None                             |
    | ip.floating.receive.packets  | cumulative | packet  | a726f93a-65fa-4cad-828b-54dbfcf4a119 | None                             | None                             |
    | ip.floating.transmit.bytes   | cumulative | B       | a726f93a-65fa-4cad-828b-54dbfcf4a119 | None                             | None                             |
    | ip.floating.transmit.packets | cumulative | packet  | a726f93a-65fa-4cad-828b-54dbfcf4a119 | None                             | None                             |
    | network                      | gauge      | network | 7fa6796b-756e-4320-9e73-87d4c52ecc83 | 15c0240142084d16b3127d6f844adbd9 | ded208991de34fe4bb7dd725097f1c7e |
    | network                      | gauge      | network | 9408e287-d3e7-41e2-89f0-5c691c9ca450 | 15c0240142084d16b3127d6f844adbd9 | ded208991de34fe4bb7dd725097f1c7e |
    | network                      | gauge      | network | b3b72b98-f61e-4e1f-9a9b-84f4f3ddec0b | 15c0240142084d16b3127d6f844adbd9 | ded208991de34fe4bb7dd725097f1c7e |
    | network                      | gauge      | network | cb829abd-e6a3-42e9-a82f-0742db55d329 | 15c0240142084d16b3127d6f844adbd9 | ded208991de34fe4bb7dd725097f1c7e |
    | network.create               | delta      | network | 7fa6796b-756e-4320-9e73-87d4c52ecc83 | 15c0240142084d16b3127d6f844adbd9 | ded208991de34fe4bb7dd725097f1c7e |
    | network.create               | delta      | network | 9408e287-d3e7-41e2-89f0-5c691c9ca450 | 15c0240142084d16b3127d6f844adbd9 | ded208991de34fe4bb7dd725097f1c7e |
    | network.create               | delta      | network | b3b72b98-f61e-4e1f-9a9b-84f4f3ddec0b | 15c0240142084d16b3127d6f844adbd9 | ded208991de34fe4bb7dd725097f1c7e |
    | network.create               | delta      | network | cb829abd-e6a3-42e9-a82f-0742db55d329 | 15c0240142084d16b3127d6f844adbd9 | ded208991de34fe4bb7dd725097f1c7e |
    | port                         | gauge      | port    | 0d401d96-c2bf-4672-abf2-880eecf25ceb | 01edcedd989f43b3a2d6121d424b254d | 82ab961f88994e168217ddd746fdd826 |
    | port                         | gauge      | port    | 211b94a4-581d-45d0-8710-c6c69df15709 | 01edcedd989f43b3a2d6121d424b254d | 82ab961f88994e168217ddd746fdd826 |
    | port                         | gauge      | port    | 2287ce25-4eef-4212-b77f-3cf590943d36 | 01edcedd989f43b3a2d6121d424b254d | 82ab961f88994e168217ddd746fdd826 |
    | port.create                  | delta      | port    | f62f3732-222e-4c40-8783-5bcbc1fd6a1c | 01edcedd989f43b3a2d6121d424b254d | 82ab961f88994e168217ddd746fdd826 |
    | port.create                  | delta      | port    | f8c89218-3cad-48e2-8bd8-46c1bc33e752 | 01edcedd989f43b3a2d6121d424b254d | 82ab961f88994e168217ddd746fdd826 |
    | port.update                  | delta      | port    | 43ed422d-b073-489f-877f-515a3cc0b8c4 | 15c0240142084d16b3127d6f844adbd9 | ded208991de34fe4bb7dd725097f1c7e |
    | subnet                       | gauge      | subnet  | 09105ed1-1654-4b5f-8c12-f0f2666fa304 | 15c0240142084d16b3127d6f844adbd9 | ded208991de34fe4bb7dd725097f1c7e |
    | subnet                       | gauge      | subnet  | 4bf00aac-407c-4266-a048-6ff52721ad82 | 15c0240142084d16b3127d6f844adbd9 | ded208991de34fe4bb7dd725097f1c7e |
    | subnet.create                | delta      | subnet  | 09105ed1-1654-4b5f-8c12-f0f2666fa304 | 15c0240142084d16b3127d6f844adbd9 | ded208991de34fe4bb7dd725097f1c7e |
    | subnet.create                | delta      | subnet  | 4bf00aac-407c-4266-a048-6ff52721ad82 | 15c0240142084d16b3127d6f844adbd9 | ded208991de34fe4bb7dd725097f1c7e |
    +------------------------------+------------+---------+--------------------------------------+----------------------------------+----------------------------------+

**Note:** The `ceilometer meter-list` command will list the meters only if images have been created and/or instances launched and/or subnet/port/floating IPs created, otherwise it will be empty. Users also need to source the `/etc/contrail/openstackrc` file when executing the command.

## Contrail Ceilometer Plugin

The Contrail Ceilometer Plugin adds capability to meter the traffic statistics of floating IPs in Ceilometer. It adds the meters `ip.floating.receive.bytes`, `ip.floating.receive.packets`, `ip.floating.transmit.bytes`, `ip.floating.transmit.packets` for each floating IP resource in Ceilometer. 

The Contrail Ceilometer Plugin configuration is done in the /etc/ceilometer/pipeline.yaml automatically when Contrail Cloud is installed.


