With release 2.2 of Contrail Cloud, OpenStack Ceilometer is installed and enabled by default when installing using Fabric. Ceilometer is used to reliably collect measurements of the utilization of the physical and virtual resources comprising deployed clouds, persist these data for subsequent retrieval and analysis, and trigger actions when defined criteria are met. For more information about Ceilometer please check the official [OpenStack Ceilometer Wiki](https://wiki.openstack.org/wiki/Ceilometer).

The Ceilometer architecture consists of running agents on the compute nodes and a collector and other server daemons on the OpenStack controller node.

To verify the Ceilometer installation, users can verify that the Ceilometer processes are up and running using the `openstack-status` command. For example, running `openstack-status` on an all-in-one node running Ubuntu 14.04.1 LTS with release 2.2 of Contrail Cloud installed will give the following output: 

