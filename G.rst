==========================================================
  Ubuntu12.04.1v64_Openstack-G
==========================================================

:Version: 1.0
:Source: https://github.com/mseknibilel/OpenStack-Grizzly-Install-Guide
:Keywords: Grizzly, Quantum, Nova, Keystone, Glance, Horizon, Cinder, OpenVSwitch, KVM, Ubuntu Server 12.04.1(64 bits).

Authors
==========

`Bilel Msekni <http://www.linkedin.com/profile/view?id=136237741&trk=tab_pro>`_ 


1. 规划
====================

:控制节点: eth0 (10.xxx.xxx.165), eth1 (10.10.10.51)
:网络节点: eth0 (10.xxx.xxx.161), eth1 (10.10.10.52), eth2 (10.20.20.52)
:计算节点: eth0 (10.xxx.xxx.162), eth1 (10.10.10.53), eth2 (10.20.20.53)


.. image:: http://i.imgur.com/Frsughe.jpg

2.系统安装
======================

安装 ubuntu 12.04.1 Server 64bit 系统

安装前拔出网线!!

安装选择::
Language  --> English
--> Install Ubuntu Server
Select a language --> English
Select your location --> United States
Configure the keyboard --> No --> English(US) --> English(US)
Configure the network --> Continue --> Do not configure the network at this time -->  输入设备名(os1…3)
Set up users and passwords --> 输入用户(如ctid)--> 重复输入一次用户名--> 输入密码(如ctid1234)-->重复输入一次密码 --> No 
Configure the clock --> Select from worldwide list --> Shanghai 
Partition disks --> Manual --> 根据要求分配空间
Configuring tasksel --> No automatic updates
Software selection --> OpenSSH serverInstall the GRUB boot loader on a hard disk --> Yes
Finish the installation --> Continue
插上网线!



3.共有部分安装
=============================

3.1配置网络::
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

3.2更新apt-get源列表
-----------------------
   
   apt-get update -y

*添加Grizzy 仓库源

   apt-get install -y ubuntu-cloud-keyring 
   echo deb http://ubuntu-cloud.archive.canonical.com/ubuntu precise-updates/grizzly main >> /etc/apt/sources.list.d/grizzly.list
   apt-get update -y

3.3安装ntp服务
----------------------------

  apt-get install -y ntp

   sed -i 's/server 0.ubuntu.pool.ntp.org/#server 0.ubuntu.pool.ntp.org/g' /etc/ntp.conf
   sed -i 's/server 1.ubuntu.pool.ntp.org/#server 1.ubuntu.pool.ntp.org/g' /etc/ntp.conf
   sed -i 's/server 2.ubuntu.pool.ntp.org/#server 2.ubuntu.pool.ntp.org/g' /etc/ntp.conf
   sed -i 's/server 3.ubuntu.pool.ntp.org/#server 3.ubuntu.pool.ntp.org/g' /etc/ntp.conf

源服务器：

   sed -i 's/server ntp.ubuntu.com/server 132.xxx.xxx.18/g' /etc/ntp.conf
   service ntp restart  

其他服务器：

   sed -i 's/server ntp.ubuntu.com/server 10.10.10.51/g' /etc/ntp.conf
   service ntp restart  

3.4安装vlan 和 bridge服务
---------------------------------------

   apt-get install -y vlan bridge-utils

* 配置IP映射::

   sed -i 's/#net.ipv4.ip_forward=1/net.ipv4.ip_forward=1/' /etc/sysctl.conf

   # 激活映射
   sysctl net.ipv4.ip_forward=1


3.5安装3.2.54内核
-----------------------------------------

wget http://kernel.ubuntu.com/~kernel-ppa/mainline/v3.2.54-precise/linux-image-3.2.54-030254-generic_3.2.54-030254.201401030035_amd64.deb 
wget http://kernel.ubuntu.com/~kernel-ppa/mainline/v3.2.54-precise/linux-headers-3.2.54-030254_3.2.54-030254.201401030035_all.deb
wget http://kernel.ubuntu.com/~kernel-ppa/mainline/v3.2.54-precise/linux-headers-3.2.54-030254-generic_3.2.54-030254.201401030035_amd64.deb

