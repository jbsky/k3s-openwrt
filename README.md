# k3s on OpenWrt
Makefile to generate OpenWrt .opkg packages from official k3s binaries.

## Usage
This requires a custom kernel with support for various cgroup, namespaces, vxlan, cfs
scheduler etc. See here for my openwrt config: https://github.com/5pi-home/openwrt/blob/master/config

### Firewall
To allow the k3s' flannel bridge to access the internet, configure a interface
for cni0 in uci:

/etc/config/network:
```
config interface 'k8s'
	option proto 'none'
	option ifname 'cni0'
```

/etc/config/firewall
```
config zone
        option name 'k8s'
        option input 'ACCEPT'
        option output 'ACCEPT'
        option forward 'ACCEPT'
        option network 'k8s'
```

## Building
Run `make` to build the default version for `x86_64`. You can override ARCH and
VERSION, e.g `make ARCH=armhf`. See ARCHS and VERSIONS files for available
architectures and versions.

## Init
### server
Before run k3s as service, set param with type `'server'`.
```bash
cat > /etc/config/k3s << EOF
config opts 'globals'
   option type 'server'
   option opts '--disable=traefik'
   option root '/var/k3s/data'
EOF
```
Service inits cluster and sets a new `init` parameter to `'1'` in k3s config file.

### agent
Before run k3s as service, set param with type `'agent'`.
```bash
cat > /etc/config/k3s << EOF
config opts 'globals'
   option type 'agent'
   option opts '--token-file /etc/token --server https://<ip or hostname of server1>:6443 --with-node-id'
EOF
```
Service joins automaticaly the server.