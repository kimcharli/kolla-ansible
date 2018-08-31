# All-in-one on BMS case

## Reference

https://docs.openstack.org/kolla-ansible/latest/user/quickstart.html


## Setup installer container

create and login to the container
```bash
ckim-mbp:PycharmProjects ckim$ docker run -itd --name centos-kolla centos:7

ckim-mbp:PycharmProjects ckim$ docker exec -it centos-kolla bash
```

setup packages
```bash
[root@3640f9d7073e /]#
[root@3640f9d7073e /]# yum -y update

[root@3640f9d7073e /]# yum -y install epel-release

[root@3640f9d7073e /]# yum -y install python-pip python-devel libffi-devel gcc openssl-devel libselinux-python git

[root@3640f9d7073e /]# pip install --upgrade --force-reinstall pip==9.0.3

[root@3640f9d7073e /]# pip install oslo.config

[root@3640f9d7073e /]# pip install --upgrade pip

[root@3640f9d7073e /]# yum -y install ansible

[root@3640f9d7073e /]# ansible --version
ansible 2.6.3
  config file = /etc/ansible/ansible.cfg
  configured module search path = [u'/root/.ansible/plugins/modules', u'/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python2.7/site-packages/ansible
  executable location = /usr/bin/ansible
  python version = 2.7.5 (default, Jul 13 2018, 13:06:57) [GCC 4.8.5 20150623 (Red Hat 4.8.5-28)]
[root@3640f9d7073e /]#

```

create ssh key and setup passworless to the target server (10.85.192.10 here)
```bash
[root@3640f9d7073e /]# ssh-keygen

[root@3640f9d7073e /]# ssh-keygen -R 10.85.192.10
# Host 10.85.192.10 found: line 1
/root/.ssh/known_hosts updated.
Original contents retained as /root/.ssh/known_hosts.old
[root@3640f9d7073e /]# ssh-copy-id root@10.85.192.10
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/root/.ssh/id_rsa.pub"
The authenticity of host '10.85.192.10 (10.85.192.10)' can't be established.
ECDSA key fingerprint is SHA256:l/dhZhNQMa9WFGiVKS3CKfJflDqPgTLq15eE/yJWxNU.
ECDSA key fingerprint is MD5:b9:cf:d7:d2:9d:36:65:e8:b9:3a:82:3c:bd:88:d1:90.
Are you sure you want to continue connecting (yes/no)? yes
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
root@10.85.192.10's password:

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'root@10.85.192.10'"
and check to make sure that only the key(s) you wanted were added.

[root@3640f9d7073e /]#


[root@3640f9d7073e /]# ssh-copy-id root@server7.pslab.net

TODO: try with non-root user contrail
[root@3640f9d7073e /]# ssh-copy-id contrail@server7.pslab.net

```

TODO: In case of non-root user (contrail here), need to set below.
```bash
[root@3640f9d7073e /]# ssh-copy-id contrail@server7.pslab.net
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/root/.ssh/id_rsa.pub"
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
contrail@server7.pslab.net's password:

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'contrail@server7.pslab.net'"
and check to make sure that only the key(s) you wanted were added.

[root@3640f9d7073e /]# ssh contrail@server7.pslab.net
Last login: Wed Aug 29 10:47:28 2018
[contrail@server7 ~]$ echo 'contrail ALL=(ALL) NOPASSWD:ALL' | sudo tee /etc/sudoers.d/contrail
[sudo] password for contrail:
contrail ALL=(ALL) NOPASSWD:ALL
[contrail@server7 ~]$ exit
logout
Connection to server7.pslab.net closed.
[root@3640f9d7073e /]#
```


Setup kolla-ansible.
```bash
[root@3640f9d7073e ~]# pip install kolla-ansible

```

create inventory file all-in-one.
```bash
ckim-mbp:kolla-ansible ckim$ docker cp server7/all-in-one centos-kolla:/all-in-one

```