dpkg -i linux-image-3.2.54-030254-generic_3.2.54-030254.201401030035_amd64.deb
dpkg -i linux-headers-3.2.54-030254_3.2.54-030254.201401030035_all.deb
dpkg -i linux-headers-3.2.54-030254-generic_3.2.54-030254.201401030035_amd64.deb

reboot


4. 控制节点安装
===============


4.1. 网络配置
------------

*添加eth1网卡配置

   auto eth1
   iface eth1 inet static
   address 10.10.10.51
   netmask 255.255.255.0

* 重启网络服务::

   /etc/init.d/networking restart

4.2. MySQL & RabbitMQ
------------

* 安装 MySQL::

   apt-get install -y mysql-server python-mysqldb

#将CRT设置成UTF-8字符，以便于输入密码

* 配置mysql::

   sed -i 's/127.0.0.1/0.0.0.0/g' /etc/mysql/my.cnf
   service mysql restart

4.3. RabbitMQ
-------------------

* 安装 RabbitMQ::

   apt-get install -y rabbitmq-server 

* 创建数据库::

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

* 安装 keystone ::

   apt-get install -y keystone

* 更新/etc/keystone/keystone.conf::

   connection = mysql://keystoneUser:keystonePass@10.10.10.51/keystone

* 重启keystone服务::

   service keystone restart
   keystone-manage db_sync

*下载数据库创建脚本::

   #下载后修改脚本中的对应IP地址
   
   wget https://raw.github.com/mseknibilel/OpenStack-Grizzly-Install-Guide/OVS_MultiNode/KeystoneScripts/keystone_basic.sh
   wget https://raw.github.com/mseknibilel/OpenStack-Grizzly-Install-Guide/OVS_MultiNode/KeystoneScripts/keystone_endpoints_basic.sh

   chmod +x keystone_basic.sh
   chmod +x keystone_endpoints_basic.sh

   ./keystone_basic.sh
   ./keystone_endpoints_basic.sh

* 创建环境变量文件::

   nano creds

   #Paste the following:
   export OS_TENANT_NAME=admin
   export OS_USERNAME=admin
   export OS_PASSWORD=admin_pass
   export OS_AUTH_URL="http://10.xxx.xxx.165:5000/v2.0/"

   # 应用变量:
   source creds

* 查看用户列表，如正常继续后面操作::

   keystone user-list

4.5. Glance
-------------------

* 安装Glance::

   apt-get install -y glance

* 更新 /etc/glance/glance-api-paste.ini ::

   [filter:authtoken]
   paste.filter_factory = keystoneclient.middleware.auth_token:filter_factory
   delay_auth_decision = true
   auth_host = 10.10.10.51
   auth_port = 35357
   auth_protocol = http
   admin_tenant_name = service
   admin_user = glance
   admin_password = service_pass

* 更新 /etc/glance/glance-registry-paste.ini ::

   [filter:authtoken]
   paste.filter_factory = keystoneclient.middleware.auth_token:filter_factory
   auth_host = 10.10.10.51
   auth_port = 35357
   auth_protocol = http
   admin_tenant_name = service
   admin_user = glance
   admin_password = service_pass

* 更新 /etc/glance/glance-api.conf ::

   sql_connection = mysql://glanceUser:glancePass@10.10.10.51/glance

* 和::

   [paste_deploy]
   flavor = keystone
   
* 更新 /etc/glance/glance-registry.conf ::

   sql_connection = mysql://glanceUser:glancePass@10.10.10.51/glance

* 和::

   [paste_deploy]
   flavor = keystone

* 重启相应服务::

   service glance-api restart; service glance-registry restart

* 同步Glance数据库::

   glance-manage db_sync

* 创建基础镜像文件::

   glance image-create --name myFirstImage --is-public true --container-format bare --disk-format qcow2 --location http://download.cirros-cloud.net/0.3.1/cirros-0.3.1-x86_64-disk.img

* 查看创建状态::

   glance image-list

4.6. Quantum
-------------------

* 安装Quantum::

   apt-get install -y quantum-server

* 更新 /etc/quantum/plugins/openvswitch/ovs_quantum_plugin.ini :: 

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

* 更新/etc/quantum/api-paste.ini ::

   [filter:authtoken]
   paste.filter_factory = keystoneclient.middleware.auth_token:filter_factory
   auth_host = 10.10.10.51
   auth_port = 35357
   auth_protocol = http
   admin_tenant_name = service
   admin_user = quantum
   admin_password = service_pass

* 更新 /etc/quantum/quantum.conf::

   [keystone_authtoken]
   auth_host = 10.10.10.51
   auth_port = 35357
   auth_protocol = http
   admin_tenant_name = service
   admin_user = quantum
   admin_password = service_pass
   signing_dir = /var/lib/quantum/keystone-signing

* 重启quantum-server服务::

   service quantum-server restart

4.7. Nova
------------------

* 安装Nova::

   apt-get install -y nova-api nova-cert novnc nova-consoleauth nova-scheduler nova-novncproxy nova-doc nova-conductor

* 更新 /etc/nova/api-paste.ini ::

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

* 更新 /etc/nova/nova.conf ::

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

* 同步nova数据库::

   nova-manage db sync

*重启所有 nova-* 服务::

   cd /etc/init.d/; for i in $( ls nova-* ); do sudo service $i restart; done   

* 确认服务状态::

   nova-manage service list

4.8. Cinder
--------------

* 安装Cinder::

   apt-get install -y cinder-api cinder-scheduler cinder-volume iscsitarget open-iscsi iscsitarget-dkms

* 配置iscsi服务::

   sed -i 's/false/true/g' /etc/default/iscsitarget

* 启动相关服务::
   
   service iscsitarget start
   service open-iscsi start

* 更新 /etc/cinder/api-paste.ini ::

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

* 更新 /etc/cinder/cinder.conf ::

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

* 同步Cinder数据库::

   cinder-manage db sync

* 创建cinder-volumes::

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

* 创建pv及vg::

   pvcreate /dev/loop2
   vgcreate cinder-volumes /dev/loop2

#删除/etc/init.d/ 产生cinder-volumes文件

* 重启 cinder 服务::

   cd /etc/init.d/; for i in $( ls cinder-* ); do sudo service $i restart; done

* 查看 cinder 服务状态::

   cd /etc/init.d/; for i in $( ls cinder-* ); do sudo service $i status; done

4.9. Horizon
--------------

* 安装horizon ::

   apt-get install -y openstack-dashboard memcached

*如果你不喜欢ubuntu-theme风格，可以一下命令关闭::

   dpkg --purge openstack-dashboard-ubuntu-theme 

* 重启 Apache 和 memcached::

   service apache2 restart; service memcached restart

* 尝试连接 http://10.xxx.xxx.165/horizon. 管理员初始用户名/密码 admin / admin_pass


5. 网络节点
================

5.1.网络配置
------------

vi /etc/network/interfaces

*添加eth1和eth2

   auto eth1
   iface eth1 inet static
   address 10.10.10.52
   netmask 255.255.255.0

   auto eth2
   iface eth2 inet static
   address 10.20.20.52
   netmask 255.255.255.0


5.2. OpenVSwitch (第一部分)
------------------

* 安装openVSwitch::

   apt-get install -y openvswitch-switch openvswitch-datapath-dkms

* 创建 bridges::

   ovs-vsctl add-br br-int
   ovs-vsctl add-br br-ex

5.3. Quantum
------------------

* 安装Quantum openvswitch agent, l3 agent 和 dhcp agent::

   apt-get -y install quantum-plugin-openvswitch-agent quantum-dhcp-agent quantum-l3-agent quantum-metadata-agent

