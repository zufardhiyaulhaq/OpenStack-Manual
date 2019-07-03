# Horizon / Dashboard
- Install Package
```
yum install openstack-dashboard -y
```
- Edit configuration
```
nano /etc/openstack-dashboard/local_settings

OPENSTACK_HOST = "10.100.100.10"
ALLOWED_HOSTS = ['*']
SESSION_ENGINE = 'django.contrib.sessions.backends.cache'

CACHES = {
    'default': {
         'BACKEND': 'django.core.cache.backends.memcached.MemcachedCache',
         'LOCATION': '10.100.100.10:11211',
    }
}
OPENSTACK_KEYSTONE_URL = "http://%s:5000/v3" % OPENSTACK_HOST
OPENSTACK_KEYSTONE_MULTIDOMAIN_SUPPORT = True
OPENSTACK_API_VERSIONS = {
    "identity": 3,
    "image": 2,
    "volume": 2,
}
OPENSTACK_KEYSTONE_DEFAULT_ROLE = "user"
```
```
nano /etc/httpd/conf.d/openstack-dashboard.conf

WSGIApplicationGroup %{GLOBAL}
```
- Restart web service
```
systemctl restart httpd.service memcached.service
```