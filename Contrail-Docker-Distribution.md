# Contrail Docker Distributions

This Document explains about the distribution TGZ structure and the explains about its contents.  
Using an example release as 4.0.0.0 and Version as 3045.

## Networking Distribution Structure:
#### contrail-networking-docker_4.0.0.0-3045.tgz (1)  
    |---- contrail-docker-images_4.0.0.0-3045.tgz (2)
    |     |---- contrail-controller-u14.04-4.0.0.0-3045.tar.gz
    |     |---- contrail-analytics-u14.04-4.0.0.0-3045.tar.gz
    |     |---- contrail-agent-u14.04-4.0.0.0-3045.tar.gz
    |     |---- contrail-lb-u14.04-4.0.0.0-3045.tar.gz
    |     |---- contrail-analyticsdb-u14.04-4.0.0.0-3045.tar.gz  
    |---- contrail-networking-tools_4.0.0.0-3045.tgz (3)
    |     |---- contrail-docker-tools-4.0.0.0-3045.tar.gz (4)
    |---- contrail-vrouter-packages_4.0.0.0-3045.tgz (5)
    |     |---- contrail-vrouter-agent_4.0.0.0-3045_amd64.deb
    |     |---- contrail-vrouter-dkms_4.0.0.0-3045_amd64.deb
    |     |---- contrail-vrouter-dpdk_4.0.0.0-3045_all.deb
    |     |---- contrail-vrouter-source_4.0.0.0-3045_amd64.deb
    |     |---- contrail-vrouter-utils_4.0.0.0-3045_amd64.deb
    |     |---- python-contrail_4.0.0.0-3045_amd64.deb
    |     |---- python-contrail-vrouter-api_4.0.0.0-3045_amd64.deb
    |     |---- contrail-vrouter-3.13.0-106-generic_4.0.0.0-3045_all.deb
    |     |---- python-opencontrail-vrouter-netns_4.0.0.0-3045_amd64.deb
    |     |---- contrail-utils_4.0.0.0-3045_amd64.deb
    |     |---- contrail-lib_4.0.0.0-3045_amd64.deb
    |     |---- contrail-nova-vif_4.0.0.0-3045_all.deb
    |     |---- contrail-openstack-vrouter_4.0.0.0-3045_all.deb
    |     |---- contrail-setup_4.0.0.0-3045_all.deb
    |     |---- contrail-vrouter-common_4.0.0.0-3045_all.deb
    |     |---- contrail-vrouter-init_4.0.0.0-3045_all.deb
    |     |---- vrouter-openstack-extra_4.0.0.0-3045-mitaka.tgz (6)
    |     |     |---- python-novaclient_2%3a3.3.1-2ubuntu1~cloud0_all.deb
    |     |     |---- ...
    |---- contrail-openstack-networking_4.0.0.0-3045.tgz (7)
    |    |---- contrail-nova-networkapi_4.0.0.0-3049_all.deb  
    |    |---- contrail-heat_4.0.0.0-3049_all.deb  
    |    |---- contrail-openstack_4.0.0.0-3049_all.deb  
    |    |---- contrail-openstack-dashboard_4.0.0.0-3049_all.deb  
    |    |---- python-neutronclient_4.1.1-2~cloud0.2contrail_all.deb  
    |---- contrail-neutron-plugin-packages_4.0.0.0-3045.tgz (8)
    |     |---- neutron-plugin-contrail_4.0.0.0-3045_all.deb
    |     |---- python-contrail_4.0.0.0-3048_amd64.deb
    |     |---- neutron-plugin-contrail-openstack-extra_4.0.0.0-3045-mitaka.tgz (9)
    |     |     |---- python-neutronclient_4.1.1-2~cloud0.2contrail_all.deb
    |     |     |---- ...
    |---- contrail-networking-thirdparty_4.0.0.0-3045.tgz (10)
    |     |---- docker-engine_1.13.0-0~ubuntu-trusty_amd64.deb
    |     |---- ansible_2.2.0.0-1ppa~trusty_all.deb
    |     |---- ...  
    |---- contrail-networking-dependents_4.0.0.0-3045.tgz (11)
    |     |---- git-man_1%3a1.9.1-1ubuntu0.3_all.deb
    |     |---- libogg0_1.3.1-1ubuntu1_amd64.deb
    |     |---- ...
    |     |---- ...
    |     |---- ...
    |     |---- gcc_4%3a4.8.2-1ubuntu6_amd64.deb
    |     |---- libmpc3_1.0.1-1ubuntu1_amd64.deb
    |     |---- python-decorator_3.4.0-2build1_all.deb  


