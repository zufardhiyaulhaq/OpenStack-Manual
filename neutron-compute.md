# Neutron Compute
- Install package
```
yum install openstack-neutron-openvswitch ebtables ipset
```
- Configure
```
crudini --set /etc/neutron/neutron.conf DEFAULT transport_url rabbit://openstack:RABBIT_PASS@10.100.100.10
crudini --set /etc/neutron/neutron.conf DEFAULT auth_strategy keystone

crudini --set /etc/neutron/neutron.conf keystone_authtoken auth_uri http://10.100.100.10:5000
crudini --set /etc/neutron/neutron.conf keystone_authtoken auth_url http://10.100.100.10:35357
crudini --set /etc/neutron/neutron.conf keystone_authtoken memcached_servers 10.100.100.10:11211
crudini --set /etc/neutron/neutron.conf keystone_authtoken auth_type password
crudini --set /etc/neutron/neutron.conf keystone_authtoken project_domain_name Default
crudini --set /etc/neutron/neutron.conf keystone_authtoken user_domain_name Default
crudini --set /etc/neutron/neutron.conf keystone_authtoken project_name service
crudini --set /etc/neutron/neutron.conf keystone_authtoken username neutron
crudini --set /etc/neutron/neutron.conf keystone_authtoken password NEUTRON_PASS

crudini --set /etc/nova/nova.conf neutron region_name RegionOne
crudini --set /etc/nova/nova.conf neutron project_domain_name Default
crudini --set /etc/nova/nova.conf neutron project_name service
crudini --set /etc/nova/nova.conf neutron auth_type password
crudini --set /etc/nova/nova.conf neutron user_domain_name Default
crudini --set /etc/nova/nova.conf neutron auth_url http://10.100.100.10:35357
crudini --set /etc/nova/nova.conf neutron url http://10.100.100.10:9696
crudini --set /etc/nova/nova.conf neutron username neutron
crudini --set /etc/nova/nova.conf neutron password NEUTRON_PASS

crudini --set /etc/neutron/neutron.conf oslo_concurrency lock_path /var/lib/neutron/tmp

crudini --set /etc/neutron/plugins/ml2/openvswitch_agent.ini ovs bridge_mappings external:br-external
crudini --set /etc/neutron/plugins/ml2/openvswitch_agent.ini ovs local_ip 10.100.100.20
crudini --set /etc/neutron/plugins/ml2/openvswitch_agent.ini agent tunnel_types vxlan
crudini --set /etc/neutron/plugins/ml2/openvswitch_agent.ini agent l2_population True
```
- Create openvswitch bridge
You can create this bridge as persistent by creating interface config files, but this tutorial using ovs-vsctl command
```
systemctl start neutron-openvswitch-agent
ovs-vsctl add-br br-external
ovs-vsctl add-port br-external eth1
ovs-vsctl show
```
- Restart nova compute
```
systemctl restart openstack-nova-compute.service
```
- Enable and restart
```
systemctl enable neutron-openvswitch-agent.service
systemctl start neutron-openvswitch-agent.service
```
- Verify
Run on controller node
```
openstack network agent list
```