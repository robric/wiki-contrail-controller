The stock libvirt doesn’t have support for rate limiting of ‘ethernet’ interface types.  For example, following xml snippet shows the network bandwidth configuration data of a network interface of a vm.
```xml
 <interface type="ethernet">  
    <mac address="02:a3:a0:87:7f:61"/>  
    <model type="virtio"/>  
    <script path=""/>  
    <target dev="tapa3a0877f-61"/>  
    <bandwidth>  
        <inbound average="800" peak="1000" burst="30"/>  
        <outbound average="800" peak="1000" burst="30"/>  
    </bandwidth>  
 </interface>  
```
The above settings of this interface will not result in any tc qdisc settings for the tap device, `tapa3a0877f-61` in the host. So, the vm network traffic will not be throttled as per the above rate limiting configuration data. In order to make this work, following patches should be applied to libvirt source. 

Libvirt-0.9.8:   
https://bitbucket.org/contrail_admin/libvirt/commits/c33be693e13fa9cd8daee448c80e9860050cb514  
https://bitbucket.org/contrail_admin/libvirt/commits/a6aa7126d42994fd33ffe5c10f37e8ca765aaed2

Libvirt – 0.10.2:  
https://bitbucket.org/contrail_admin/libvirt/commits/4e50747ce22b9532f75b48edb5d3d6feba962ecc  
https://bitbucket.org/contrail_admin/libvirt/commits/a6aa7126d42994fd33ffe5c10f37e8ca765aaed2

There is a launchpad bug (1367095) for the above issue.