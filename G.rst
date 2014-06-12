==========================================================
  Ubuntu12.04.1v64_Openstack-G
==========================================================

:Version: 1.0
:Source: https://github.com/mseknibilel/OpenStack-Grizzly-Install-Guide
:Keywords: Grizzly, Quantum, Nova, Keystone, Glance, Horizon, Cinder, OpenVSwitch, KVM, Ubuntu Server 12.04.1(64 bits).

Authors
==========

`Bilel Msekni <http://www.linkedin.com/profile/view?id=136237741&trk=tab_pro>`_ 


1. �滮
====================

:���ƽڵ�: eth0 (10.xxx.xxx.165), eth1 (10.10.10.51)
:����ڵ�: eth0 (10.xxx.xxx.161), eth1 (10.10.10.52), eth2 (10.20.20.52)
:����ڵ�: eth0 (10.xxx.xxx.162), eth1 (10.10.10.53), eth2 (10.20.20.53)


.. image:: http://i.imgur.com/Frsughe.jpg

2.ϵͳ��װ
======================

��װ ubuntu 12.04.1 Server 64bit ϵͳ

��װǰ�γ�����!!

��װѡ��::
Language  --> English
--> Install Ubuntu Server
Select a language --> English
Select your location --> United States
Configure the keyboard --> No --> English(US) --> English(US)
Configure the network --> Continue --> Do not configure the network at this time -->  �����豸��(os1��3)
Set up users and passwords --> �����û�(��ctid)--> �ظ�����һ���û���--> ��������(��ctid1234)-->�ظ�����һ������ --> No 
Configure the clock --> Select from worldwide list --> Shanghai 
Partition disks --> Manual --> ����Ҫ�����ռ�
Configuring tasksel --> No automatic updates
Software selection --> OpenSSH serverInstall the GRUB boot loader on a hard disk --> Yes
Finish the installation --> Continue
��������!



3.���в��ְ�װ
=============================

3.1��������::
-----------------

sudo -s
vi /etc/network/interfaces

auto eth0
iface eth0 inet static
address 10.xxx.xxx.xxx
netmask 255.xxx.xxx.xxx
gateway 10.xxx.xxx.xxx
dns-nameservers 8.8.8.8

/etc/init.d/networking restart

3.2����apt-getԴ�б�
-----------------------
   
   apt-get update -y

*���Grizzy �ֿ�Դ

   apt-get install -y ubuntu-cloud-keyring 
   echo deb http://ubuntu-cloud.archive.canonical.com/ubuntu precise-updates/grizzly main >> /etc/apt/sources.list.d/grizzly.list
   apt-get update -y

3.3��װntp����
----------------------------

  apt-get install -y ntp

   sed -i 's/server 0.ubuntu.pool.ntp.org/#server 0.ubuntu.pool.ntp.org/g' /etc/ntp.conf
   sed -i 's/server 1.ubuntu.pool.ntp.org/#server 1.ubuntu.pool.ntp.org/g' /etc/ntp.conf
   sed -i 's/server 2.ubuntu.pool.ntp.org/#server 2.ubuntu.pool.ntp.org/g' /etc/ntp.conf
   sed -i 's/server 3.ubuntu.pool.ntp.org/#server 3.ubuntu.pool.ntp.org/g' /etc/ntp.conf

Դ��������

   sed -i 's/server ntp.ubuntu.com/server 132.xxx.xxx.18/g' /etc/ntp.conf
   service ntp restart  

������������

   sed -i 's/server ntp.ubuntu.com/server 10.10.10.51/g' /etc/ntp.conf
   service ntp restart  

3.4��װvlan �� bridge����
---------------------------------------

   apt-get install -y vlan bridge-utils

* ����IPӳ��::

   sed -i 's/#net.ipv4.ip_forward=1/net.ipv4.ip_forward=1/' /etc/sysctl.conf

   # ����ӳ��
   sysctl net.ipv4.ip_forward=1


3.5��װ3.2.54�ں�
-----------------------------------------

wget http://kernel.ubuntu.com/~kernel-ppa/mainline/v3.2.54-precise/linux-image-3.2.54-030254-generic_3.2.54-030254.201401030035_amd64.deb 
wget http://kernel.ubuntu.com/~kernel-ppa/mainline/v3.2.54-precise/linux-headers-3.2.54-030254_3.2.54-030254.201401030035_all.deb
wget http://kernel.ubuntu.com/~kernel-ppa/mainline/v3.2.54-precise/linux-headers-3.2.54-030254-generic_3.2.54-030254.201401030035_amd64.deb

