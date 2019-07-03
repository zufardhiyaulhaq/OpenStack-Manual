# Nova Controller
- Create database
```
mysql -u root -pDBROOT_PASS
CREATE DATABASE nova_api;
CREATE DATABASE nova;
CREATE DATABASE nova_cell0;
GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'localhost' \
  IDENTIFIED BY 'NOVA_DBPASS';
GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'%' \
  IDENTIFIED BY 'NOVA_DBPASS';
GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'localhost' \
  IDENTIFIED BY 'NOVA_DBPASS';
GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'%' \
  IDENTIFIED BY 'NOVA_DBPASS';
GRANT ALL PRIVILEGES ON nova_cell0.* TO 'nova'@'localhost' \
  IDENTIFIED BY 'NOVA_DBPASS';
GRANT ALL PRIVILEGES ON nova_cell0.* TO 'nova'@'%' \
  IDENTIFIED BY 'NOVA_DBPASS';
FLUSH PRIVILEGES;
```
- Create user nova
```
source admin_rc
openstack user create --domain default --password NOVA_PASS nova
openstack role add --project service --user nova admin
```
- Create Service nova
```
openstack service create --name nova \
  --description "OpenStack Compute" compute
```
- Create nova endpoint
```
openstack endpoint create --region RegionOne \
  compute public http://10.100.100.10:8774/v2.1
openstack endpoint create --region RegionOne \
  compute internal http://10.100.100.10:8774/v2.1
openstack endpoint create --region RegionOne \
  compute admin http://10.100.100.10:8774/v2.1
```
- Create placement service user
```
openstack user create --domain default --password PLACEMENT_PASS placement
openstack role add --project service --user placement admin
```
- Create placement service
```
openstack service create --name placement --description "Placement API" placement
```
- Create placement Endpoint
```
openstack endpoint create --region RegionOne placement public http://10.100.100.10:8778
openstack endpoint create --region RegionOne placement internal http://10.100.100.10:8778
openstack endpoint create --region RegionOne placement admin http://10.100.100.10:8778
```
- Install package
```
yum install openstack-nova-api openstack-nova-conductor \
  openstack-nova-console openstack-nova-novncproxy \
  openstack-nova-scheduler openstack-nova-placement-api -y
```
- Configuration
```
crudini --set /etc/nova/nova.conf DEFAULT enabled_apis osapi_compute,metadata
crudini --set /etc/nova/nova.conf DEFAULT transport_url rabbit://openstack:RABBIT_PASS@10.100.100.10
crudini --set /etc/nova/nova.conf DEFAULT my_ip 10.100.100.10
crudini --set /etc/nova/nova.conf DEFAULT use_neutron True
crudini --set /etc/nova/nova.conf DEFAULT firewall_driver nova.virt.firewall.NoopFirewallDriver

crudini --set /etc/nova/nova.conf vnc enabled True
crudini --set /etc/nova/nova.conf vnc server_listen 10.100.100.10
crudini --set /etc/nova/nova.conf vnc server_proxyclient_address 10.100.100.10

crudini --set /etc/nova/nova.conf glance api_servers http://10.100.100.10:9292
crudini --set /etc/nova/nova.conf oslo_concurrency lock_path /var/lib/nova/tmp

crudini --set /etc/nova/nova.conf placement os_region_name RegionOne
crudini --set /etc/nova/nova.conf placement project_domain_name Default
crudini --set /etc/nova/nova.conf placement project_name service
crudini --set /etc/nova/nova.conf placement auth_type password
crudini --set /etc/nova/nova.conf placement user_domain_name Default
crudini --set /etc/nova/nova.conf placement auth_url http://10.100.100.10:5000/v3
crudini --set /etc/nova/nova.conf placement username placement
crudini --set /etc/nova/nova.conf placement password PLACEMENT_PASS

crudini --set /etc/nova/nova.conf api_database connection mysql+pymysql://nova:NOVA_DBPASS@10.100.100.10/nova_api
crudini --set /etc/nova/nova.conf database connection mysql+pymysql://nova:NOVA_DBPASS@10.100.100.10/nova

crudini --set /etc/nova/nova.conf api auth_strategy keystone

crudini --set /etc/nova/nova.conf keystone_authtoken auth_url http://10.100.100.10:5000/v3
crudini --set /etc/nova/nova.conf keystone_authtoken memcached_servers 10.100.100.10:11211
crudini --set /etc/nova/nova.conf keystone_authtoken auth_type password
crudini --set /etc/nova/nova.conf keystone_authtoken project_domain_name Default
crudini --set /etc/nova/nova.conf keystone_authtoken user_domain_name Default
crudini --set /etc/nova/nova.conf keystone_authtoken project_name service
crudini --set /etc/nova/nova.conf keystone_authtoken username nova
crudini --set /etc/nova/nova.conf keystone_authtoken password NOVA_PASS
```
- Edit Placement API
```
nano /etc/httpd/conf.d/00-nova-placement-api.conf

<Directory /usr/bin>
   <IfVersion >= 2.4>
      Require all granted
   </IfVersion>
   <IfVersion < 2.4>
      Order allow,deny
      Allow from all
   </IfVersion>
</Directory>
```
- Restart httpd
```
systemctl restart httpd
```
- Populate nova-api
```
su -s /bin/sh -c "nova-manage api_db sync" nova
```
- Register cell0 database
```
su -s /bin/sh -c "nova-manage cell_v2 map_cell0" nova
```
- Create cell1 cell
```
su -s /bin/sh -c "nova-manage cell_v2 create_cell --name=cell1 --verbose" nova
```
- Populate nova database
```
su -s /bin/sh -c "nova-manage db sync" nova
```
- Verify nova cell0 and cell1
```
nova-manage cell_v2 list_cells
```
- Start service
```
systemctl enable openstack-nova-api.service \
  openstack-nova-consoleauth.service openstack-nova-scheduler.service \
  openstack-nova-conductor.service openstack-nova-novncproxy.service
systemctl start openstack-nova-api.service \
  openstack-nova-consoleauth.service openstack-nova-scheduler.service \
  openstack-nova-conductor.service openstack-nova-novncproxy.service
systemctl status openstack-nova-api.service \
  openstack-nova-consoleauth.service openstack-nova-scheduler.service \
  openstack-nova-conductor.service openstack-nova-novncproxy.service
```
