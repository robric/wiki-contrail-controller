Caveat: These steps only works from systems from within Juniper Networks, Inc.

```
# Use local disk space e.g. /build/$USER
rm -rf BUILD
mkdir -p BUILD
cd BUILD
/cs-shared/tools/bin/repo init --quiet --repo-url=https://github.com/opencontrail-ci-admin/git-repo.git -u git@github.com:Juniper/contrail-vnc-private -m R3.1/ubuntu-14-04/manifest-mitaka.xml
/cs-shared/tools/bin/repo sync
python third_party/fetch_packages.py 
scons -uj8 --optimization=production control-node [contrail-vrouter-agent]
ls -al build/production/control-node/contrail-control build/production/vnsw/agent/contrail/contrail-vrouter-agent
```