create /etc/kolla
```bash
[root@3640f9d7073e ~]# cp -r /usr/share/kolla-ansible/etc_examples/kolla /etc/
```

update globals.yml file
```bash
ckim-mbp:kolla-ansible ckim$ docker cp server7/globals.yml centos-kolla:/etc/kolla/globals.yml

```

check connectivity
```bash
[root@3640f9d7073e /]# ansible all -m ping -i all-in-one
server7 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
[root@3640f9d7073e /]#

```

## Generate passwords
```bash
[root@3640f9d7073e /]# kolla-genpwd

```

## bootstrap servers
```bash
[root@3640f9d7073e /]# kolla-ansible -i all-in-one bootstrap-servers

```

It was stalling at
```
TASK [baremetal : Synchronizing time one-time]
```

Updated ntp as below, and moved on.
```bash
ckim-mbp:portal ckim$ ssh root@server7.pslab.net

[root@server7 ~]# ps -ef | grep ntp
root     14834 14833  0 11:11 pts/0    00:00:00 ntpd -gq
root     14875 14858  0 11:13 pts/1    00:00:00 grep --color=auto ntp

[root@server7 ~]# vi /etc/ntp.conf

server 10.85.130.130

[root@server7 ~]# kill -HUP 14834

```



## precheks
```bash
[root@3640f9d7073e /]# kolla-ansible -i all-in-one prechecks
Pre-deployment checking : ansible-playbook -i all-in-one -e @/etc/kolla/globals.yml -e @/etc/kolla/passwords.yml -e CONFIG_DIR=/etc/kolla  -e action=precheck /usr/share/kolla-ansible/ansible/site.yml
 [WARNING]: Found variable using reserved name: action

```

## deploy
```bash
[root@3640f9d7073e /]# kolla-ansible -i all-in-one deploy
Deploying Playbooks : ansible-playbook -i all-in-one -e @/etc/kolla/globals.yml -e @/etc/kolla/passwords.yml -e CONFIG_DIR=/etc/kolla  -e action=deploy /usr/share/kolla-ansible/ansible/site.yml
 [WARNING]: Found variable using reserved name: action

```

Occasionally image pull fails. workaround is to pull them manually from the target server, and re-deploy
```bash
[contrail@server7 ~]$ docker pull kolla/centos-source-cron:queens

```

Found that some containers are kept in restarting.
```bash
[root@server7 nova-compute]# docker ps
CONTAINER ID        IMAGE                                           COMMAND             CREATED             STATUS                          PORTS               NAMES
66f00c52ec61        kolla/centos-source-nova-consoleauth:queens     "kolla_start"       47 minutes ago      Restarting (1) 19 minutes ago                       nova_consoleauth
2b0915aaba1e        kolla/centos-source-nova-conductor:queens       "kolla_start"       47 minutes ago      Restarting (1) 19 minutes ago                       nova_conductor
1d7257b2664b        kolla/centos-source-nova-scheduler:queens       "kolla_start"       47 minutes ago      Restarting (1) 19 minutes ago                       nova_scheduler
3c5f279cb63c        kolla/centos-source-nova-api:queens             "kolla_start"       47 minutes ago      Restarting (1) 6 seconds ago                        nova_api
3a4568a237c5        kolla/centos-source-nova-compute:queens         "kolla_start"       About an hour ago   Up 47 seconds                                       nova_compute
4d3f2aa36b6f        kolla/centos-source-nova-novncproxy:queens      "kolla_start"       About an hour ago   Up About an hour                                    nova_novncproxy
97ef07fced2c        kolla/centos-source-nova-placement-api:queens   "kolla_start"       About an hour ago   Up About an hour                                    placement_api
...
[root@server7 nova-compute]# docker rm -f nova_consoleauth nova_conductor nova_scheduler nova_api nova_compute

```