* 更新 /etc/quantum/api-paste.ini::

   [filter:authtoken]
   paste.filter_factory = keystoneclient.middleware.auth_token:filter_factory
   auth_host = 10.10.10.51
   auth_port = 35357
   auth_protocol = http
   admin_tenant_name = service
   admin_user = quantum
   admin_password = service_pass

* 更新 /etc/quantum/plugins/openvswitch/ovs_quantum_plugin.ini :: 

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

* 更新 /etc/quantum/metadata_agent.ini::
   
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

* 更新 /etc/quantum/quantum.conf ::

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

* 更新 /etc/sudoers.d/quantum_sudoers  ::

   nano /etc/sudoers.d/quantum_sudoers
   
   #Modify the quantum user
   quantum ALL=NOPASSWD: ALL

* 重启quantum服务::

   cd /etc/init.d/; for i in $( ls quantum-* ); do sudo service $i restart; done

5.4. OpenVSwitch (第二部分)
------------------
* 更新 /etc/network/interfaces ::

   auto eth0
   iface eth0 inet manual
   up ifconfig $IFACE 0.0.0.0 up
   up ip link set $IFACE promisc on
   down ip link set $IFACE promisc off
   down ifconfig $IFACE down

原有eth0 配置变更为 br-ex

* 添加eth0 到 br-ex并重启网络::

   ovs-vsctl add-port br-ex eth0;/etc/init.d/networking restart

  6. 计算节点
=========================

6.1. 网络配置
------------

*添加eth1 和 eth2 网卡

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

* 安装cpu-checker::

   apt-get install -y cpu-checker
   kvm-ok

* 安装KVM::

   apt-get install -y kvm libvirt-bin pm-utils

* 更新  /etc/libvirt/qemu.conf ::

   cgroup_device_acl = [
   "/dev/null", "/dev/full", "/dev/zero",
   "/dev/random", "/dev/urandom",
   "/dev/ptmx", "/dev/kvm", "/dev/kqemu",
   "/dev/rtc", "/dev/hpet","/dev/net/tun"
   ]

* 关闭KVM default virtual bridge ::

   virsh net-destroy default
   virsh net-undefine default

* 更新 /etc/libvirt/libvirtd.conf::

   listen_tls = 0
   listen_tcp = 1
   auth_tcp = "none"

* 更新 /etc/init/libvirt-bin.conf ::

   env libvirtd_opts="-d -l"

* 更新 /etc/default/libvirt-bin  ::

   libvirtd_opts="-d -l"

* 重启dbus 和 libvirt-bin服务::

    service dbus restart && service libvirt-bin restart

6.3. OpenVSwitch
------------------

* 安装 openVSwitch::

   apt-get install -y openvswitch-switch openvswitch-datapath-dkms

* 创建bridges::

   ovs-vsctl add-br br-int

6.4. Quantum
------------------

* 安装 Quantum openvswitch agent::

   apt-get -y install quantum-plugin-openvswitch-agent

* 更新 /etc/quantum/plugins/openvswitch/ovs_quantum_plugin.ini :: 

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

* 更新/etc/quantum/quantum.conf::
   
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

* 重启服务::

   service quantum-plugin-openvswitch-agent restart

6.5. Nova
------------------

* 安装Nova::

   apt-get install -y nova-compute-kvm

* 更新 /etc/nova/api-paste.ini ::

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

* 更新/etc/nova/nova-compute.conf  ::
   
   [DEFAULT]
   libvirt_type=kvm
   libvirt_ovs_bridge=br-int
   libvirt_vif_type=ethernet
   libvirt_vif_driver=nova.virt.libvirt.vif.LibvirtHybridOVSBridgeDriver
   libvirt_use_virtio_for_bridges=True

* 更新 /etc/nova/nova.conf ::

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

* 重启 nova-* 服务::

   cd /etc/init.d/; for i in $( ls nova-* ); do sudo service $i restart; done   

* 确认nova服务状态::

   nova-manage service list


