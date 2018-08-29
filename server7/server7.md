

https://docs.openstack.org/kolla-ansible/latest/user/quickstart.html


ckim-mbp:PycharmProjects ckim$ docker run -itd --name centos-kolla centos:7

ckim-mbp:PycharmProjects ckim$ docker exec -it centos-kolla bash
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


[root@3640f9d7073e /]# ssh-keygen

[root@3640f9d7073e /]# ssh-copy-id root@server7.pslab.net

TODO: try with non-root user contrail
[root@3640f9d7073e /]# ssh-copy-id contrail@server7.pslab.net











[root@3640f9d7073e ~]# pip install kolla-ansible


ckim-mbp:kolla-ansible ckim$ docker cp server7/all-in-one centos-kolla:/all-in-one


[root@3640f9d7073e /]# mkdir -p /etc/kolla

ckim-mbp:kolla-ansible ckim$ docker cp server7/globals.yml centos-kolla:/etc/kolla/globals.yml



[root@3640f9d7073e /]# ansible all -m ping -i all-in-one
server7 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
[root@3640f9d7073e /]#


[root@3640f9d7073e /]# kolla-genpwd



[root@3640f9d7073e /]# kolla-ansible -i all-in-one bootstrap-servers



[root@3640f9d7073e /]# kolla-ansible -i all-in-one prechecks



[root@3640f9d7073e /]# kolla-ansible -i all-in-one deploy


in case
[contrail@server7 ~]$ docker pull kolla/centos-source-cron:queens








# install from git may have some hickups.


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





	

























