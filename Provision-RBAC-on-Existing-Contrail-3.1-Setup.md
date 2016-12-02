Below are the steps to provision RBAC in an EXISTING 3.1 setup.
 
          Set “aaa_mode=rbac” in /etc/contrail/contrail-api.conf

In /etc/neutron/api-paste.ini

          REPLACE

          “keystone = cors request_id catch_errors authtoken keystone context extensions neutronapiapp_v2_0”  
          
          WITH  

          “keystone = user_token cors request_id catch_errors authtoken keystone context extensions neutronapiapp_v2_0”

ADD below lines,

           [filter:user_token]
           paste.filter_factory = neutron_plugin_contrail.plugins.opencontrail.neutron_middleware:token_factory

In /etc/contrail/contrail-

Restart services,

           service contrail-api restart
           service neutron-server restart
           service supervisor-webui restart