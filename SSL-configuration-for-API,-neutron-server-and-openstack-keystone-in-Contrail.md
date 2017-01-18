# Introduction

Provisioning keystone, api-server and neutron-server with SSL. This is achieved by configuring keystone with native SSL and api-server/neutron-server through SSL termination using Haproxy.

# Section1: Keystone Settings for SSL

## 1. Create ssl directories and assign ownership

        # In Keystone Node,
        mkdir -p /etc/keystone/ssl; chown keystone:keystone /etc/keystone/ssl

## 2. Download the script to create self-signed certs

Download the script from github, if provisionig contrail release less than 3.0.3.2.
otherwise the script will be available at /opt/contrail/bin/create-ssl-certs.sh when 
installing contrail-setup package.

        wget https://raw.githubusercontent.com/Juniper/contrail-provisioning/master/contrail_provisioning/common/scripts/create-ssl-certs.sh

## 3. Create self-signed SSL certs for Keystone

        # In Keystone Node,
        create-ssl-certs.sh <KeystoneNodeIP|VIP> /etc/keystone/ssl/ keystone


# Section2: api-server SSL settings

## 1. Create ssl directories and assign ownership

        # In api-server Node,
        mkdir -p /etc/contrail/ssl; chown contrail:contrail /etc/contrail/ssl

## 2. Download the script to create self-signed certs

Download the script from github, if provisionig contrail release less than 3.0.3.2.
otherwise the script will be available at /opt/contrail/bin/create-ssl-certs.sh when 
installing contrail-setup package.

        wget https://raw.githubusercontent.com/Juniper/contrail-provisioning/master/contrail_provisioning/common/scripts/create-ssl-certs.sh\

## 3. Create self-signed SSL certs for api-server

        # In api-server Node,
        create-ssl-certs.sh <ConfigNodeIP|VIP> /etc/contrail/ssl/ apiserver

## 4. Create certificate bundle

Certificates bundle will be used in Haproxy for SSL termination,

        # In api-server Node,
        cd /etc/contrail/ssl/; cat certs/apiserver_ca.pem private/apiserver.key certs/apiserver.pem >> certs/apiservercertbundle.pem

## 5. Copy keystone certs to api-server node

keystone certificate and CA needs to be available in api-server node , so that
api-server can talk to keystone securely using keystone certs/CA.

        # From api-server node,
        scp <user>@<keystoneNodeIp>:/etc/keystone/ssl/certs/keystone.pem /etc/contrail/ssl/certs/
        scp <user>@<keystoneNodeIp>:/etc/keystone/ssl/certs/keystone_ca.pem /etc/contrail/ssl/certs/
        chown -R contrail:contrail /etc/contrail/ssl/certs/


# Section3: neutron-server SSL settings

## 1. Create ssl directories and assign ownership

        # In neutron-server Node,
        mkdir -p /etc/neutron/ssl; chown neutron:neutron /etc/neutron/ssl

        # In api-server Node,
        mkdir -p /etc/contrail/ssl; chown contrail:contrail /etc/contrail/ssl

## 2. Download the script to create self-signed certs

Download the script from github, if provisionig contrail release less than 3.0.3.2.
otherwise the script will be available at /opt/contrail/bin/create-ssl-certs.sh when 
installing contrail-setup package.

        wget https://raw.githubusercontent.com/Juniper/contrail-provisioning/master/contrail_provisioning/common/scripts/create-ssl-certs.sh

## 3. Create self-signed SSL certs for neutron-server

        # In neutron-server Node,
        create-ssl-certs.sh <NeutronNodeIP|VIP> /etc/neutron/ssl/ neutron

## 4. Create certificate bundle

Certificates bundle will be used in Haproxy for SSL termination,

        # In neutron-server Node,
        cd /etc/neutron/ssl/; cat certs/neutron_ca.pem private/neutron.key certs/neutron.pem >> certs/neutroncertbundle.pem

## 5. Copy keystone certs to neutron-server node

