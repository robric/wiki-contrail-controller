## Setting up the undercloud environment
1.  Log in to your machine (baremetal or VM) where you want to install the undercloud as a non-root user (such as the stack user):
```
ssh <non-root-user>@<undercloud-machine>
```
If you don’t have a non-root user created yet, log in as root and create one with following commands:
```
sudo useradd stack
sudo passwd stack  # specify a password

echo "stack ALL=(root) NOPASSWD:ALL" | sudo tee -a /etc/sudoers.d/stack
sudo chmod 0440 /etc/sudoers.d/stack

su - stack
```
## Undercloud installation
Ensure that there is a FQDN hostname set and that the $HOSTNAME environment variable matches that value.You can set the hostname settings manually. The manual steps are as follows:

```
undercloud_name=`hostname -s`
undercloud_suffix=`hostname -d`
hostnamectl set-hostname ${undercloud_name}.${undercloud_suffix}
hostnamectl set-hostname --transient ${undercloud_name}.${undercloud_suffix}
```
Get the undercloud ip and set the correct entries in /etc/hosts, ie (assuming the mgmt nic is eth0):
```
undercloud_ip=`ip addr sh dev eth0 |grep "inet " |awk '{print $2}' |awk -F"/" '{print $1}'`
echo ${undercloud_ip} ${undercloud_name}.${undercloud_suffix} ${undercloud_name} >> /etc/hosts
sudo  chmod 777 /etc/hosts
```
tripleo queens/current
```
tripeo_repos=`python -c 'import requests;r = requests.get("https://trunk.rdoproject.org/centos7-queens/current"); print r.text ' |grep python2-tripleo-repos|awk -F"href=\"" '{print $2}'|awk -F"\"" '{print $1}'`
yum install -y https://trunk.rdoproject.org/centos7-queens/current/${tripeo_repos}
tripleo-repos -b queens current
sudo yum install -y python-tripleoclient
```
## Modify the undercloud.conf with the following entries:


