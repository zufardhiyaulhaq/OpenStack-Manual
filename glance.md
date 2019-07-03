# Glance

Exec on controller node
- Create databases
```
mysql -u root -pDBROOT_PASS
CREATE DATABASE glance;
GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'localhost' \
  IDENTIFIED BY 'GLANCE_DBPASS';
GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'%' \
  IDENTIFIED BY 'GLANCE_DBPASS';
FLUSH PRIVILEGES;
```
- Source adminrc
```
source admin_rc
```
- Create glance user
```
openstack user create --domain default --password GLANCE_PASS glance
openstack role add --project service --user glance admin
```
- Create glance service
```
openstack service create --name glance \
  --description "OpenStack Image" image
```
- Create glance API
```
openstack endpoint create --region RegionOne \
  image public http://10.100.100.10:9292
openstack endpoint create --region RegionOne \
  image internal http://10.100.100.10:9292
openstack endpoint create --region RegionOne \
  image admin http://10.100.100.10:9292
```
- Install package
```
yum install openstack-glance -y
```
- Configuration
```
crudini --set /etc/glance/glance-api.conf database connection mysql+pymysql://glance:GLANCE_DBPASS@10.100.100.10/glance
crudini --set /etc/glance/glance-api.conf keystone_authtoken auth_uri http://10.100.100.10:5000
crudini --set /etc/glance/glance-api.conf keystone_authtoken auth_url http://10.100.100.10:5000
crudini --set /etc/glance/glance-api.conf keystone_authtoken memcached_servers 10.100.100.10:11211
crudini --set /etc/glance/glance-api.conf keystone_authtoken auth_type password
crudini --set /etc/glance/glance-api.conf keystone_authtoken project_domain_name Default
crudini --set /etc/glance/glance-api.conf keystone_authtoken user_domain_name Default
crudini --set /etc/glance/glance-api.conf keystone_authtoken project_name service
crudini --set /etc/glance/glance-api.conf keystone_authtoken username glance
crudini --set /etc/glance/glance-api.conf keystone_authtoken password GLANCE_PASS
crudini --set /etc/glance/glance-api.conf paste_deploy flavor keystone
crudini --set /etc/glance/glance-api.conf glance_store stores file,http
crudini --set /etc/glance/glance-api.conf glance_store default_store file
crudini --set /etc/glance/glance-api.conf glance_store filesystem_store_datadir /var/lib/glance/images/

crudini --set /etc/glance/glance-registry.conf database connection mysql+pymysql://glance:GLANCE_DBPASS@10.100.100.10/glance
crudini --set /etc/glance/glance-registry.conf keystone_authtoken auth_uri http://10.100.100.10:5000
crudini --set /etc/glance/glance-registry.conf keystone_authtoken auth_url http://10.100.100.10:5000
crudini --set /etc/glance/glance-registry.conf keystone_authtoken memcached_servers 10.100.100.10:11211
crudini --set /etc/glance/glance-registry.conf keystone_authtoken auth_type password
crudini --set /etc/glance/glance-registry.conf keystone_authtoken project_domain_name Default
crudini --set /etc/glance/glance-registry.conf keystone_authtoken user_domain_name Default
crudini --set /etc/glance/glance-registry.conf keystone_authtoken project_name service
crudini --set /etc/glance/glance-registry.conf keystone_authtoken username glance
crudini --set /etc/glance/glance-registry.conf keystone_authtoken password GLANCE_PASS
crudini --set /etc/glance/glance-registry.conf paste_deploy flavor keystone
```
- Populate database
```
su -s /bin/sh -c "glance-manage db_sync" glance
```
- Start service
```
systemctl enable openstack-glance-api.service \
  openstack-glance-registry.service
systemctl start openstack-glance-api.service \
  openstack-glance-registry.service
```
- Create image
```
wget http://download.cirros-cloud.net/0.4.0/cirros-0.4.0-x86_64-disk.img
openstack image create "cirros" \
  --file cirros-0.4.0-x86_64-disk.img \
  --disk-format qcow2 --container-format bare \
  --public
openstack image list
```

