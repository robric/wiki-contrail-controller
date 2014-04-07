## DevCloud Components
There are three components that needs to be installed and provisioned to make DevCloud with Contrail work.
* Management Server - CloudStack (running in the Host machine)
* Compute Node - Xen with XCP and Contrail Networking component (DevCloud image running as a VM)
* Contrail Control Node (Ubuntu 12.04 VM)

For more details on the architecture and how these components interact together, visit http://opencontrail.org/getting-started/

## Preparation
### CloudStack Dev Environment
In your host machine, use the instructions [here](https://cwiki.apache.org/confluence/display/CLOUDSTACK/Setting+up+CloudStack+Development+Environment) to setup a CloudStack development environment.

### Setting up VirtualBox 
* Install VirtualBox for your environment from [here](https://www.virtualbox.org/wiki/Downloads).
* Create and config a "host-only" network in VirtualBox, if you don't have one (or have just installed VirtualBox).
    * To create a network, go to File -> Preferences -> Network -> "Add host only network", it would usually have a name like vboxnet0 etc. (Windows Only: You don't have to perform the Add host only network" step, move on to step 3.2)
    * To config the network created in step 3.1, right click and select "Edit host-only network", then uncheck "Enable server" in the "DHCP server" tab
    *(Windows only) Start Administrative Tools > Windows Firewall with Advanced Security. Click on the "Windows Firewall Properties" (central panel, possibly quite small print) and for each of the profiles (Domain, Private, Public) click on the Protected Network Connections "Customize..." button and uncheck the "Virtual Box Host only Network" so that the Windows Firewall does not block communications on that network.

### Xen Hypervisor with XCP 1.6 
* Download the DevCloud image for CloudStack 4.3 and above from [here] (http://people.apache.org/~sebgoa/devcloud2.ova)
* Import the image into VirtualBox. After VM boots up, login into VM with username: root, password: password

### Contrail Control Node
* Download any of the Ubuntu VirtualBox images from [here] (http://virtualboximages.com/Ubuntu+12.04+amd64+LAMP/Tomcat+Server+Virtual+Appliance).
* Create two network interfaces for the VM.
    * Host-only network interface
    * NAT network interface

## Building the code from Source
### Cloudstack
* [Checkout](https://cwiki.apache.org/confluence/display/CLOUDSTACK/Getting+the+Source+Code) the latest master code
* Start the management server on your host machine (laptop)

***

`mvn -P developer,systemvm clean install`

***

* Create a file with name `contrail.properties` in path `client/target/generated-webapp/WEB-INF/classes/contrail.properties` and add the following text into it.

`api.hostname = 192.168.56.30
api.port = 8082`


***

`mvn -P developer -pl developer,tools/devcloud -Ddeploydb`

`mvn -pl :cloud-client-ui jetty:run`

***

Wait till the management server is up and running.

