# Prerequisites
- User and Password

| Password       | Description     |
| :------------- | :----------: |
| DBROOT_PASS | Root password for the database |
|  ADMIN_PASS | Password of user admin |
|  GLANCE_PASS | Password of Image service user glance |
|  GLANCEDB_PASS | Database password for Image service |
| KEYSTONE_DBPASS | Database password of Identity service | 
|  NEUTRON_PASS | Password of Networking service user neutron |
|  NEURONDB_PASS | Database password for the Networking service |
|  NOVA_PASS | Password of Compute service user nova |
|  NOVADB_PASS | Database password for Compute service |
| PLACEMENT_PASS | Password of the Placement service user placement |
|  RABBIT_PASS | Password of RabbitMQ user openstack |
|  RABBIT_PASS | Password of RabbitMQ user openstack |

### Hosts Mapping
```
nano /etc/hosts
```
```
10.100.100.10 zu-controller
10.100.100.20 zu-compute
```
and verification
```
ping -c 2 zu-controller
ping -c 2 zu-compute
ping -c 2 10.101.101.10
ping -c 2 10.101.101.20
ping -c 2 google.com
```
### Repository
```
yum -y update
yum -y install centos-release-openstack-queens epel-release
yum repolist
yum -y update
```
### Utility Package
```
yum -y install vim nano wget screen crudini htop
```
### NTP
```
yum -y install chrony
systemctl enable chronyd.service
systemctl restart chronyd.service
systemctl status chronyd.service
chronyc sources
```
### Firewall
We disable firewall and install openstack-selinux for simplicity
```
yum -y install openstack-selinux
systemctl stop firewalld.service
systemctl disable firewalld.service
systemctl status firewalld.service
```
### OpenStack Client
```
yum -y install python-openstackclient
```
### SQL database
Install only on controller node
- Install Package
```
yum -y install mariadb mariadb-server python2-PyMySQL
```
- Create openstack configuration database
```
nano /etc/my.cnf.d/openstack.cnf

[mysqld]
bind-address = 10.100.100.10

default-storage-engine = innodb
innodb_file_per_table = on
max_connections = 4096
collation-server = utf8_general_ci
character-set-server = utf8
```
- Enable and start services
```
systemctl enable mariadb.service
systemctl start mariadb.service
```
- Secure by selecting root password
```
mysql_secure_installation
```
### Message Queue
Install only on controller node
- Install package
```
yum -y install rabbitmq-server
```
- Enable and Start
```
systemctl enable rabbitmq-server.service
systemctl start rabbitmq-server.service
```
- Add Openstack user
```
rabbitmqctl add_user openstack RABBIT_PASS
rabbitmqctl set_permissions openstack ".*" ".*" ".*"
```
### Memcached
Install only on controller node
- Install Package
``` 
yum -y install memcached python-memcached
```
- Edit configuration files
```
nano /etc/sysconfig/memcached
OPTIONS="-l 127.0.0.1,::1,10.100.100.10"
```
- Start service
```
systemctl enable memcached.service
systemctl start memcached.service
```
### Etcd
Install only on controller node
- Install package
```
yum install etcd -y
```
- Configuration
```
nano /etc/etcd/etcd.conf 

[Member]
ETCD_DATA_DIR="/var/lib/etcd/default.etcd"
ETCD_LISTEN_PEER_URLS="http://10.100.100.10:2380"
ETCD_LISTEN_CLIENT_URLS="http://10.100.100.10:2379"
ETCD_NAME="controller"

[Clustering]
ETCD_INITIAL_ADVERTISE_PEER_URLS="http://10.100.100.10:2380"
ETCD_ADVERTISE_CLIENT_URLS="http://10.100.100.10:2379"
ETCD_INITIAL_CLUSTER="controller=http://10.100.100.10:2380"
ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster-01"
ETCD_INITIAL_CLUSTER_STATE="new"
```
- Start service
```
systemctl enable etcd
systemctl start etcd
```