dpkg -i linux-image-3.2.54-030254-generic_3.2.54-030254.201401030035_amd64.deb
dpkg -i linux-headers-3.2.54-030254_3.2.54-030254.201401030035_all.deb
dpkg -i linux-headers-3.2.54-030254-generic_3.2.54-030254.201401030035_amd64.deb

reboot


4. ���ƽڵ㰲װ
===============


4.1. ��������
------------

*���eth1��������

   auto eth1
   iface eth1 inet static
   address 10.10.10.51
   netmask 255.255.255.0

* �����������::

   /etc/init.d/networking restart

4.2. MySQL & RabbitMQ
------------

* ��װ MySQL::

   apt-get install -y mysql-server python-mysqldb

#��CRT���ó�UTF-8�ַ����Ա�����������

* ����mysql::

   sed -i 's/127.0.0.1/0.0.0.0/g' /etc/mysql/my.cnf
   service mysql restart

4.3. RabbitMQ
-------------------

* ��װ RabbitMQ::

   apt-get install -y rabbitmq-server 

* �������ݿ�::

   mysql -u root -p
   
   #Keystone
   CREATE DATABASE keystone;
   GRANT ALL ON keystone.* TO 'keystoneUser'@'%' IDENTIFIED BY 'keystonePass';
   
   #Glance
   CREATE DATABASE glance;
   GRANT ALL ON glance.* TO 'glanceUser'@'%' IDENTIFIED BY 'glancePass';

   #Quantum
   CREATE DATABASE quantum;
   GRANT ALL ON quantum.* TO 'quantumUser'@'%' IDENTIFIED BY 'quantumPass';

   #Nova
   CREATE DATABASE nova;
   GRANT ALL ON nova.* TO 'novaUser'@'%' IDENTIFIED BY 'novaPass';      

   #Cinder
   CREATE DATABASE cinder;
   GRANT ALL ON cinder.* TO 'cinderUser'@'%' IDENTIFIED BY 'cinderPass';

   quit;


4.4. Keystone
-------------------

* ��װ keystone ::

   apt-get install -y keystone

* ����/etc/keystone/keystone.conf::

   connection = mysql://keystoneUser:keystonePass@10.10.10.51/keystone

* ����keystone����::

   service keystone restart
   keystone-manage db_sync

*�������ݿⴴ���ű�::

   #���غ��޸Ľű��еĶ�ӦIP��ַ
   
   wget https://raw.github.com/mseknibilel/OpenStack-Grizzly-Install-Guide/OVS_MultiNode/KeystoneScripts/keystone_basic.sh
   wget https://raw.github.com/mseknibilel/OpenStack-Grizzly-Install-Guide/OVS_MultiNode/KeystoneScripts/keystone_endpoints_basic.sh

   chmod +x keystone_basic.sh
   chmod +x keystone_endpoints_basic.sh

   ./keystone_basic.sh
   ./keystone_endpoints_basic.sh

* �������������ļ�::

   nano creds

   #Paste the following:
   export OS_TENANT_NAME=admin
   export OS_USERNAME=admin
   export OS_PASSWORD=admin_pass
   export OS_AUTH_URL="http://10.xxx.xxx.165:5000/v2.0/"

   # Ӧ�ñ���:
   source creds

* �鿴�û��б������������������::

   keystone user-list

4.5. Glance
-------------------

* ��װGlance::

   apt-get install -y glance

* ���� /etc/glance/glance-api-paste.ini ::

   [filter:authtoken]
   paste.filter_factory = keystoneclient.middleware.auth_token:filter_factory
   delay_auth_decision = true
   auth_host = 10.10.10.51
   auth_port = 35357
   auth_protocol = http
   admin_tenant_name = service
   admin_user = glance
   admin_password = service_pass

* ���� /etc/glance/glance-registry-paste.ini ::

   [filter:authtoken]
   paste.filter_factory = keystoneclient.middleware.auth_token:filter_factory
   auth_host = 10.10.10.51
   auth_port = 35357
   auth_protocol = http
   admin_tenant_name = service
   admin_user = glance
   admin_password = service_pass

* ���� /etc/glance/glance-api.conf ::

   sql_connection = mysql://glanceUser:glancePass@10.10.10.51/glance

* ��::

   [paste_deploy]
   flavor = keystone
   
