1. Checkout code using instructions in [contrail-dev-env](https://github.com/Juniper/contrail-dev-env)

2. Most of the code is in here controller/src/bgp and controller/src/xmpp

```
mkdir -p /build/$USER/fresh-sandbox
cd /build/$USER/fresh-sandbox
/usr/local/bin/repo init --quiet -u git@github.com:Juniper/contrail-vnc
/usr/local/bin/repo sync
python third_party/fetch_packages.py
BUILD_ONLY=TRUE NO_HEAPCHECK=TRUE scons -uj32 --optimization=debug src/bgp:bgp_server_test
```


