# 1 Introduction:

Database management tool to check, heal and clean inconsistent DB entries.

where, 

* DB - Refers to zookeeper and Cassandra and 
* Inconsistent DB entries - Refers to data inconsistency between tables in Cassandra or inconsistency between zookeeper and Cassandra.

## 1.1 Commands to manage all inconsistencies

Tool supports various positional arguments treated as commands to for checking, healing and cleaning various DB inconsistencies. Following three commands can perform all inconsistency check/heal/clean by itself.

* `check` command is used to check all supported inconsistency .
* `heal` command is used to heal all supported inconsistency.
* `clean` command is used to clean all supported inconsistency.


## 1.2 Commands to manage sub set of  inconsistencies

Tool also provides commands to support sub set of DB inconsistency check/heal/clean. They are prefixed with `check`, `heal` and `clean` for checking, healing and cleaning. 

Following are some examples,

* `check_route_targets_id` command is used to checks and display the list of stale and missing route targets id.
* `heal_route_targets_id` command is used to create the missing route targets id in DB's
* `clean_stale_route_target_id`command is used to delete the stale route target id in DB's

# 2 Help and Usage:    

Database management tool (db_manage.py) is located in config node at the following location.

``
    /usr/lib/python2.7/site-packages/vnc_cfg_api_server
``

Tool can be executed with --help option to display the help and usage information about the database management tool.

``
    python db_manage.py --help
``
   
    usage: db_manage.py [-h] [--api-conf API_CONF] [--execute] [--verbose]
                        [--debug]
                        operation

    EXAMPLES:

        Checker example,

            python db_manage.py check_route_targets_id
                - Checks and displays the list of stale and missing route targets id.

        Healer examples,

            python db_manage.py heal_route_targets_id
                - Displays the list of missing route targets id to be healed(dry-run).

            python db_manage.py --execute heal_route_targets_id
                - Creates the missing route targets id in DB's with(--execute).

        Cleaner examples,

            python db_manage.py clean_stale_route_target_id
                - Displays the list of stale route target id to be cleaned(dry-run).

            python db_manage.py --execute clean_stale_route_target_id
                - Deletes the stale route target id in DB's with(--execute).

            python db_manage.py clean_stale_route_target
                - Displays the list of stale route target to be cleaned(dry-run).

            python db_manage.py --execute clean_stale_route_target
                - Deletes the stale route target in DB's with(--execute).

        Where,
            ['check_route_targets_id', 'heal_route_targets_id', 'clean_stale_route_target_id', 'clean_stale_route_target']
                - are some of the supported operations listed in positional arguments.
            DB's - refer to zookeeper and cassandra.

    NOTES:
            check, heal and clean operations are used to do all checker, healer and cleaner operations respectively.

    DOCUMENTAION:
            Documentaion for each supported operation is displayed in positional arguments below.

    positional arguments:
      operation            Supported healers,
                       
                               heal_subnet_addr_alloc
                                   -  Creates missing virtaul-networks, sunbets and instance-ips
                                   in zk.
                               heal_subnet_uuid
                                   - Creates missing subnet uuid in useragent_keyval_table
                                   of cassandra.
                               heal_route_targets_id
                                   - Creates missing route-targets id's in zk.
                               heal
                                   - Creates missing entries in DB's for all inconsistencies.
                               heal_security_groups_id
                                   - Creates missing security-group id's in zk.
                               heal_children_index
                                   - Creates missing children index for parents in obj_fq_name_table
                                   of cassandra.
                               heal_fq_name_index
                                   - Creates missing rows/columns in obj_fq_name_table of cassandra for
                                   all entries found in obj_uuid_table.
                               heal_virtual_networks_id
                                   - Creates missing virtual-network id's in zk.
                       
                           Supported checkers,
                       
                               check_subnet_addr_alloc
                                   - Displays inconsistencies between zk and cassandra for Ip's, Subnets
                                   and VN's.
                               check_zk_mode_and_node_count
                                   - Displays error info about broken zk cluster.
                               check_obj_mandatory_fields
                                   - Displays missing mandatory fields of objects at obj-uuid-table
                                   in Cassandra.
                               check_cassandra_keyspace_replication
                                   - Displays error info about wrong replication factor in Cassandra.
                               check_virtual_networks_id
                                   - Displays VN ID inconsistencies between zk and cassandra.
                               check
                                   - Checks and displays all the inconsistencies in DB.
                               check_security_groups_id
                                   - Displays Security group ID inconsistencies between zk and cassandra.
                               
                               check_orphan_resources
                               - Displays orphaned entries in obj_uuid_table of cassandra.
                               check_subnet_uuid
                                   - Displays inconsistencies between useragent/obj-uuid-table for
                                   subnets, subnet uuids in Cassandra.
                               check_route_targets_id
                                   - Displays route targets ID inconsistencies between zk and cassandra.
                               
                               check_fq_name_uuid_match
                                   - Displays mismatch between obj-fq-name-table and obj-uuid-table
                                   in Cassandra.
                       
                           Supported cleaners,
                       
                               clean_subnet_addr_alloc
                                   - Removes extra VN's, subnets and IP's from zk.
                               clean_stale_route_target
                                   - Removes stale RT's without routing-instance or
                                   logical-router backrefs
                               clean_stale_back_refs
                                   - Removes stale backref entries from obj_uuid_table of cassandra.
                               clean_stale_instance_ip
                                   - Removes stale IIP's without virtual-network or
                                   virtual-machine-interface refs
                               clean_orphan_resources
                                   - Removes extra VN's, subnets and IP's from zk.
                               clean_stale_route_target_id
                                   - Removes stale RT ID's from route_target_table of cassandra and zk.
                               
                               clean_obj_missing_mandatory_fields
                                   - Removes stale resources which are missing mandatory fields from
                                   obj_uuid_table of cassandra.
                               clean_stale_virtual_network_id
                                   - Removes stale VN ID's from obj_uuid_table of cassandra and zk.
                               clean_stale_children
                                   - Removes stale children entries from obj_uuid_table of cassandra.
                               clean_stale_security_group_id
                                   - Removes stale SG ID's from obj_uuid_table of cassandra and zk.
                               clean
                                   - Removes stale entries from DB's.
                               clean_vm_with_no_vmi
                                   - Removes VM's without VMI from cassandra.
                               clean_stale_subnet_uuid
                                   - Removes stale UUID's from useragent_keyval_table of cassandra.
                               clean_stale_fq_names
                                   - Removes stale entries from obj_fq_name_table of cassandra.
                       

    optional arguments:
      -h, --help           show this help message and exit
      --api-conf API_CONF  Path to contrail-api conf file, default /etc/contrail-api.conf
      --execute            Exceute database modifications
      --verbose            Run in verbose/INFO mode, default False
      --debug              Run in debug mode, default False