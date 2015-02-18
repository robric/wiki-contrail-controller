Currently, the OpenContrail ceilometer driver populates the `switch.port.receive.bytes`, `switch.port.receive.packets`, `switch.port.transmit.bytes`,  and `switch.port.transmit.packets` meters in ceilometer. 

The driver can be configured to populate these meters with the following options:

1. The floating IP statistics or the interface statistics

2. For a particular virtual machine / instance or all virtual machines / instances

3. For a particular virtual network or all virtual networks. 

The driver is configured using the `resource` field in the ceilometer pipeline configuration file - `pipeline.yaml`. For example, the below configuration in `pipeline.yaml` will configure the driver to populate the meters with the floating IP statistics for all virtual machines with destination virtual network `default-domain:demo:vn3-public`


