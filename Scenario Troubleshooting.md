## VM doesn't ping the default gateway (subnet gateway)
1. Initiate continuous ping from VM to subnet gateway
2. Find tap interface of VM (TODO a. from contrail UI b. agent introspect)
3. tcpdump -ni <tap-intf> - see ICMP requests but no replies received
4. tcpdump -ni pkt0 -X - don't see the packet going to Vrouter-Agent with metadata
5. watch -n1 "dropstats | egrep '[1-9]+'" - observed flow unsable count increases
6. find VRF id for net from introspect (TODO)
7. rt --dump 34 | less observed NH for VJX0 ip is 0 which is discard.
8. Source code check to determine bug in state machine sending from Agent to kernel module.