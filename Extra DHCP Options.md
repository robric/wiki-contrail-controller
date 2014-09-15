DHCP options can be configured using extra-dhcp-options option in the following Neutron command. This support is available from R1.10 onwards.

`neutron port-create net1 --extra-dhcp-opts type=dict list=true opt_name='dhcp_option_name',opt_value='value'`

The following table lists the DHCP options, their options names and expected values to be used in the --extra-dhcp-opts option. The opt_name field can have either the DHCP code or the DHCP name specified below for the corresponding code.
```
 DHCP Code  DHCP opt_name                          DHCP opt_value

   1        subnet-mask                            Single IP
   2        time-offset                            32-bit signed int
   3        routers                                1+ IP
   4        time-servers                           1+ IP
   5        name-servers                           1+ IP
   6        domain-name-servers                    1+ IP
   7        log-servers                            1+ IP
   8        quote-servers                          1+ IP
   9        lpr-servers                            1+ IP
  10        impress-servers                        1+ IP
  11        resource-location-servers              1+ IP
  12        host-name                              String
  13        boot-size                              16-bit uint
  14        merit-dump                             String
  15        domain-name                            String
  16        swap-server                            Single IP
  17        root-path                              String
  18        extension-path                         String
  19        ip-forwarding                          Bool (0 or 1)
  20        non-local-source-routing               Bool (0 or 1)
  21        policy-filter                          Multiples of two IPs
  22        max-dgram-reassembly                   16-bit uint
  23        default-ip-ttl                         Byte
  24        path-mtu-aging-timeout                 Unsigned 32-bit int
  25        path-mtu-plateau-table                 16-bit uint array
  26        interface-mtu                          16-bit uint
  27        all-subnets-local                      Bool (0 or 1)
  28        broadcast-address                      Single IP (Vrouter overrides)
  29        perform-mask-discovery                 Bool (0 or 1)
  30        mask-supplier                          Bool (0 or 1)
  31        router-discovery                       Bool (0 or 1)
  32        router-solicitation-address            Single IP
  33        static-routes                          Multiples of two IPs
  34        trailer-encapsulation                  Bool (0 or 1)
  35        arp-cache-timeout                      32-bit uint
  36        ieee802-3-encapsulation                Bool (0 or 1)
  37        default-tcp-ttl                        Byte
  38        tcp-keepalive-interval                 32-bit uint
  39        tcp-keepalive-garbage                  Bool (0 or 1)
  40        nis-domain                             String
  41        nis-servers                            1+ IP
  42        ntp-servers                            1+ IP
  43        vendor-encapsulated-options            String
  44        netbios-name-servers                   1+ IP
  45        netbios-dd-server                      1+ IP
  46        netbios-node-type                      Byte
  47        netbios-scope                          String
  48        font-servers                           1+ IP
  49        x-display-manager                      1+ IP
  50        dhcp-requested-address                 Single IP
  51        dhcp-lease-time                        32-bit uint (Vrouter overrides)
  52        dhcp-option-overload                   Byte
  53        dhcp-message-type                      Byte (Vrouter overrides)
  54        dhcp-server-identifier                 Single IP (Vrouter overrides)
  55        dhcp-parameter-request-list            Byte Array
  56        dhcp-message                           String
  57        dhcp-max-message-size                  16-bit uint
  58        dhcp-renewal-time                      32-bit uint
  59        dhcp-rebinding-time                    32-bit uint
  60        class-id                               String
  61        dhcp-client-identifier                 Byte followed by a String
  62        nwip-domain                            String
  63        nwip-suboptions                        Byte Array (encode data)
  64        nisplus-domain                         String
  65        nisplus-servers                        1+ IP
  66        tftp-server-name                       String
  67        bootfile-name                          String
  68        mobile-ip-home-agent                   0+ IP
  69        smtp-server                            1+ IP
  70        pop-server                             1+ IP
  71        nntp-server                            1+ IP
  72        www-server                             1+ IP
  73        finger-server                          1+ IP
  74        irc-server                             1+ IP
  75        streettalk-server                      1+ IP
  76        streettalk-directory-assistance-server 1+ IP
  77        user-class                             Byte Array (encode data)
  78        slp-directory-agent                    Byte followed by 1+ IP
  79        slp-service-scope                      Byte followed by a String
  80        rapid-commit                           No data
  81        client-fqdn                            Byte Array (Sent by clients)
  83        storage-ns                             Byte Array (encode data)
  85        nds-servers                            1+ IP
  86        nds-tree-name                          Byte Array (encode data)
  87        nds-context                            Byte Array (encode data)
  88        bcms-controller-names                  Byte Array (encode data)
  89        bcms-controller-address                Byte Array (encode data)
  90        dhcp-auth                              Byte Array (encode data)
  91        dhcp-client-last-time                  32-bit uint
  92        associated-ip                          1+ IP
  93        system-architecture                    16-bit uint
  94        interface-id                           Byte Array
  95        ldap-servers                           1+ IP
  97        machine-id                             String
  98        user-auth                              String
  99        geoconf-civic                          Byte Array (encode data)
 100        ieee-1003-1-tz                         String
 101        ref-tz-db                              String
 112        netinfo-server-address                 String
 113        netinfo-server-tag                     1+ IP
 114        default-url                            String
 116        auto-configure                         Bool (0 or 1)
 117        name-search                            16-bit uint array
 118        subnet-selection                       Single IP
 119        domain-search                          Domain Name
 120        sip-servers                            Byte Array (encode data)
 121        classless-static-routes                Multiples of <subnet/plen, gw>
 122        dhcp-ccc                               Byte Array (encode data)
 123        dhcp-geoconf                           Byte Array (encode data)
 124        vendor-class-identifier                Byte Array (encode data)
 125        vivso                                  Byte Array (encode data)
 128        tftp-server                            1+ IP
 129        pxe-vendor-specific-129                String
 130        pxe-vendor-specific-130                String
 131        pxe-vendor-specific-131                String
 132        pxe-vendor-specific-132                String
 133        pxe-vendor-specific-133                String
 134        pxe-vendor-specific-134                String
 135        pxe-vendor-specific-135                String
 136        pana-agent                             1+ IP
 137        lost-server                            Byte Array (encode data)
 138        capwap-ac-v4                           1+ IP
 139        dhcp-mos                               Byte Array (encode data)
 140        dhcp-fqdn-mos                          Byte Array (encode data)
 141        sip-ua-config-domain                   Byte Array (encode data)
 142        andsf-servers                          1+ IP
 144        dhcp-geoloc                            Byte Array (encode data)
 145        force-renew-nonce-cap                  Byte Array (encode data)
 146        rdnss-selection                        Byte Array (encode data)
 150        tftp-server-address                    1+ IP
 151        status-code                            Byte followed by a String
 152        dhcp-base-time                         32-bit uint
 153        dhcp-state-start-time                  32-bit uint
 154        dhcp-query-start-time                  32-bit uint
 155        dhcp-query-end-time                    32-bit uint
 156        dhcp-state                             Byte
 157        data-source                            Byte
 158        pcp-server                             Byte Array (encode data)
 208        dhcp-pxe-magic                         32-bit uint
 209        config-file                            String
 210        path-prefix                            String
 211        reboot-time                            32-bit uint
 212        dhcp-6rd                               Byte Array (encode data)
 213        dhcp-access-domain                     Byte Array
 220        subnet-allocation                      Byte Array (encode data)
 221        dhcp-vss                               Byte followed by a String
```
IP address should be in dotted IPv4 format.