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
* * Create and config a "host-only" network in VirtualBox, if you don't have one (or have just installed VirtualBox).
*** To create a network, go to File -> Preferences -> Network -> "Add host only network", it would usually have a name like vboxnet0 etc. (Windows Only: You don't have to perform the Add host only network" step, move on to step 3.2)
*** To config the network created in step 3.1, right click and select "Edit host-only network", then uncheck "Enable server" in the "DHCP server" tab
*** (Windows only) Start Administrative Tools > Windows Firewall with Advanced Security. Click on the "Windows Firewall Properties" (central panel, possibly quite small print) and for each of the profiles (Domain, Private, Public) click on the Protected Network Connections "Customize..." button and uncheck the "Virtual Box Host only Network" so that the Windows Firewall does not block communications on that network.

### Xen Hypervisor with XCP 1.6 
* Download the DevCloud image for CloudStack 4.3 and above from [here] (http://people.apache.org/~sebgoa/devcloud2.ova)