Then try this and redo deploy.
```bash
[root@server7 ~]# docker ps | awk '{ system( "docker rm -f " $1 ) }' ;\
docker images | awk '{ system ( "docker rmi " $3 ) }' ;\
systemctl stop docker ;\
rm -rf /var/lib/docker/volumes/ ;\
systemctl start docker

```

```bash
kolla-ansible pull
kolla-ansible destroy -i <<inventory-file>>

```

some cases
```bash
mkdir -p /etc/docker
tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://registry.docker-cn.com"]
}
EOF
systemctl restart docker

tools/cleanup-containers

tools/cleanup-host

kolla-ansible -i INVENTORY mariadb_recovery

kolla-ansible -i INVENTORY post-deploy

kolla-ansible -i INVENTORY reconfigure is used to reconfigure OpenStack service.

```

## post-deploy

```bash
[root@3640f9d7073e /]# kolla-ansible -i all-in-one post-deploy

[root@3640f9d7073e /]# ls /etc/kolla/admin-openrc.sh
/etc/kolla/admin-openrc.sh
[root@3640f9d7073e /]#
[root@3640f9d7073e /]# scp /etc/kolla/admin-openrc.sh root@server7.pslab.net:/etc/kolla/kolla-toolbox/
admin-openrc.sh                                                                                                                                                                               100%  388     4.3KB/s   00:00
[root@3640f9d7073e /]#
```

get admin password
```bash
[root@3640f9d7073e /]# grep admin /etc/kolla/passwords.yml
grafana_admin_password: PyqCj3QJHFdKwr5s9utzK5IzCHddnh4Uzt1UMUMc
heat_domain_admin_password: 7oWAqSYpN22A70lrASbHD2lIRv5vuSyelq10nQf4
keystone_admin_password: ixCD3lnggpaENk4yJZUVhILVt6WsNpOTkByNidSQ
[root@3640f9d7073e /]#
```



## test

