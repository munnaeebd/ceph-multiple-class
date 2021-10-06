# cinder-multiple-class

volume-count or list
```
rbd list volumes | wc -l
rbd list volumes-ssd | wc -l
```

## to list pool and set device class
```
ceph osd pool ls
ceph osd crush tree --show-shadow
ceph osd crush rm-device-class osd.20
ceph osd crush set-device-class nvme osd.20
```
## Now we need to be sure ceph doesn't default the devices, so we disable the update on start. This will allow us to modify classes as they won't be set when they come up.

## Modify your local /etc/ceph/ceph.conf to include an osd entry:
```
[osd]
osd_class_update_on_start = false
systemctl restart ceph-osd.target

ceph osd crush rule create-replicated <rule-name> <root> <failure-domain> <class>

### (default ceph root "default") ###

ceph osd crush rule list
ceph osd crush rule dump replicated_rule

ceph osd pool create volumes-ssd 64 replicated_rule-ssd
ceph osd pool set volumes-nvme size 2

ceph osd pool application enable volumes-nvme  rbd

not sure
ceph osd pool create <pool_name> <pg_num> --size=<replica_size>


```

## Before Adding new device:, in admin node it will not set device class automatically
```
vi  /home/ceph-cloud/ceph-cluster/ceph.conf
[osd]
osd_class_update_on_start = false

```

### cinder authentication
```
ceph auth ls
ceph auth get client.cinder

ceph auth caps client.cinder mon 'allow r'  osd 'allow rwx pool=volumes-ssd, allow rwx pool=volumes, allow rwx pool=vms, allow rx pool=images'
```
##Restart cinder volume service in compute node



##change in openstack

after the existing ceph column of the file /etc/cinder/cinder.conf
 
```
[ceph-ssd]
backend_host = ceph-cluster-ssd
volume_driver = cinder.volume.drivers.rbd.RBDDriver
volume_backend_name = ceph-ssd
rbd_cluster_name = ceph
rbd_pool = volumes-ssd
rbd_ceph_conf = /etc/ceph/ceph.conf
rbd_flatten_volume_from_snapshot = false
rbd_max_clone_depth = 5
rbd_store_chunk_size = 4
glance_api_version = 2
rados_connect_timeout = -1
rbd_user = cinder
rbd_secret_uuid = ad2ee2d8-02a5-4d64-b0ca-2a79af888ad8
````
```
openstack volume type create --public CEPH-SSD
openstack volume type set --property volume_backend_name=ceph-ssd CEPH-SSD
```





