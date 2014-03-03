It might be required for a virtual machine to access specific services running on the fabric infrastructure. For example, a VM might be a nova client requiring access to the nova api service running in the fabric. Such access can be provided by configuring the required service as a link local service.

A link local address (169.254.x.x and a service port) is chosen for the specific service running on a TCP / UDP port on a server in the fabric. Once the link local service is configured, virtual machines can access the service using the link local address.

Link local service can be configured using the webui (Configure -> Infrastructure -> Link Local Services).