## Cloud Distribution Structure:
#### contrail-cloud-docker_4.0.0.0-3045-mitaka.tgz (12)
    |---- contrail-cloud-tools-4.0.0.0-3048-mitaka.tar.gz (13)
    |     |---- contrail-puppet-4.0.0.0-3048-mitaka.tar.gz (14)
    |     |---- contrail-ansible-4.0.0.0-3048.tgz (15)
    |---- contrail-networking-docker_4.0.0.0-3045.tgz (1)
    |---- contrail-cloud-thirdparty_4.0.0.0-3045.tgz (16)
    |     |---- docker-engine_1.13.0-0~ubuntu-trusty_amd64.deb
    |     |---- ansible_2.2.0.0-1ppa~trusty_all.deb
    |     |---- ...
    |---- contrail-openstack-packages_4.0.0.0-3045-mitaka.tgz (17)
    |     |---- python-neutron_2%3a8.3.0-0ubuntu1.1~cloud0_all.deb
    |     |---- python-nova_13.0.0-0ubuntu2~cloud0.1contrail1_all.deb
    |     |---- nova-api_13.0.0-0ubuntu2~cloud0.1contrail1_all.deb
    |     |---- ...
    |---- contrail-cloud-dependents_4.0.0.0-3045.tgz (18)
    |     |---- git-man_1%3a1.9.1-1ubuntu0.3_all.deb
    |     |---- libogg0_1.3.1-1ubuntu1_amd64.deb
    |     |---- ...
    |     |---- ...
    |     |---- ...
    |     |---- gcc_4%3a4.8.2-1ubuntu6_amd64.deb
    |     |---- libmpc3_1.0.1-1ubuntu1_amd64.deb
    |     |---- python-decorator_3.4.0-2build1_all.deb


#### 1) contrail-docker-networking_4.0.0.0-3045.tgz:  
Wrapper TGZ Containing below tgzs  
* contrail-vrouter-packages_4.0.0.0-3045.tgz
* contrail-neutron-plugin-packages_4.0.0.0-3045.tgz
* contrail-docker-images_4.0.0.0-3045.tgz
* contrail-networking-dependencies_4.0.0.0-3045.tgz
* contrail-networking-thirdparty_4.0.0.0-3045.tgz
* vrouter-openstack-extra_4.0.0.0-3045-mitaka.tgz
* neutron-plugin-contrail-openstack-extra_4.0.0.0-3045-mitaka.tgz

#### 2) contrail-docker-images_4.0.0.0-3045.tgz   
Contains below docker images + ansible code used by SM  
contrail-controller-u14.04-4.0.0.0-3045.tar.gz  
contrail-analytics-u14.04-4.0.0.0-3045.tar.gz  
contrail-agent-u14.04-4.0.0.0-3045.tar.gz   
contrail-lb-u14.04-4.0.0.0-3045.tar.gz  
contrail-analyticsdb-u14.04-4.0.0.0-3045.tar.gz  

#### 3) contrail-networking-tools_4.0.0.0-3045.tgz  
Contrail tools used to provision docker, vrouter etc  
Contains  
* contrail-ansible-4.0.0.0-3045.tar.gz  

#### 4) contrail-docker-tools-4.0.0.0-3045.tar.gz
Contains helper and issue related scripts.

#### 5) contrail-vrouter-packages_4.0.0.0-3045.tgz  
Contains contrail packages required to install below provided list of packages  
* contrail-vrouter-3.13.0-106-generic  (Based on Recommended Kernel version)  
* contrail-vrouter-dkms  
* contrail-vrouter-dpdk  
* contrail-vrouter-dpdk-init  
* contrail-openstack-vrouter  

