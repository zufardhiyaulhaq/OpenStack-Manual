# Keystone

Exec on controller node
- Create databases
```
mysql -u root -pDBROOT_PASS
CREATE DATABASE keystone;
GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'localhost' \
IDENTIFIED BY 'KEYSTONE_DBPASS';
GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'%' \
IDENTIFIED BY 'KEYSTONE_DBPASS';
FLUSH PRIVILEGES;
```
- Install package
```
yum install openstack-keystone httpd mod_wsgi -y
```
- Edit configuration
```
crudini --set /etc/keystone/keystone.conf database connection mysql+pymysql://keystone:KEYSTONE_DBPASS@10.100.100.10/keystone
crudini --set /etc/keystone/keystone.conf token provider fernet
```
- Populate database
```
su -s /bin/sh -c "keystone-manage db_sync" keystone
```
- Init fernet key
```
keystone-manage fernet_setup --keystone-user keystone --keystone-group keystone
keystone-manage credential_setup --keystone-user keystone --keystone-group keystone
```
- Bootstrap keystone
```
keystone-manage bootstrap --bootstrap-password ADMIN_PASS \
  --bootstrap-admin-url http://10.100.100.10:5000/v3/ \
  --bootstrap-internal-url http://10.100.100.10:5000/v3/ \
  --bootstrap-public-url http://10.100.100.10:5000/v3/ \
  --bootstrap-region-id RegionOne
```
- Configure httpd
```
nano  /etc/httpd/conf/httpd.conf
ServerName zu-controller
```
- Create link
```
ln -s /usr/share/keystone/wsgi-keystone.conf /etc/httpd/conf.d/
```
- Start
```
systemctl enable httpd.service
systemctl start httpd.service
```
- Create Environment files
```
cat << EOF >> admin_rc
export OS_USERNAME=admin
export OS_PASSWORD=ADMIN_PASS
export OS_PROJECT_NAME=admin
export OS_USER_DOMAIN_NAME=Default
export OS_PROJECT_DOMAIN_NAME=Default
export OS_AUTH_URL=http://zu-controller:5000/v3
export OS_IDENTITY_API_VERSION=3
EOF
source admin_rc
```
- Verify
```
openstack --os-auth-url http://controller:35357/v3 \
  --os-project-domain-name Default --os-user-domain-name Default \
  --os-project-name admin --os-username admin token issue
```
- Create service project
```
openstack project create --domain default \
  --description "Service Project" service
```