```bash
[root@server7 ~]# docker ps
CONTAINER ID        IMAGE                                                  COMMAND             CREATED             STATUS              PORTS               NAMES
b55ef75d20e5        kolla/centos-source-horizon:queens                     "kolla_start"       24 minutes ago      Up 24 minutes                           horizon
4babe327e128        kolla/centos-source-heat-engine:queens                 "kolla_start"       25 minutes ago      Up 25 minutes                           heat_engine
da3ad122ee18        kolla/centos-source-heat-api-cfn:queens                "kolla_start"       25 minutes ago      Up 25 minutes                           heat_api_cfn
e2f24d10e349        kolla/centos-source-heat-api:queens                    "kolla_start"       25 minutes ago      Up 25 minutes                           heat_api
83bb5d54c47d        kolla/centos-source-neutron-metadata-agent:queens      "kolla_start"       28 minutes ago      Up 27 minutes                           neutron_metadata_agent
c6aaa44c72bc        kolla/centos-source-neutron-l3-agent:queens            "kolla_start"       28 minutes ago      Up 28 minutes                           neutron_l3_agent
e888d91c2a8b        kolla/centos-source-neutron-dhcp-agent:queens          "kolla_start"       28 minutes ago      Up 28 minutes                           neutron_dhcp_agent
b7dd7d636a46        kolla/centos-source-neutron-openvswitch-agent:queens   "kolla_start"       28 minutes ago      Up 28 minutes                           neutron_openvswitch_agent
53c1b53056cd        kolla/centos-source-neutron-server:queens              "kolla_start"       28 minutes ago      Up 28 minutes                           neutron_server
9a83b8f91c1c        kolla/centos-source-openvswitch-vswitchd:queens        "kolla_start"       33 minutes ago      Up 33 minutes                           openvswitch_vswitchd
6bff2d6024b2        kolla/centos-source-openvswitch-db-server:queens       "kolla_start"       33 minutes ago      Up 33 minutes                           openvswitch_db
775577c4617f        kolla/centos-source-nova-compute:queens                "kolla_start"       36 minutes ago      Up 36 minutes                           nova_compute
9af0f09eb437        kolla/centos-source-nova-novncproxy:queens             "kolla_start"       37 minutes ago      Up 37 minutes                           nova_novncproxy
b86b2a321f52        kolla/centos-source-nova-consoleauth:queens            "kolla_start"       37 minutes ago      Up 37 minutes                           nova_consoleauth
f347676f2185        kolla/centos-source-nova-conductor:queens              "kolla_start"       37 minutes ago      Up 37 minutes                           nova_conductor
ea1ee4552987        kolla/centos-source-nova-scheduler:queens              "kolla_start"       37 minutes ago      Up 37 minutes                           nova_scheduler
b7b9f50ffdf3        kolla/centos-source-nova-api:queens                    "kolla_start"       37 minutes ago      Up 37 minutes                           nova_api
3e65b7ad35ef        kolla/centos-source-nova-placement-api:queens          "kolla_start"       37 minutes ago      Up 37 minutes                           placement_api
be9776b06df2        kolla/centos-source-nova-libvirt:queens                "kolla_start"       37 minutes ago      Up 37 minutes                           nova_libvirt
8c17221d3174        kolla/centos-source-nova-ssh:queens                    "kolla_start"       38 minutes ago      Up 38 minutes                           nova_ssh
bf2ee76d2329        kolla/centos-source-cinder-backup:queens               "kolla_start"       44 minutes ago      Up 44 minutes                           cinder_backup
e35ab5ac10a7        kolla/centos-source-cinder-volume:queens               "kolla_start"       44 minutes ago      Up 44 minutes                           cinder_volume
037ba6c6a3cd        kolla/centos-source-cinder-scheduler:queens            "kolla_start"       44 minutes ago      Up 44 minutes                           cinder_scheduler
e5c8b1fd7abc        kolla/centos-source-cinder-api:queens                  "kolla_start"       45 minutes ago      Up 45 minutes                           cinder_api
c06e5ae58b48        kolla/centos-source-glance-registry:queens             "kolla_start"       47 minutes ago      Up 47 minutes                           glance_registry
8da5833cd2d6        kolla/centos-source-glance-api:queens                  "kolla_start"       47 minutes ago      Up 47 minutes                           glance_api
cf15e97e7069        kolla/centos-source-keystone:queens                    "kolla_start"       49 minutes ago      Up 49 minutes                           keystone
cbb3a61f44c0        kolla/centos-source-rabbitmq:queens                    "kolla_start"       51 minutes ago      Up 51 minutes                           rabbitmq
3c149028fbeb        kolla/centos-source-tgtd:queens                        "kolla_start"       52 minutes ago      Up 52 minutes                           tgtd
d8d709911a3e        kolla/centos-source-iscsid:queens                      "kolla_start"       52 minutes ago      Up 52 minutes                           iscsid
3adbcd492a8a        kolla/centos-source-mariadb:queens                     "kolla_start"       53 minutes ago      Up 53 minutes                           mariadb
efd953eae1dd        kolla/centos-source-memcached:queens                   "kolla_start"       54 minutes ago      Up 54 minutes                           memcached
c60bb14c8079        kolla/centos-source-chrony:queens                      "kolla_start"       55 minutes ago      Up 55 minutes                           chrony
8fb6bf25a12a        kolla/centos-source-cron:queens                        "kolla_start"       55 minutes ago      Up 55 minutes                           cron
63494e2c6aa6        kolla/centos-source-kolla-toolbox:queens               "kolla_start"       55 minutes ago      Up 55 minutes                           kolla_toolbox
89591da5215f        kolla/centos-source-fluentd:queens                     "kolla_start"       56 minutes ago      Up 56 minutes                           fluentd
[root@server7 ~]#
[root@server7 ~]# docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
53feca39482b        bridge              bridge              local
3033f1d47b5b        host                host                local
bc09499c6288        none                null                local
[root@server7 ~]#
[root@server7 ~]# ip r
default via 10.85.192.1 dev eno1 proto static metric 100
10.85.192.0/26 dev eno1 proto kernel scope link src 10.85.192.10 metric 100
172.17.0.0/16 dev docker0 proto kernel scope link src 172.17.0.1
[root@server7 ~]#
[root@server7 ~]# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eno1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 1c:98:ec:28:8c:d0 brd ff:ff:ff:ff:ff:ff
    inet 10.85.192.10/26 brd 10.85.192.63 scope global noprefixroute eno1
       valid_lft forever preferred_lft forever
    inet6 fe80::7f4d:f159:2163:6782/64 scope link noprefixroute
       valid_lft forever preferred_lft forever

14: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default
    link/ether 02:42:cf:66:c9:99 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 scope global docker0
       valid_lft forever preferred_lft forever
15: ovs-system: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether 76:6f:f4:dc:22:43 brd ff:ff:ff:ff:ff:ff
16: br-ex: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether 14:02:ec:74:f3:0c brd ff:ff:ff:ff:ff:ff
19: br-int: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether d2:91:b6:01:fa:46 brd ff:ff:ff:ff:ff:ff
20: br-tun: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether 46:e9:ca:68:eb:43 brd ff:ff:ff:ff:ff:ff
[root@server7 ~]#

```