keystone certificate and CA needs to be available in neutron-server node , so that
neutron-server can talk to keystone securely using keystone certs/CA.

        # From neutron-server node,
        scp <user>@<keystoneNodeIp>:/etc/keystone/ssl/certs/keystone.pem /etc/neutron/ssl/certs/
        scp <user>@<keystoneNodeIp>:/etc/keystone/ssl/certs/keystone_ca.pem /etc/neutron/ssl/certs/
        chown -R neutron:neutron /etc/neutron/ssl/certs/

## 6. Configure neutron-server frontend

Ensure the neutron-server haproxy config looks like below in /etc/haproxy.cfg

        frontend neutron-server
            bind *:9696 ssl crt /etc/neutron/ssl/certs/neutroncertbundle.pem
            default_backend    neutron-server-backend

        backend neutron-server-backend
            option nolinger
            option forwardfor
            balance     roundrobin
            http-request set-header X-Forwarded-Port %[dst_port]
            http-request add-header X-Forwarded-Proto https if { ssl_fc }
            server <NeutronHostIp1> <NeutronHostIp1>:9697 check inter 2000 rise 2 fall 3
            server <NeutronHostIp2> <NeutronHostIp2>:9697 check inter 2000 rise 2 fall 3
            server <NeutronHostIp3> <NeutronHostIp3>:9697 check inter 2000 rise 2 fall 3

Restart harproxy,

        service haproxy restart

## 7. Configure neutron.conf

Add the keystone certificate information in keystone_authtoken section of neutron.conf

        openstack-config --set /etc/neutron/neutron.conf keystone_authtoken auth_protocol https
        openstack-config --set /etc/neutron/neutron.conf keystone_authtoken certfile /etc/neutron/ssl/certs/keystone.pem
        openstack-config --set /etc/neutron/neutron.conf keystone_authtoken keyfile /etc/neutron/ssl/certs/keystone.pem
        openstack-config --set /etc/neutron/neutron.conf keystone_authtoken cafile /etc/neutron/ssl/certs/keystone_ca.pem

## 8. Configure ContrailPlugin.ini

Add the api-server certificate information in APISERVER section of ContrailPlugin

        openstack-config --set /etc/neutron/plugins/opencontrail/ContrailPlugin.ini APISERVER use_ssl True
        openstack-config --set /etc/neutron/plugins/opencontrail/ContrailPlugin.ini APISERVER insecure False
        openstack-config --set /etc/neutron/plugins/opencontrail/ContrailPlugin.ini APISERVER certfile /etc/neutron/ssl/certs/apiserver.pem
        openstack-config --set /etc/neutron/plugins/opencontrail/ContrailPlugin.ini APISERVER keyfile /etc/neutron/ssl/certs/apiserver.pem
        openstack-config --set /etc/neutron/plugins/opencontrail/ContrailPlugin.ini APISERVER cafile /etc/neutron/ssl/certs/apiserver_ca.pem

## 8. Configure vnc_api_lib.ini


Configure vnc_api_lib.ini in neutron-server, which will be used by vnc_api client library to talk to api-server.
vnc_api library is used by neutron contrail plugin.

        chown contrail:contrail /etc/contrail/vnc_api_lib.ini 
        usermod -a -G contrail neutron

        # SKIP below commands in (8. Configure vnc_api_lib.ini), if neutron-server/plugin is in same node as config node.
        openstack-config --set /etc/contrail/vnc_api_lib.ini global insecure False
        openstack-config --set /etc/contrail/vnc_api_lib.ini global certfile /etc/neutron/ssl/certs/apiserver.pem
        openstack-config --set /etc/contrail/vnc_api_lib.ini global keyfile /etc/neutron/ssl/certs/apiserver.pem
        openstack-config --set /etc/contrail/vnc_api_lib.ini global cafile /etc/neutron/ssl/certs/apiserver_ca.pem

        openstack-config --set /etc/contrail/vnc_api_lib.ini auth insecure False
        openstack-config --set /etc/contrail/vnc_api_lib.ini auth AUTHN_PROTOCOL https
        openstack-config --set /etc/contrail/vnc_api_lib.ini auth certfile /etc/neutron/ssl/certs/keystone.pem
        openstack-config --set /etc/contrail/vnc_api_lib.ini auth keyfile /etc/neutron/ssl/certs/keystone.pem
        openstack-config --set /etc/contrail/vnc_api_lib.ini auth cafile /etc/neutron/ssl/certs/keystone_ca.pem



