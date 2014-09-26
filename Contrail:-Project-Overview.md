Contrail Project Overview
-------------------------

OpenContrail is community supported Apache project to provide network virtualization functionality on Openstack and other orchestration systems. Currently, this is primarily supported on Openstack. The code is available publically on GitHub and project is hosted on launchpad.
This document is meant to be the overview of the project to developers and users new to the community.

This document covers following topics:

* Launchpad projects: OpenContrail vs JuniperOpenstack
* Project Management
* Developer resources.
* FAQs

- - -

####OpenContrail vs JuniperOpenstack ####

OpenContrail (or Contrail) provides virtual network functionality based on open protocols and Rest APIs. It consists of following major components (aka roles): 

* Virtual Network Controller (VNC): SDN controller based on BGP/L3VPN.
* Configuration Node: Manages contrail configuration and exposes north bound REST APIs (VNC API). Neutron v2 APIs are also supported.
* vRouter: Implements dataplane forwarding LKM and user space 'Agent' to manage forwarding.
* Analytics: Collects logs, statistics, flow data from all nodes. These data can be queried via REST APIs.
* UI: Contrail UI implements configuration and monitoring system based on standard APIs supported by Configuration and Analytics node.

OpenContrail can be integrated with Openstack and other orchestration systems. Currently this is actively maintained for Openstack. Beta level support is available for Cloudstack.

JuniperOpenstack is a distribution of OpenStack by Juniper that is used by Juniper Private Cloud, Juniper's NFV Solution for Service Provider Market, OpenContrail build system, and OpenContrail test system. In addition to Contrail networking this includes following components:

* Block and Object storage.
* Installation, provisioning and monitoring of cluster.
* Contrail extension to openstack componenets such as Horizon, Neutron Client etc.
* High availability for Openstack.
* Other features to ease the deployment and operation of cloud.

JuniperOpenstack is based on Ubuntu cloud archive openstack packages with fixes/extension as needed. Its bundled with all the dependent packages and installation scripts. Note that Ubuntu Cloud archive has additonal openstack packages which may not be qualified for JuniperOpenstack and so will not be part of the release. That does not mean those package cannot be installed on top of Juniper's distribution.

At this time, CentOS and its variant are not distributed as part of JuniperOpenstack. These are available as Contrail networking product, with the expectation that it will be integrated with third party openstack distributions. More specifically, components that are part of JuniperOpenstack (eg. storage, Openstack HA, and Server Manager) are not supported for CentOS target. Having said that we maintain exact feature and performance parity on CentOS wrt Contrail networking features.

