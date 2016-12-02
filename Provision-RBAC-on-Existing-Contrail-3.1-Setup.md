## Below are the steps to provision RBAC in an EXISTING 3.1/3.0.3.X setup

### In /etc/contrail/contrail-api.conf

          SET “aaa_mode=rbac”

### In /etc/neutron/api-paste.ini

          REPLACE

          “keystone = cors request_id catch_errors authtoken keystone context extensions neutronapiapp_v2_0”  
          
          WITH  

          “keystone = user_token cors request_id catch_errors authtoken keystone context extensions neutronapiapp_v2_0”

           AND ADD BELOW LINES,

           [filter:user_token]
           paste.filter_factory = neutron_plugin_contrail.plugins.opencontrail.neutron_middleware:token_factory

### In /etc/contrail/contrail-analytics-api.conf

           SET "aaa_mode = no-auth"

### Restart services,

           service contrail-api restart
           service neutron-server restart
           service supervisor-webui restart