```bash
[root@server7 ~]# cd /etc/kolla/kolla-toolbox/
[root@server7 kolla-toolbox]# curl http://download.cirros-cloud.net/0.4.0/cirros-0.4.0-x86_64-disk.img -o cirros-0.4.0-x86_64-disk.img
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 12.1M  100 12.1M    0     0  2824k      0  0:00:04  0:00:04 --:--:-- 2881k
[root@server7 kolla-toolbox]# file cirros-0.4.0-x86_64-disk.img
cirros-0.4.0-x86_64-disk.img: QEMU QCOW Image (v3), 46137344 bytes
[root@server7 kolla-toolbox]#
[root@server7 kolla-toolbox]# docker exec -it kolla_toolbox bash
(kolla-toolbox)[ansible@server7 /]$ stty cols 240
(kolla-toolbox)[ansible@server7 /]$
(kolla-toolbox)[ansible@server7 /]$ cd /var/lib/kolla/config_files/
(kolla-toolbox)[ansible@server7 /var/lib/kolla/config_files]$
(kolla-toolbox)[ansible@server7 /var/lib/kolla/config_files]$ source admin-openrc.sh 
Please enter your OpenStack Password for project admin as user admin:
(kolla-toolbox)[ansible@server7 /var/lib/kolla/config_files]$
(kolla-toolbox)[ansible@server7 /var/lib/kolla/config_files]$ openstack image create --disk-format qcow2 --container-format bare --public --file cirros-0.4.0-x86_64-disk.img cirros

(kolla-toolbox)[ansible@server7 /var/lib/kolla/config_files]$ openstack flavor create --ram 512 --disk 1 --vcpus 1 --public tiny

openstack extension list -c Alias -c Name --network

openstack network create net1

openstack subnet create subnet1 --network net1 --subnet-range 192.0.2.0/24

```




```bash

openstack image create --disk-format qcow2 --container-format bare --public --file junos-x86-64-18.2R1.9.img vrr-18.2R1.9openstack image create --disk-format qcow2 --container-format bare --public --file junos-x86-64-18.2R1.9.img vrr-18.2R1.9

openstack image create --disk-format qcow2 --container-format bare --public --file junos-x86-64-17.3R3.9.img vrr-17.3R3.9

```



```bash
[root@server7 ~]# less /var/lib/docker/volumes/kolla_logs/_data/nova/nova-scheduler.log

```