* ���� /etc/glance/glance-registry.conf ::

   sql_connection = mysql://glanceUser:glancePass@10.10.10.51/glance

* ��::

   [paste_deploy]
   flavor = keystone

* ������Ӧ����::

   service glance-api restart; service glance-registry restart

* ͬ��Glance���ݿ�::

   glance-manage db_sync

* �������������ļ�::

   glance image-create --name myFirstImage --is-public true --container-format bare --disk-format qcow2 --location http://download.cirros-cloud.net/0.3.1/cirros-0.3.1-x86_64-disk.img

* �鿴����״̬::

   glance image-list

4.6. Quantum
-------------------

* ��װQuantum::

   apt-get install -y quantum-server

* ���� /etc/quantum/plugins/openvswitch/ovs_quantum_plugin.ini :: 

   #Under the database section
   [DATABASE]
   sql_connection = mysql://quantumUser:quantumPass@10.10.10.51/quantum

   #Under the OVS section
   [OVS]
   tenant_network_type = gre
   tunnel_id_ranges = 1:1000
   enable_tunneling = True

   #Firewall driver for realizing quantum security group function
   [SECURITYGROUP]
   firewall_driver = quantum.agent.linux.iptables_firewall.OVSHybridIptablesFirewallDriver

* ����/etc/quantum/api-paste.ini ::

   [filter:authtoken]
   paste.filter_factory = keystoneclient.middleware.auth_token:filter_factory
   auth_host = 10.10.10.51
   auth_port = 35357
   auth_protocol = http
   admin_tenant_name = service
   admin_user = quantum
   admin_password = service_pass

* ���� /etc/quantum/quantum.conf::

   [keystone_authtoken]
   auth_host = 10.10.10.51
   auth_port = 35357
   auth_protocol = http
   admin_tenant_name = service
   admin_user = quantum
   admin_password = service_pass
   signing_dir = /var/lib/quantum/keystone-signing

* ����quantum-server����::

   service quantum-server restart

4.7. Nova
------------------

* ��װNova::

   apt-get install -y nova-api nova-cert novnc nova-consoleauth nova-scheduler nova-novncproxy nova-doc nova-conductor

* ���� /etc/nova/api-paste.ini ::

   [filter:authtoken]
   paste.filter_factory = keystoneclient.middleware.auth_token:filter_factory
   auth_host = 10.10.10.51
   auth_port = 35357
   auth_protocol = http
   admin_tenant_name = service
   admin_user = nova
   admin_password = service_pass
   signing_dirname = /tmp/keystone-signing-nova
   # Workaround for https://bugs.launchpad.net/nova/+bug/1154809
   auth_version = v2.0

* ���� /etc/nova/nova.conf ::

   [DEFAULT] 
   logdir=/var/log/nova
   state_path=/var/lib/nova
   lock_path=/run/lock/nova
   verbose=True
   api_paste_config=/etc/nova/api-paste.ini
   compute_scheduler_driver=nova.scheduler.simple.SimpleScheduler
   rabbit_host=10.10.10.51
   nova_url=http://10.10.10.51:8774/v1.1/
   sql_connection=mysql://novaUser:novaPass@10.10.10.51/nova
   root_helper=sudo nova-rootwrap /etc/nova/rootwrap.conf

   # Auth
   use_deprecated_auth=false
   auth_strategy=keystone

   # Imaging service
   glance_api_servers=10.10.10.51:9292
   image_service=nova.image.glance.GlanceImageService

   # Vnc configuration
   novnc_enabled=true
   novncproxy_base_url=http://10.xxx.xxx.165:6080/vnc_auto.html
   novncproxy_port=6080
   vncserver_proxyclient_address=10.10.10.51
   vncserver_listen=0.0.0.0

   # Network settings
   network_api_class=nova.network.quantumv2.api.API
   quantum_url=http://10.10.10.51:9696
   quantum_auth_strategy=keystone
   quantum_admin_tenant_name=service
   quantum_admin_username=quantum
   quantum_admin_password=service_pass
   quantum_admin_auth_url=http://10.10.10.51:35357/v2.0
   libvirt_vif_driver=nova.virt.libvirt.vif.LibvirtHybridOVSBridgeDriver
   linuxnet_interface_driver=nova.network.linux_net.LinuxOVSInterfaceDriver
   #If you want Quantum + Nova Security groups
   firewall_driver=nova.virt.firewall.NoopFirewallDriver
   security_group_api=quantum
   #If you want Nova Security groups only, comment the two lines above and uncomment line -1-.
   #-1-firewall_driver=nova.virt.libvirt.firewall.IptablesFirewallDriver

   #Metadata
   service_quantum_metadata_proxy = True
   quantum_metadata_proxy_shared_secret = helloOpenStack

   # Compute #
   compute_driver=libvirt.LibvirtDriver

   # Cinder #
   volume_api_class=nova.volume.cinder.API
   osapi_volume_listen_port=5900

