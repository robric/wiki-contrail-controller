1. Checkout code using instructions in [contrail-dev-env](https://github.com/Juniper/contrail-dev-env)

2. Most of the code is in here controller/src/bgp and controller/src/xmpp

```
# Go to one of build server
ssh $one_of_build_server

# Create a sandbox (Note: /build space is local and not backed up)
mkdir -p /build/$USER/fresh-sandbox
cd /build/$USER/fresh-sandbox

# Checkout code
/usr/local/bin/repo init --quiet -u git@github.com:Juniper/contrail-vnc
/usr/local/bin/repo sync
python third_party/fetch_packages.py

# Checkout your branch
cd controller/src/control-node
git checkout -b master github/master
cd /build/$USER/fresh-sandbox

# Build unit test
BUILD_ONLY=TRUE NO_HEAPCHECK=TRUE scons -uj32 --optimization=debug src/bgp:bgp_server_test

# Run the unit test
./build/debug/bgp/test/bgp_server_test

```