## Add keystone config to neutron.conf, for example:
    [keystone_authtoken]
    admin_tenant_name = service
    admin_user = neutron
    admin_password = neutronservicepassword
    auth_host = keystone.hostname.fqdn
    auth_port=35357
    admin_token = openstack_identity_bootstrap_token
    auth_protocol = https
    certfile=/etc/neutron/ssl/neutron.pem
    keyfile=/etc/neutron/ssl/neutron.key
    cafile=/etc/neutron/ssl/neutron_ca.pem
## Create /etc/contrail/contrail-keystone-auth.conf, example:
    [KEYSTONE]
    auth_host=keystone.hostname.fqdn
    auth_protocol=https
    auth_port=35357
    admin_user=admin
    admin_password=adminpassword
    admin_token=admintoken
    admin_tenant_name=admin
    certfile=/etc/contrail/ssl/apiserver.pem
    keyfile=/etc/contrail/ssl/apiserver.key
    cafile=/etc/contrail/ssl/apiserver_ca.pem
    memcache_servers=127.0.0.1:11211`

## Add to vnc_api_lib.ini
    [auth]
    AUTHN_TYPE = keystone
    AUTHN_SERVER = keystone.hostname.fqdn
    AUTHN_PROTOCOL = https
    AUTHN_PORT = 35357
    AUTHN_URL = /v2.0/tokens
    certfile=/etc/contrail/ssl/apiserver.pem
    keyfile=/etc/contrail/ssl/apiserver.key
    cafile=/etc/contrail/ssl/apiserver_ca.pem
*NOTE: The keystone server uses the hostname, this is important because the keystone certs are generated for the hostname, not IP.*
# Contrail API and Neutron server SSL termination in HAproxy
## Ensure that the contrail-api sections look similar to this:
    frontend  contrail-api
        bind *:8082 ssl crt /etc/contrail/ssl/contrailbundle.pem
        default_backend    contrail-api-backend

    backend contrail-api-backend
        option nolinger
        option forwardfor
        balance     roundrobin
        http-request set-header X-Forwarded-Port %[dst_port]
        http-request add-header X-Forwarded-Proto https if { ssl_fc }
        server <api host ip> <api host ip>:9100 check inter 2000 rise 2 fall 3
## Ensure the neutron section looks like this:
    frontend neutron-server
        bind *:9696 ssl crt /etc/neutron/ssl/neutronbundle.pem
        default_backend    neutron-server-backend
    backend neutron-server-backend
        option nolinger
        option forwardfor
        balance     roundrobin
        http-request set-header X-Forwarded-Port %[dst_port]
        http-request add-header X-Forwarded-Proto https if { ssl_fc }
        server <neutron host ip> <neutron host ip>:9697 check inter 2000 rise 2 fall 3

## Update the neutron endpoint in keystone to use https:
### From Keystone server
    mysql -ukeystone -pkeystonemysqlpass keystone
    select * from service;
### take neutron service id
    update endpoint set url='https://neutron.server.fqdn:9696' where service_id='<neutron service id>';
### Add neutron_ca, cert, and nova ca to the os controller at /etc/nova/ssl/certs/sslsdn.pem, then add to nova.conf
    neutron_url=https://<neutron server fqdn>:9696
    neutron_ca_certificates_file=/etc/nova/ssl/certs/sslsdn.pem
### On the contrail controller change /etc/neutron/plugins/opencontrail/ContrailPlugin.ini to use port 9100 (internal traffic)
    api_server_port = 9100
### Point local contrail services to port 9100 for contrail-api
    /etc/contrail/contrail-schema.conf
    /etc/contrail/contrail-svc-monitor.conf
    /etc/contrail/contrail-device-manager.conf