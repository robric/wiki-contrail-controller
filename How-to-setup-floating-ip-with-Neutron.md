(See also https://github.com/Juniper/contrail-controller/wiki/Simple-Gateway)
(See also Noel's network test script is at:  https://github.com/noelbk/devstack/blob/contrail/contrail/test_network.sh )

# Add This on LocalRC

```
CONTRAIL_VGW_INTERFACE=vgw
CONTRAIL_VGW_PUBLIC_SUBNET=10.99.99.0/24 # replace with your floating IP range
CONTRAIL_VGW_PUBLIC_NETWORK=default-domain:admin:public:public
```

# Setup public network in neutron

```
. openrc admin admin
neutron net-create public
public_id=`neutron net-list | awk '/public/{print }'`
neutron subnet-create --name public-subnet1 $public_id $CONTRAIL_VGW_PUBLIC_SUBNET --disable-dhcp
```

# Setup floating ip pool in contrail

```
   python /opt/stack/contrail/controller/src/config/utils/create_floating_pool.py --public_vn_name default-domain:admin:public --floating_ip_pool_name floatingip_pool
   python /opt/stack/contrail/controller/src/config/utils/use_floating_pool.py --project_name default-domain:admin --floating_ip_pool_name default-domain:admin:public:floatingip_pool
```

# Setup Security Group

```
. openrc admin demo
nova secgroup-list
nova secgroup-list-rules default
nova secgroup-add-rule default tcp 22 22 0.0.0.0/0
nova secgroup-add-rule default icmp -1 -1 0.0.0.0/0
nova secgroup-list-rules default
```

# Setup floating ip
```
neutron floatingip-create $public_id
neutron floatingip-associate $floatingip_id $port_id
```

# Test it
neutron floatingip-show $floatingip_id
ssh cirros@$floatingip_ip