#### 6) vrouter-openstack-extra_4.0.0.0-3045-mitaka.tgz  
Dependent packages from Openstack Repo which are required to install contrail vrouter role. 
This TGZ is tagged with SKU name as it contains packages from relevant SKU. In this case, it contains packages from Openstack Mitaka repo  
Note: In Networking distribution, these packages are expected to be provided by User and this TGZ serves as a reference. 


#### 7) contrail-openstack-networking_4.0.0.0-3045.tgz
Openstack packages created or rebuilt by contrail and are required to install contrail-openstack role  
Contains below listed packages  
* contrail-nova-networkapi_4.0.0.0-3049_all.deb  
* contrail-heat_4.0.0.0-3049_all.deb  
* contrail-openstack_4.0.0.0-3049_all.deb  
* contrail-openstack-dashboard_4.0.0.0-3049_all.deb  
* python-neutronclient_4.1.1-2~cloud0.2contrail_all.deb  

#### 8) contrail-neutron-plugin-packages_4.0.0.0-3045.tgz
Contains contrail packages required to install contrail neutron plugin role

#### 9) neutron-plugin-contrail-openstack-extra_4.0.0.0-3045-mitaka.tgz  
Dependent packages from Openstack Repo which are required to install contrail neutron plugin role.  
This TGZ is tagged with SKU name as it contains packages from relevant SKU. In this case, it contains packages from Openstack Mitaka repo  
Note: In Networking distribution, these packages are expected to be provided by User and this TGZ serves as a reference. 

#### 10) contrail-networking-thirdparty_4.0.0.0-3045.tgz  
Thirparty packages required to install contrail vrouter + contrail neutron plugin  

#### 11) contrail-networking-dependents_4.0.0.0-3045.tgz
Dependent packages from Ubuntu upstream repos (trusty, trusty-updates, trusty-security) required for installing  
1) contrail vrouter  
2) neutron plugin    

It will also have upstream packages required for below   
1. Kernel upgrade dependents  
2. Docker-Engine dependents  
3. Ansible dependents  


#### 12) contrail-docker-cloud_4.0.0.0-3045-mitaka.tgz  
Wrapper TGZ containing  
contrail-puppet-4.0.0.0-3048-mitaka.tar.gz  
contrail-docker-networking_4.0.0.0-3045.tgz  
contrail-cloud-dependencies_4.0.0.0-3045.tgz  
contrail-cloud-thirdparty_4.0.0.0-3045.tgz  
contrail-openstack-packages_4.0.0.0-3045-mitaka.tgz  
This TGZ is tagged with SKU name to represent it's a Mitaka distribution  

#### 13) contrail-cloud-tools-4.0.0.0-3048-mitaka.tar.gz 
Contrail tools required to provision Docker, Vrouter, Openstack role etc...  
Contains  
contrail-networking-tools-4.0.0.0-3048-mitaka.tar.gz  
contrail-puppet-4.0.0.0-3048-mitaka.tar.gz  

#### 14) contrail-puppet-4.0.0.0-3048-mitaka.tar.gz  
Puppet Code used by SM to provision Openstack Role  
This TGZ is tagged with SKU name as it contains code for relevant SKU. In this case, it contains puppet code required to bring Openstack Mitaka

#### 15) contrail-ansible-4.0.0.0-3045.tar.gz
Contains External ansible code used by SM to trigger docker/node role provisioning.

#### 16) contrail-cloud-thirdparty_4.0.0.0-3045.tgz  
Thirparty packages required to install contrail vrouter + contrail neutron plugin + Openstack role  

#### 17) contrail-openstack-packages_4.0.0.0-3045-mitaka.tgz  
Packages from Openstack repo needed to install Openstack role  
This TGZ is tagged with SKU name as it contains packages from relevant SKU. In this case, it contains packages from Openstack Mitaka repo  

#### 18) contrail-cloud-dependents_4.0.0.0-3045.tgz  
Dependent packages from Ubuntu upstream repos (trusty, trusty-updates, trusty-security) required for installing  
1) contrail vrouter   
2) neutron plugin  
3) Openstack role   

It will also have upstream packages required for below  
1. Kernel upgrade dependents  
2. Docker-Engine dependents  
3. Ansible dependents  
