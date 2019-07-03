# OpenStack Manual
This is Manual Instalation guide with step-by-step tutorial to install OpenStack Queens in CentOS with 2 node

### Requirement
- Controller Node
```
Hostname   : zu-controller

Interface  : eth0
IP Address : 10.100.100.10
Netmask    : 255.255.255.0
Gateway    : 10.100.100.1

Interface  : eth1
IP Address : 10.101.101.10
Netmask    : 255.255.255.0
Gateway    : without gateway

DNS Server : 8.8.8.8
```
- Compute Node
```
Hostname   : zu-compute

Interface  : eth0
IP Address : 10.100.100.20
Netmask    : 255.255.255.0
Gateway    : 10.100.100.1

Interface  : eth1
IP Address : 10.101.101.20
Netmask    : 255.255.255.0
Gateway    : without gateway

DNS Server : 8.8.8.8
```

This IP address may change depends on your networking.

### Network Design
- Using 2 network, Management network and External Network
- Management Network is a combination of API network, Data Network, and Managemenet Network itself.

### Services
- This tutorial will install basic services like keystone, nvoa, neutron, and glance.