* ͬ��nova���ݿ�::

   nova-manage db sync

*�������� nova-* ����::

   cd /etc/init.d/; for i in $( ls nova-* ); do sudo service $i restart; done   

* ȷ�Ϸ���״̬::

   nova-manage service list

4.8. Cinder
--------------

* ��װCinder::

   apt-get install -y cinder-api cinder-scheduler cinder-volume iscsitarget open-iscsi iscsitarget-dkms

* ����iscsi����::

   sed -i 's/false/true/g' /etc/default/iscsitarget

* ������ط���::
   
   service iscsitarget start
   service open-iscsi start

* ���� /etc/cinder/api-paste.ini ::

   [filter:authtoken]
   paste.filter_factory = keystoneclient.middleware.auth_token:filter_factory
   service_protocol = http
   service_host = 10.xxx.xxx.165
   service_port = 5000
   auth_host = 10.10.10.51
   auth_port = 35357
   auth_protocol = http
   admin_tenant_name = service
   admin_user = cinder
   admin_password = service_pass
   signing_dir = /var/lib/cinder

* ���� /etc/cinder/cinder.conf ::

   [DEFAULT]
   rootwrap_config=/etc/cinder/rootwrap.conf
   sql_connection = mysql://cinderUser:cinderPass@10.10.10.51/cinder
   api_paste_config = /etc/cinder/api-paste.ini
   iscsi_helper=ietadm
   volume_name_template = volume-%s
   volume_group = cinder-volumes
   verbose = True
   auth_strategy = keystone
   iscsi_ip_address=10.10.10.51

* ͬ��Cinder���ݿ�::

   cinder-manage db sync

* ����cinder-volumes::

   dd if=/dev/zero of=cinder-volumes bs=1 count=0 seek=2G
   losetup /dev/loop2 cinder-volumes
   fdisk /dev/loop2
   #Type in the followings:
   n
   p
   1
   ENTER
   ENTER
   t
   8e
   w

* ����pv��vg::

   pvcreate /dev/loop2
   vgcreate cinder-volumes /dev/loop2

#ɾ��/etc/init.d/ ����cinder-volumes�ļ�

* ���� cinder ����::

   cd /etc/init.d/; for i in $( ls cinder-* ); do sudo service $i restart; done

* �鿴 cinder ����״̬::

   cd /etc/init.d/; for i in $( ls cinder-* ); do sudo service $i status; done

4.9. Horizon
--------------

* ��װhorizon ::

   apt-get install -y openstack-dashboard memcached

*����㲻ϲ��ubuntu-theme��񣬿���һ������ر�::

   dpkg --purge openstack-dashboard-ubuntu-theme 

* ���� Apache �� memcached::

   service apache2 restart; service memcached restart

* �������� http://10.xxx.xxx.165/horizon. ����Ա��ʼ�û���/���� admin / admin_pass


5. ����ڵ�
================

5.1.��������
------------

vi /etc/network/interfaces

*���eth1��eth2

   auto eth1
   iface eth1 inet static
   address 10.10.10.52
   netmask 255.255.255.0

   auto eth2
   iface eth2 inet static
   address 10.20.20.52
   netmask 255.255.255.0


5.2. OpenVSwitch (��һ����)
------------------

* ��װopenVSwitch::

   apt-get install -y openvswitch-switch openvswitch-datapath-dkms

* ���� bridges::

   ovs-vsctl add-br br-int
   ovs-vsctl add-br br-ex

5.3. Quantum
------------------

* ��װQuantum openvswitch agent, l3 agent �� dhcp agent::

   apt-get -y install quantum-plugin-openvswitch-agent quantum-dhcp-agent quantum-l3-agent quantum-metadata-agent

* ���� /etc/quantum/api-paste.ini::

   [filter:authtoken]
   paste.filter_factory = keystoneclient.middleware.auth_token:filter_factory
   auth_host = 10.10.10.51
   auth_port = 35357
   auth_protocol = http
   admin_tenant_name = service
   admin_user = quantum
   admin_password = service_pass