error
```bash
(kolla-toolbox)[ansible@server7 /var/lib/kolla/config_files]$ openstack server show cirros3 -f json -c fault
{
  "fault": {
    "message": "Exceeded maximum number of retries. Exhausted all hosts available for retrying build failures for instance d9896325-8e39-4795-80e3-962f91954e91.",
    "code": 500,
    "details": "  File \"/var/lib/kolla/venv/lib/python2.7/site-packages/nova/conductor/manager.py\", line 580, in build_instances\n    raise exception.MaxRetriesExceeded(reason=msg)\n",
    "created": "2018-08-30T19:56:39Z"
  }
}
(kolla-toolbox)[ansible@server7 /var/lib/kolla/config_files]$

```

```bash
[root@server7 ~]# grep ERROR /var/lib/docker/volumes/kolla_logs/_data/nova/nova-compute.log | tail -2
2018-08-30 15:56:37.546 7 ERROR nova.compute.manager [instance: d9896325-8e39-4795-80e3-962f91954e91] 2018-08-30T19:56:36.775933Z qemu-kvm: failed to initialize KVM: Permission denied
2018-08-30 15:56:37.546 7 ERROR nova.compute.manager [instance: d9896325-8e39-4795-80e3-962f91954e91]
[root@server7 ~]#
[root@server7 ~]# ls -l /dev/kvm
crw-rw----+ 1 root 42427 10, 232 Aug 29 10:35 /dev/kvm
[root@server7 ~]#
[root@server7 ~]# cat /sys/module/kvm_intel/parameters/nested
N
[root@server7 ~]#
[root@server7 ~]# echo 'options kvm_intel nested=1' >> /etc/modprobe.d/kvm.conf
[root@server7 ~]# rmmod kvm_intel
[root@server7 ~]# modprobe kvm_intel
[root@server7 ~]# cat /sys/module/kvm_intel/parameters/nested
Y
[root@server7 ~]#
[root@server7 ~]# cat /usr/lib/udev/rules.d/81-kvm-rhel.rules
DEVPATH=="*/kvm", ACTION=="change", RUN+="/lib/udev/udev-kvm-check $env{COUNT} $env{EVENT}"
[root@server7 ~]#

[root@server7 ~]# grep -r kvm /etc/kolla/
/etc/kolla/nova-conductor/nova.conf:virt_type = kvm
/etc/kolla/nova-api/nova.conf:virt_type = kvm
/etc/kolla/nova-scheduler/nova.conf:virt_type = kvm
/etc/kolla/nova-novncproxy/nova.conf:virt_type = kvm
/etc/kolla/placement-api/nova.conf:virt_type = kvm
/etc/kolla/nova-compute/nova.conf:virt_type = kvm
/etc/kolla/nova-consoleauth/nova.conf:virt_type = kvm
[root@server7 ~]#


775577c4617f
/etc/modprobe.d/kvm.conf:#options kvm_intel nested=1
/etc/nova/nova.conf:virt_type = kvm

9af0f09eb437
/etc/nova/nova.conf:virt_type = kvm

[root@server7 ~]# cat /etc/kolla/nova-libvirt/qemu.conf
stdio_handler = "file"

user = "nova"
group = "nova"
[root@server7 ~]#


```




