# # Step 0 - Prerequisites

* Ensure on all servers disk on ****all partitions**** are not exhausted.

    ``df -kh``

* Ensure time is synchronized on all servers with ntp.

* Collect output of ``contrail-status`` on all nodes

* While instantiating a VM, say you are using a CirrOS image directly downloaded from web and creating it through the horizon UI i.e. you didn't use glance to create it. Leave the architecture field "**blank**". Cause if you put amd64 / x86-64 for the respective 64-bit image, you can end up in the following issue:
https://ask.openstack.org/en/question/47756/instance-not-supported/