More details on the release content of JuniperOpenstack can be found at [http://techwiki.juniper.net](http://http://techwiki.juniper.net/Documentation/Contrail).

- - -

####Developer Resources ####

###### Code Repositories: ######
Entire Contrail code, test scripts and features available as part of JuniperOpenstack are available in public at [GitHub](https://github.com/Juniper?query=). The developers are recommended to use android repo tool to create the development environment. The manifest files for OpenContrail is published at https://github.com/Juniper/contrail-vnc. For more information on building the OpenContrail packages refer to this [document](http://juniper.github.io/contrail-vnc/README.html).

Note: All the code is distributed with Apache 2.0 license.

###### Submitting patches ######

Developers can upstream their patches via OpenContrail CI process. The tools and workflow is similar to that in Openstack project. To get started:
1. Sign the [contributor agreement](https://secure.echosign.com/public/hostedForm?formid=6G36BHPX974EXY)
2. Have a Launchpad ID

After that one can submit the patch to [review.opencontrail.org](https://review.opencontrail.org) by following the [CI document](https://github.com/Juniper/contrail-controller/wiki/OpenContrail-Continuous-Integration-(CI). The CI tool enables code review and runs test script. After passing these steps code will be merged automatically to mainline.

###### Documentation ######
Here are the pointers to Contrail documents:
* Contrail architecture document is at [http://opencontrail.org/ebook/](http://opencontrail.org/ebook/)
* JuniperOpenstack user guide & release notes are hosted on [techwiki](http://techwiki.juniper.net/Documentation/Contrail)
* Developer contributed wiki on Contrail internals, installations etc are hosted on [GitHub](https://github.com/Juniper/contrail-controller/wiki)
* Community contributed blogs on Contrail features & capabilities are at [http://opencontrail.org/blog/](http://opencontrail.org/blog/)

###### Support ######
To get support on Contrail, join the mailing list as indicated in [http://opencontrail.org/community/](http://opencontrail.org/community/)

- - -

####Project Management ####

######Official releases: ######
JuniperOpenstack follows continuous integration development model. It has a minor release every 2 months. The schedule and branch information of the project is maintained on [launchpad](https://launchpad.net/juniperopenstack). The top level branch diagram shows 'milestone' and 'series'. Depending on release content we do maintenance release on a branch, as needed. This is updated on launchpad as soon as the plan is in place.

We tag the branch (ie., each repo) with the release tag. e.g., for release 1.06 and 1.10 we tagged the branches with v1.06 and v1.10 respectively. The corresponding packages are available on OpenContrail launchpad PPA as stable opencontrail packages. These packages are available [here](https://launchpad.net/~opencontrail/+archive/ubuntu/ppa). Peridically good working versions are also uploaded from mainline at snapshot area [here](https://launchpad.net/~opencontrail/+archive/ubuntu/snapshots). The installation instruction for OpenContrail packages can be found on this [wiki](https://github.com/Juniper/contrail-controller/wiki/OpenContrail-bring-up-and-provisioning). These packages are community supoprted via mailing lists as indicated above. 

######Bug tracking: ######
Bug trakcing on this project is done publically on Launchpad. Two Launchpad IDs (aka projects) are maintained for this project, [JuniperOpenstack](https://launchpad.net/juniperopenstack) and [OpenContrail](https://launchpad.net/opencontrail). Bugs found during internal testing of JuniperOpenstack, are tracked at JuniperOpenstack project. All the bugs are made public, unless they have proprietary customer information. By default only Juniper Engineering team can create issues on this project. Anyone can add his/her ID to track a particular bug(s). Launchpad will generate email update for any change on these bugs.

'OpenContrail' project account is meant to track community reported bugs. By default these bugs are public. No special priviledge is needed to create a bug on this project. Optionally user can cross link these public bugs to make it dependent on JuniperOpenstack project and vice versa.

There is no restriction w.r.t code commit against bugs on any of these 2 launchpad projects.

- - -

#### FAQs ####

1. Do we have all code distributed as part of JuniperOpenstack in public?

	Yes, including test suites. Juniper developers also develop on the opensource repos like any other developer in the community.
    
2. What tests do you run before code gets merged via Contrail CI?

   Contrail CI is evolving continuously. Currently it does build for CentOS, Ubuntu 12.04 & 14.04. It runs Unit tests on those builds. Then it installs JuniperOpenstack and runs few basic tests covering basic functionality around Virtual network, policies and packet forwarding. For more detail refer to this [document](https://github.com/Juniper/contrail-controller/wiki/OpenContrail-Continuous-Integration-(CI)).

3. I am an application developer. Where can I find documentation on Contrail APIs?

   Contrail API documentation is maintained with the code. They can be found after installation of Contrail at http://<config-node-ip>:8082/documentation/index.html. Analytics API are at http://<analytics-node-ip>:8083/documentation/index.html.
   
4. Why is 'xyz' feature supported on Ubuntu but not on CentOS?

   In general all contrail networking feature are supported on both platforms. Some of features specific to JuniperOpenstack distribution are only qualified on Ubuntu. This is mainly to reduce the test matrix. The missing features can be integrated on CentOS as well starting from Open Source code. An example of such feature is 'Openstack HA'.
   
5. Why did Juniper chose Ubuntu as opposed to CentOS for JuniperOpenstack?

   This is mainly because CentOS kernel version for long time was quite old. After CentOS 7.0 is out this concern is no longer there. So, depending on interest, JuniperOpenstack could be productized on CentOS as well. 
   
6. Why does Juniper distribute OpenStack packages with JuniperOpenstack on CentOS too?

	For CentOS we distribute minimal working openstack pakcages so CentOS version of Contrail could be deployed for POCs. CentOS customers should deploy only the 'Contrail' packages from the bundle. 

6. What are high level content of JuniperOpenstack distribution. How can I get those?

	JuniperOpenstack distribution is available from Juniper. If you are interested in evaluation, pl get in touch with the support team. The bundle includes provisioning scripts, contrail and openstack packages qualified by Juniper. Also, all the dependent package to install these on base OS version (CentOS or Ubuntu) is included. For more details on installation please refer to [http://techwiki.juniper.net](http://techwiki.juniper.net/Documentation/Contrail).
    
7. Can I only deploy Contrail packages from JuniperOpenstack bundle?

     Yes, abosoltely. Many of our customer have live deployment on Contrail packages. Customers are free to chose alternate solutions/sources for non networking features included in JuniperOpenstack.

8. Do you support devstack?

      Yes. To install devstack follow the guide here. TBD.

9. How closely do you track Upstream Openstack?

      We support upstream Openstack on JuniperOpenstack release within 6 weeks of upstream release. Contrail networking support is released earlier as beta for customers who want to integrate Contrail with latest openstack sooner.