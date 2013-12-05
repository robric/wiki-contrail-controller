The metadata service is configured by setting the "linklocal-services" property on the "global-grouter-config" object.

The linklocal-services element should have an entry of the form:
  <linklocal-service-name>metadata</>
  <ip-fabric-service-ip>[server-ip-address]</>
  <ip-fabric-service-port>[server-port]</>

In addition to the link localservice configuration, the agent configuration file in each compute node is expected to contain the metadata service shared secret.
