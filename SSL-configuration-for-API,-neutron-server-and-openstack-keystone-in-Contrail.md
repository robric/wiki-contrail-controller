# Certs and SSL Keystone connection
## Create ssl directories, give proper ownership, generate certificates
This example uses self signed certificates)
    mkdir /etc/contrail/ssl
    chown contrail:contrail /etc/contrail/ssl
    mkdir /etc/neutron/ssl
    chown neutron:neutron /etc/neutron/ssl
 
    ./certs.sh <IP> #Contrail certs (generated in /etc/contrail/ssl)
    ./certs.sh <hostname> #Neutron certs  (generated in /etc/neutron/ssl)
 
## Rename neutron certs (for clarity)
    cd /etc/neutron/ssl
    mv apiserver_ca.pem neutron_ca.pem
    mv apiserver.key neutron.key
    mv apiserver.pem neutron.pem

## Combine certs into bundles
    cd /etc/contrail/ssl
    cat apiserver_ca.pem apiserver.key apiserver.pem >> contrailbundle.pem
    cd /etc/neutron/ssl
    cat neutron_ca.pem neutron.key neutron.pem >> neutronbundle.pem

(Copy keystone CA into neutron and contrail CA's)

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
# HAproxy
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