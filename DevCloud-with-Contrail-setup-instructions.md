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
* Install VirtualBox for your host environment from [here](https://www.virtualbox.org/wiki/Downloads).
* Create and config a "host-only" network in VirtualBox, if you don't have one (or have just installed VirtualBox).
    * To create a network, go to File -> Preferences -> Network -> "Add host only network", it would usually have a name like vboxnet0 etc. (Windows Only: You don't have to perform the Add host only network" step, move on to step 3.2)
    * To config the network created in step 3.1, right click and select "Edit host-only network", then uncheck "Enable server" in the "DHCP server" tab
    *(Windows only) Start Administrative Tools > Windows Firewall with Advanced Security. Click on the "Windows Firewall Properties" (central panel, possibly quite small print) and for each of the profiles (Domain, Private, Public) click on the Protected Network Connections "Customize..." button and uncheck the "Virtual Box Host only Network" so that the Windows Firewall does not block communications on that network.

### Xen Hypervisor with XCP 1.6 
* Download the DevCloud image for CloudStack 4.3 and above from [here] (http://people.apache.org/~sebgoa/devcloud2.ova)
* Import the image into VirtualBox. After VM boots up, login into VM with `username: root, password: password`

### Contrail Control Node
* Download any of the Ubuntu VirtualBox images from [here] (http://virtualboximages.com/Ubuntu+12.04.4+amd64+Desktop+VirtualBox+VDI). (This is a Desktop image, feel free to disable the XServer or choose a Ubuntu image of your choice. Instructions to disable XServer can be found [here] (http://askubuntu.com/questions/16371/how-do-i-disable-x-at-boot-time-so-that-the-system-boots-in-text-mode))
* Create two network interfaces for the VM. (Open "Settings" of the image and choose "Network" to create the interfaces)
    * Host-only network interface - Assign a new static IP address of 192.168.56.30 to this. (Add the following config to `/etc/network/interfaces`.

            auto eth0
            iface eth0 inet static
                address 192.168.56.30
                netmask 255.255.255.0
                network 192.168.56.0
                broadcast 192.168.56.255
                gateway 192.168.56.1
     
    * NAT network interface - This interface is to connect to the internet.
* Import the image into VirtualBox. After VM boots up, login into VM with `username: adminuser, password: adminuser`
* Run `sudo passwd root` and set a root password to `adminuser` for the Ubuntu VM. The scripts uses `adminuser` as the root password.
* Install ssh if not already installed, `sudo apt-get update` and `sudo apt-get install openssh-server`
* Reboot the VM once.

## Building the code from Source
### Cloudstack
* [Checkout](https://cwiki.apache.org/confluence/display/CLOUDSTACK/Getting+the+Source+Code) the latest master code
* Build the  management server on your host machine (laptop)

         mvn -P developer,systemvm clean install

### Compute Node (Xen with XCP and Contrail Networking)
* **Note:** This section assumes that you have a valid Git account and you have setup the account for ssh access. If you have not, follow [these] (https://help.github.com/articles/generating-ssh-keys) instructions 
* In the DevCloud VM, clone the scripts which would build and install the Contrail bits.
         `git clone https://github.com/rranjeet/vrouter-xen-utils.git`
* Go to the directory, `cd vrouter-xen-utils/contrail-devcloud`
* Run `sudo install_dependencies.sh`
* And run `download_the_code.sh`. The shell script will prompt for your git password to download the Contrail code.
    * Sometimes, wget freezes while downloading some of the packages. If you see that the download is frozen for a long time, break in and restart the script.
* To build, run `build_copy.sh`. The build would take 60 minutes to complete.
* To setup, run `xen_setup.sh`.
* Restart the VM.

### Contrail Control Node 
* **Note:** This section assumes that you have a valid Git account and you have setup the account for ssh access. If you have not, follow [these] (https://help.github.com/articles/generating-ssh-keys) instructions 

* Install git, `apt-get install git`
* In the Ubuntu VM, clone the scripts which would build and install the Contrail Control Node bits.
         `git clone https://github.com/rranjeet/vrouter-xen-utils.git`
* Go to the directory, `cd vrouter-xen-utils/contrail-devcloud`
* Run `install_control_dependencies.sh`
* And run `download_the_code.sh`. The shell script will prompt for your git password to download the Contrail code.
    * Sometimes, wget freezes while downloading some of the packages. If you see that the download is frozen for a long time, break in and restart the script.


## Provisioning/Starting the setup
### CloudStack
* Create a file with name `contrail.properties` in path `client/target/generated-webapp/WEB-INF/classes/contrail.properties` and add the following text into it.

        api.hostname = 192.168.56.30
        api.port = 8082

* Drop the existing Cloudstack state (if any) and set up the database clean.

        mvn -P developer -pl developer,tools/devcloud -Ddeploydb

* Start the management Server

        mvn -pl :cloud-client-ui jetty:run

  Wait till the management server is up and running.