* ���� /etc/quantum/plugins/openvswitch/ovs_quantum_plugin.ini :: 

   #Under the database section
   [DATABASE]
   sql_connection = mysql://quantumUser:quantumPass@10.10.10.51/quantum

   #Under the OVS section
   [OVS]
   tenant_network_type = gre
   tunnel_id_ranges = 1:1000
   integration_bridge = br-int
   tunnel_bridge = br-tun
   local_ip = 10.20.20.52
   enable_tunneling = True

   #Firewall driver for realizing quantum security group function
   [SECURITYGROUP]
   firewall_driver = quantum.agent.linux.iptables_firewall.OVSHybridIptablesFirewallDriver

* ���� /etc/quantum/metadata_agent.ini::
   
   # The Quantum user information for accessing the Quantum API.
   auth_url = http://10.10.10.51:35357/v2.0
   auth_region = RegionOne
   admin_tenant_name = service
   admin_user = quantum
   admin_password = service_pass

   # IP address used by Nova metadata server
   nova_metadata_ip = 10.10.10.51

   # TCP Port used by Nova metadata server
   nova_metadata_port = 8775

   metadata_proxy_shared_secret = helloOpenStack

* ���� /etc/quantum/quantum.conf ::

   rabbit_host = 10.10.10.51

   #And update the keystone_authtoken section

   [keystone_authtoken]
   auth_host = 10.10.10.51
   auth_port = 35357
   auth_protocol = http
   admin_tenant_name = service
   admin_user = quantum
   admin_password = service_pass
   signing_dir = /var/lib/quantum/keystone-signing

* ���� /etc/sudoers.d/quantum_sudoers  ::

   nano /etc/sudoers.d/quantum_sudoers
   
   #Modify the quantum user
   quantum ALL=NOPASSWD: ALL

* ����quantum����::

   cd /etc/init.d/; for i in $( ls quantum-* ); do sudo service $i restart; done

5.4. OpenVSwitch (�ڶ�����)
------------------
* ���� /etc/network/interfaces ::

   auto eth0
   iface eth0 inet manual
   up ifconfig $IFACE 0.0.0.0 up
   up ip link set $IFACE promisc on
   down ip link set $IFACE promisc off
   down ifconfig $IFACE down

ԭ��eth0 ���ñ��Ϊ br-ex

* ���eth0 �� br-ex����������::

   ovs-vsctl add-port br-ex eth0;/etc/init.d/networking restart

  6. ����ڵ�
=========================

6.1. ��������
------------

*���eth1 �� eth2 ����

   auto eth1
   iface eth1 inet static
   address 10.10.10.53
   netmask 255.255.255.0

   auto eth2
   iface eth2 inet static
   address 10.20.20.53
   netmask 255.255.255.0

6.2 KVM
------------------

* ��װcpu-checker::

   apt-get install -y cpu-checker
   kvm-ok

* ��װKVM::

   apt-get install -y kvm libvirt-bin pm-utils

* ����  /etc/libvirt/qemu.conf ::

   cgroup_device_acl = [
   "/dev/null", "/dev/full", "/dev/zero",
   "/dev/random", "/dev/urandom",
   "/dev/ptmx", "/dev/kvm", "/dev/kqemu",
   "/dev/rtc", "/dev/hpet","/dev/net/tun"
   ]

* �ر�KVM default virtual bridge ::

   virsh net-destroy default
   virsh net-undefine default

* ���� /etc/libvirt/libvirtd.conf::

   listen_tls = 0
   listen_tcp = 1
   auth_tcp = "none"

* ���� /etc/init/libvirt-bin.conf ::

   env libvirtd_opts="-d -l"

* ���� /etc/default/libvirt-bin  ::

   libvirtd_opts="-d -l"

* ����dbus �� libvirt-bin����::

    service dbus restart && service libvirt-bin restart

6.3. OpenVSwitch
------------------

* ��װ openVSwitch::

   apt-get install -y openvswitch-switch openvswitch-datapath-dkms

* ����bridges::

   ovs-vsctl add-br br-int

6.4. Quantum
------------------

* ��װ Quantum openvswitch agent::

   apt-get -y install quantum-plugin-openvswitch-agent