```bash

==> /var/lib/docker/volumes/kolla_logs/_data/nova/nova-compute.log <==

: libvirtError: internal error: qemu unexpectedly closed the monitor: 2018-08-31T01:47:23.585048Z qemu-kvm: -chardev pty,id=charserial0,logfile=/var/lib/nova/instances/0de7d254-9866-4d68-a040-49bfba4dea43/console.log,logappend=off: Unable to open logfile /var/lib/nova/instances/0de7d254-9866-4d68-a040-49bfba4dea43/console.log: Permission denied

2018-08-30 21:47:24.338 7 ERROR nova.compute.manager [instance: 0de7d254-9866-4d68-a040-49bfba4dea43] libvirtError: internal error: qemu unexpectedly closed the monitor: 2018-08-31T01:47:23.585048Z qemu-kvm: -chardev pty,id=charserial0,logfile=/var/lib/nova/instances/0de7d254-9866-4d68-a040-49bfba4dea43/console.log,logappend=off: Unable to open logfile /var/lib/nova/instances/0de7d254-9866-4d68-a040-49bfba4dea43/console.log: Permission denied

==> /var/lib/docker/volumes/kolla_logs/_data/nova/nova-conductor.log <==
2018-08-30 21:47:26.519 26 ERROR nova.scheduler.utils [req-54ebf7d7-3b3f-4b42-8523-52b7ef5408fa 46425d4a0cc8452eabd63abe48b95b8f c967da5c90844797b2aff5b483dc5044 - default default] [instance: 0de7d254-9866-4d68-a040-49bfba4dea43] Error from last host: server7.pslab.net (node server7.pslab.net): [u'Traceback (most recent call last):\n', u'  File "/var/lib/kolla/venv/lib/python2.7/site-packages/nova/compute/manager.py", line 1840, in _do_build_and_run_instance\n    filter_properties, request_spec)\n', u'  File "/var/lib/kolla/venv/lib/python2.7/site-packages/nova/compute/manager.py", line 2120, in _build_and_run_instance\n    instance_uuid=instance.uuid, reason=six.text_type(e))\n', u'RescheduledException: Build of instance 0de7d254-9866-4d68-a040-49bfba4dea43 was re-scheduled: internal error: qemu unexpectedly closed the monitor: 2018-08-31T01:47:23.585048Z qemu-kvm: -chardev pty,id=charserial0,logfile=/var/lib/nova/instances/0de7d254-9866-4d68-a040-49bfba4dea43/console.log,logappend=off: Unable to open logfile /var/lib/nova/instances/0de7d254-9866-4d68-a040-49bfba4dea43/console.log: Permission denied\n']

```



# TODO: install from git had some troubles.

