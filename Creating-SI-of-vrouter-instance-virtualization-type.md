# Since version R2.2 contrail-controller supports two vrouter instance types of vrouter-instance: docker and libvirt-qemu.

## Introduction:

"vrouter-instance" type enables users to create SI without 3rdparty applications (like Nova). Virtual machines are created directly by vrouter-agent application. At this moment vrouter-agent supports two adapters: docker and libvirt-qemu.

docker adapter enables vrouter-agent to create SI as docker container.
libvirt-qemu adapter enables vrouter-agent to create SI using libvirt API.

### Pre-requirements for docker:
1. Make sure docker is installed, running and is capable to create container on vrouter.
2. Make sure image is stored into local docker image base or available in the public.

### Pre-requirements for libvirt-qemu:
1. Make sure libvirt-bin is installed, running and is capable to create virtual machine on vrouter
2. Make sure VM image is available before you create SI (vrouter-agent will not download image on the local disk itself)
3. Contrail-controller must be compiled with --with-libvirt option enabled

# Example commands:

## docker:

Before you start, make sure you have one compute-node with docker running

Firstly, you need to create service-template, for docker it may be something like this:

`curl -X POST -H "Content-Type: application/json; charset=UTF-8" -d '{"service-template": {"parent_type": "project", "fq_name": ["default-domain", "admin", "docker-template"], "service_template_properties": {"service_mode": "transparent", "image_name": "docker-vm-name", "service_type": "firewall", "flavor": null, "service_scaling": false, "instance_data": "{\"command\":\"/bin/bash\"}", "availability_zone_enable": false, "service_virtualization_type": "vrouter-instance", "vrouter_instance_type": "docker", "ordered_interfaces": true, "interface_type": [{"static_route_enable": false,"shared_ip": false,"service_interface_type": "management"}, {"static_route_enable": false, "shared_ip": false, "service_interface_type": "left"}, {"static_route_enable": false, "shared_ip": false, "service_interface_type": "right"}],}}}' http://localhost:8082/service-templates`

The most important parameters are:
service_virtualization_type and vrouter_instance_type. First must be equal to the "vrouter-instance", second parameter must be equal to "docker".

Ignored parameters: flavor, service_scaling, availability_zone_enable

## libvirt-qemu:

Example command for service-template is:

`curl -X POST -H "Content-Type: application/json; charset=UTF-8" -d '{"service-template": {"parent_type": "project", "fq_name": ["default-domain", "admin", "libvirt-template"], "service_template_properties": {"service_mode": "transparent", "image_name": "anything", "service_type": "firewall", "flavor": null, "service_scaling": false, "instance_data": "<xml>Here goes configuration for libvirt, skipping here to make curl shorter</xml>", "availability_zone_enable": false, "service_virtualization_type": "vrouter-instance", "vrouter_instance_type": "libvirt-qemu", "ordered_interfaces": true, "interface_type": [{"static_route_enable": false,"shared_ip": false,"service_interface_type": "management"}, {"static_route_enable": false, "shared_ip": false, "service_interface_type": "left"}, {"static_route_enable": false, "shared_ip": false, "service_interface_type": "right"}],}}}' http://localhost:8082/service-templates`

The most important parameters are:
service_virtualization_type and vrouter_instance_type. First must be equal to the "vrouter-instance", second parameter must be equal to "libvirt-qemu". Also, instance_data parameter is very important: it contains XML configuration that will be passed to libvirt. But you must know that vrouter-agent will modify it a little during machine creation:
1. It will append first letters of machine UUID to name node
2. It will append <interfaces> nodes to <devices>.

For more reference (especially how to create XML configuration) check libvirt documentation.

Example XML configuration:
`<domain type='kvm' id='2'>
      <name>name</name>
      <memory unit='MB'>1024</memory>
      <currentMemory unit='MB'>1024</currentMemory>
      <vcpu placement='static'>1</vcpu>
      <resource>
          <partition>/machine</partition>
      </resource>
      <os>
          <type arch='x86_64' machine='pc-1.0'>hvm</type>
          <boot dev='hd'/>
      </os>
      <clock offset='utc'/>
      <on_poweroff>destroy</on_poweroff>
      <on_reboot>restart</on_reboot>
      <on_crash>destroy</on_crash>
      <devices>
          <emulator>/usr/bin/qemu-system-x86_64</emulator>
          <disk type='file' device='disk'>
              <driver name='qemu' type='qcow2'/>
              <source file='/var/lib/libvirt/images/image_filename.img'/>
              <target dev='vda' bus='virtio'/>
          </disk>
          <controller type='virtio-serial' index='0' ports='16' vectors='4'/>
          <controller type='ide' index='0'>
              <alias name='ide0'/>
              <address type='pci' domain='0x0000' bus='0x00' slot='0x01' function='0x1'/>
          </controller>
          <controller type='pci' index='0' model='pci-root'></controller>
          <memballoon model='virtio'>
              <alias name='balloon0'/>
              <address type='pci' domain='0x0000' bus='0x00' slot='0x03' function='0x0'/>
          </memballoon>
          <serial type='pty'>
              <target port='0'/>
          </serial>
          <console type='pty'>
              <target type='serial' port='0'/>
          </console>
      </devices>
      <seclabel type='none'/>
      <features>
          <acpi/>
      </features>
    </domain>`

Note that <interface> nodes will be added by vrouter-agent!
For the XML above make sure image is stored in provided directory (/var/lib/libvirt/images/...) before creating service instance

## Creating service instance

In order to create such a service instance (docker or libvirt-qemu), execute this command:

`curl -X POST -H "Content-Type: application/json; charset=UTF-8" -d '{"service-instance": {"parent_type": "project", "fq_name": ["default-domain", "admin", "libvirt-instance"], "service_template_refs": [{"to": ["default-domain", "libvirt-template"], "href": "http://localhost:9100/service-template/69ba0ed9-a595-4f16-aeb0-68f305f50553", "attr": null, "uuid": "69ba0ed9-a595-4f16-aeb0-68f305f50553"}], "service_instance_properties": {"right_virtual_network": "", "left_ip_address": null, "availability_zone": null, "management_virtual_network": "", "auto_policy": false, "ha_mode": null, "virtual_router_id": "35d86cdb-2ca7-47d3-b136-32000aa05785", "interface_list": [{"virtual_network": "", "ip_address": null, "static_routes": null}, {"virtual_network": "", "ip_address": null, "static_routes": null}, {"virtual_network": "", "ip_address": null, "static_routes": null}], "right_ip_address": null, "left_virtual_network": "", "scale_out": {"auto_scale": false, "max_instances": 1}}}}' http://localhost:8082/service-instances`

This command will create one virtual machine one specified vrouter (using virtual_router_id param).

scale_out parameter is ignored

# Limitations:

1. <interfaces> nodes in libvirt are added automatically: which means for example that all sub-nodes will also be generated on fly: users are not able to edit them.
2. libvirt/docker will create interfaces will following order: left, right, management.