* ���� /etc/quantum/plugins/openvswitch/ovs_quantum_plugin.ini :: 

   #Under the database section
   [DATABASE]
   sql_connection = mysql://quantumUser:quantumPass@10.10.10.51/quantum

   #Under the OVS section
   [OVS]
   tenant_network_type = gre
   tunnel_id_ranges = 1:1000
   integration_bridge = br-int
   tunnel_bridge = br-tun
   local_ip = 10.20.20.53
   enable_tunneling = True
   
   #Firewall driver for realizing quantum security group function
   [SECURITYGROUP]
   firewall_driver = quantum.agent.linux.iptables_firewall.OVSHybridIptablesFirewallDriver

* ����/etc/quantum/quantum.conf::
   
   rabbit_host = 10.10.10.51

   #And update the keystone_authtoken section

   [keystone_authtoken]
   auth_host = 10.10.10.51
   auth_port = 35357
   auth_protocol = http
   admin_tenant_name = service
   admin_user = quantum
   admin_password = service_pass
   signing_dir = /var/lib/quantum/keystone-signing

* ��������::

   service quantum-plugin-openvswitch-agent restart

6.5. Nova
------------------

* ��װNova::

   apt-get install -y nova-compute-kvm

* ���� /etc/nova/api-paste.ini ::

   [filter:authtoken]
   paste.filter_factory = keystoneclient.middleware.auth_token:filter_factory
   auth_host = 10.10.10.51
   auth_port = 35357
   auth_protocol = http
   admin_tenant_name = service
   admin_user = nova
   admin_password = service_pass
   signing_dirname = /tmp/keystone-signing-nova
   # Workaround for https://bugs.launchpad.net/nova/+bug/1154809
   auth_version = v2.0

* ����/etc/nova/nova-compute.conf  ::
   
   [DEFAULT]
   libvirt_type=kvm
   libvirt_ovs_bridge=br-int
   libvirt_vif_type=ethernet
   libvirt_vif_driver=nova.virt.libvirt.vif.LibvirtHybridOVSBridgeDriver
   libvirt_use_virtio_for_bridges=True

* ���� /etc/nova/nova.conf ::

   [DEFAULT] 
   logdir=/var/log/nova
   state_path=/var/lib/nova
   lock_path=/run/lock/nova
   verbose=True
   api_paste_config=/etc/nova/api-paste.ini
   compute_scheduler_driver=nova.scheduler.simple.SimpleScheduler
   rabbit_host=10.10.10.51
   nova_url=http://10.10.10.51:8774/v1.1/
   sql_connection=mysql://novaUser:novaPass@10.10.10.51/nova
   root_helper=sudo nova-rootwrap /etc/nova/rootwrap.conf

   # Auth
   use_deprecated_auth=false
   auth_strategy=keystone

   # Imaging service
   glance_api_servers=10.10.10.51:9292
   image_service=nova.image.glance.GlanceImageService

   # Vnc configuration
   novnc_enabled=true
   novncproxy_base_url=http://192.168.100.51:6080/vnc_auto.html
   novncproxy_port=6080
   vncserver_proxyclient_address=10.10.10.53
   vncserver_listen=0.0.0.0

   # Network settings
   network_api_class=nova.network.quantumv2.api.API
   quantum_url=http://10.10.10.51:9696
   quantum_auth_strategy=keystone
   quantum_admin_tenant_name=service
   quantum_admin_username=quantum
   quantum_admin_password=service_pass
   quantum_admin_auth_url=http://10.10.10.51:35357/v2.0
   libvirt_vif_driver=nova.virt.libvirt.vif.LibvirtHybridOVSBridgeDriver
   linuxnet_interface_driver=nova.network.linux_net.LinuxOVSInterfaceDriver
   #If you want Quantum + Nova Security groups
   firewall_driver=nova.virt.firewall.NoopFirewallDriver
   security_group_api=quantum
   #If you want Nova Security groups only, comment the two lines above and uncomment line -1-.
   #-1-firewall_driver=nova.virt.libvirt.firewall.IptablesFirewallDriver
   
   #Metadata
   service_quantum_metadata_proxy = True
   quantum_metadata_proxy_shared_secret = helloOpenStack

   # Compute #
   compute_driver=libvirt.LibvirtDriver

   # Cinder #
   volume_api_class=nova.volume.cinder.API
   osapi_volume_listen_port=5900
   cinder_catalog_info=volume:cinder:internalURL

* ���� nova-* ����::

   cd /etc/init.d/; for i in $( ls nova-* ); do sudo service $i restart; done   

* ȷ��nova����״̬::

   nova-manage service list