```bash

[root@3640f9d7073e /]# git clone -b stable/queens https://github.com/openstack/kolla
Cloning into 'kolla'...
remote: Counting objects: 61123, done.
remote: Compressing objects: 100% (39/39), done.
remote: Total 61123 (delta 14), reused 14 (delta 7), pack-reused 61076
Receiving objects: 100% (61123/61123), 10.02 MiB | 245.00 KiB/s, done.
Resolving deltas: 100% (37140/37140), done.
[root@3640f9d7073e /]#
[root@3640f9d7073e /]# git clone -b stable/queens https://github.com/openstack/kolla-ansible
Cloning into 'kolla-ansible'...
remote: Counting objects: 76161, done.
remote: Compressing objects: 100% (29/29), done.
remote: Total 76161 (delta 13), reused 1 (delta 1), pack-reused 76131
Receiving objects: 100% (76161/76161), 12.25 MiB | 2.32 MiB/s, done.
Resolving deltas: 100% (49135/49135), done.
[root@3640f9d7073e /]#


[root@3640f9d7073e /]# pip install -r kolla/requirements.txt

[root@3640f9d7073e /]# pip install -r kolla-ansible/requirements.txt




[root@3640f9d7073e /]# cp kolla-ansible/ansible/inventory/* .

ckim-mbp:kolla-ansible ckim$ docker cp server7/all-in-one centos-kolla:/all-in-one


[root@3640f9d7073e /]# mkdir -p /etc/kolla
[root@3640f9d7073e /]# cp -r kolla-ansible/etc/kolla/* /etc/kolla

ckim-mbp:kolla-ansible ckim$ docker cp server7/globals.yml centos-kolla:/etc/kolla/globals.yml



[root@3640f9d7073e /]# ansible all -i all-in-one -m ping
server7 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
[root@3640f9d7073e /]#


[root@3640f9d7073e /]# cd kolla-ansible/tools/
[root@3640f9d7073e tools]# ./generate_passwords.py
[root@3640f9d7073e tools]#



## bootstrap servers

[root@3640f9d7073e tools]# ./kolla-ansible -i ../ansible/inventory/all-in-one bootstrap-servers


had error

TASK [baremetal : Ensure localhost in /etc/hosts] *******************************************************************************************************************************************************
An exception occurred during task execution. To see the full traceback, use -vvv. The error was: OSError: [Errno 16] Device or resource busy
fatal: [localhost]: FAILED! => {"changed": false, "msg": "Unable to make /tmp/tmpm9zQTx into to /etc/hosts, failed final rename from /etc/.ansible_tmpOa1txxhosts: [Errno 16] Device or resource busy"}
	to retry, use: --limit @/kolla-ansible/ansible/kolla-host.retry
	
workaround: made it to skip	



another error

TASK [baremetal : Grant kolla user passwordless sudo] ***************************************************************************************************************************************************
fatal: [localhost]: FAILED! => {"changed": false, "msg": "Destination /etc/sudoers does not exist !", "rc": 257}
	to retry, use: --limit @/kolla-ansible/ansible/kolla-host.retry

workaround: edited the file and skip


another error

TASK [baremetal : Reload docker service file] ***********************************************************************************************************************************************************
fatal: [localhost]: FAILED! => {"changed": false, "msg": "failure 1 during daemon-reload: Failed to get D-Bus connection: Operation not permitted\n"}
	to retry, use: --limit @/kolla-ansible/ansible/kolla-host.retry
	
	
workaround: skip it


TASK [baremetal : Start docker] *************************************************************************************************************************************************************************
fatal: [localhost]: FAILED! => {"changed": false, "msg": "Could not find the requested service docker: "}
	to retry, use: --limit @/kolla-ansible/ansible/kolla-host.retry

workaround: skip it


TASK [baremetal : Enable docker] ************************************************************************************************************************************************************************
fatal: [localhost]: FAILED! => {"changed": false, "msg": "Could not find the requested service docker: "}
	to retry, use: --limit @/kolla-ansible/ansible/kolla-host.retry
	
workaround: skip it
	

TASK [baremetal : Stop time service] ********************************************************************************************************************************************************************
fatal: [localhost]: FAILED! => {"changed": false, "msg": "Could not find the requested service ntpd: "}
	to retry, use: --limit @/kolla-ansible/ansible/kolla-host.retry
	
workaround: skip it


setup ntp on server7, and disable check

[root@3640f9d7073e tools]# grep enable_host_ntp ../ansible/roles/baremetal/defaults/main.yml
enable_host_ntp: False
[root@3640f9d7073e tools]#




## prechecks

[root@3640f9d7073e tools]# ./kolla-ansible -i ../ansible/inventory/all-in-one prechecks


TASK [prechecks : Check if ansible user can do passwordless sudo] ***************************************************************************************************************************************
[DEPRECATION WARNING]: Using tests as filters is deprecated. Instead of using `result|failed` use `result is failed`. This feature will be removed in version 2.9. Deprecation warnings can be disabled
by setting deprecation_warnings=False in ansible.cfg.
 [WARNING]: Consider using 'become', 'become_method', and 'become_user' rather than running sudo

fatal: [localhost]: FAILED! => {"changed": true, "cmd": "sudo -n true", "delta": "0:00:00.234651", "end": "2018-08-29 07:05:54.632342", "failed_when_result": true, "msg": "non-zero return code", "rc": 127, "start": "2018-08-29 07:05:54.397691", "stderr": "/bin/sh: sudo: command not found", "stderr_lines": ["/bin/sh: sudo: command not found"], "stdout": "", "stdout_lines": []}
	to retry, use: --limit @/kolla-ansible/ansible/site.retry


SKIP IT


```

