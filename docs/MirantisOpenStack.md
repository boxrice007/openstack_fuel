OpenStack 是由 Rackspace 和 NASA 共同开发的云计算平台，帮助服务商和企业内部实现类似于 Amazon EC2 和 S3 的云基础架构服务(Infrastructure as a Service, IaaS)。OpenStack 包含两个主要模块：Nova 和 Swift，前者是 NASA 开发的虚拟服务器部署和业务计算模块；后者是Rackspace开发的分布式云存储模块，两者可以一起用，也可以分开单独用。OpenStack 是开源项目，除了有 Rackspace 和 NASA 的大力支持外，后面还有包括 Dell、Citrix、 Cisco、 Canonical 这些重量级公司的贡献和支持，发展速度非常快。

Openstack集群搭建使用5台机器，一台Fuel管理机，一台Controller，一台Compute，两台Storage。这是一个最小化的安装，安装完成后可以对集群进行扩容。
1. 网络环境准备

网络规划：

Floating/Public 网络 172.16.2.0/24 in VLAN 100 (untagged on servers) • Floating IP range 172.16.2.130 - 254 # 用于集群公网和虚拟机浮动IP，需要能与外网通信

Internal network (private) 192.168.100.0/24 # 用于虚拟机间通信Gateway 192.168.100.1 # 虚拟机的网关地址

DNS 8.8.4.4, 8.8.8.8 # DNS地址

Management network 192.168.0.0/24 in VLAN 501 # 管理网络

Storage network 192.168.1.0/24 in VLAN 502 # 存储网络

Administrative network (for Fuel) 10.20.0.0/24 in VLAN 503 # Fuel集群管理网络

服务器网卡配置：

Fuel管理节点（Openstack集群管理）：

eth0 10.20.0.2 — 插到交换机5 - 10口上

eth1 172.16.2.128 — 插到交换机11 - 16口上

控制节点，计算节点，存储节点：

eth0 10.20.0.0/24 — 插到交换机5 - 10口上

eth1 公有网络172.16.2.0/24，管理网络192.168.0.0/24， 存储网络192.168.1.0/24 — 插到交换机11 - 16口上

eth2 私有网络，192.168.100.0/24 — 插到交换机17 - 21口上

 2 Fuel管理节点安装
 
从Mirantis下载镜像：https://software.mirantis.com/，本次安装使用MirantisOpenStack-5.0.1.iso镜像。
通过远程管理卡进行安装，打开虚拟介质，挂载下载好的镜像，从虚拟镜像启动服务器。
到如下界面时按tab键，修改参数，hostname改为你的主机名，showmenu改为yes，回车继续：

![](/img/1.png)

到如下界面，设置网络、PXE启动、DNS&主机名、root密码等，网络配置界面，每配置一块网卡都需要Apply，然后再配置下一块网卡，都配置完成后保存退出。

![](/img/2.png)

安装完成后通过浏览访问http://10.20.0.2:8000/#clusters，点击新建Openstack环境，填写名称，并选择Openstack版本，然后点前进。

![](/img/3.png)

选择部署模式，本文使用多节点，非HA模式，然后点前进。

![](/img/4.png)

选择虚拟化管理器类型，本文选择KVM，然后点前进。

![](/img/5.png)

选择网络模式，本文选择Neutron VLAN模式，然后点前进。

![](/img/6.png)

选择存储类型，本文使用Ceph做后端存储，然后点前进。

![](/img/7.png)

附加服务，不选择，直接点前进。

![](/img/8.png)

点击完成，完成环境设置。

![](/img/9.png)

3. Openstack部署

启动各节点设置磁盘Raid：

控制节点和计算节点使用Raid 5；存储节点两块磁盘做Raid 1，剩下四块磁盘做单盘Raid 0（每块磁盘启动一个Ceph进程，以保证性能）。

计算节点需要在BIOS中打开虚拟化选项，否则在创建虚拟机时会报如下错误：

Error: No valid host was found. 

查看Nova日志中有如下报错：

libvirtError: internal error no supported architecture for os type ‘hvm'

配置完成后从网卡启动服务器，启动完成后，回到管理界面，两台服务器已经被发现。

对每台服务器的网卡进行如下配置：

controller:

![](/img/a.png)

compute:

![](/img/b.png)

点击 网络 选选卡，对网络进行如下配置：

![](/img/c.png)

![](/img/d.png)



修改完成后点 保存设置，然后点 验证网络，如果网络配置正确会显示验证成功。

点击 设置 选项卡，如下进行设置：

![](/img/12.png)

勾选Nova quotas，这样可以对虚拟资源做配额。

![](/img/13.png)

CentOS 6.5 需要设置OVS VLAN splinters特性。

![](/img/14.png)

存储勾选以上四个选项，其中 Ceph RBD选项运行虚拟机进行热迁移，完成后点 保存设置。

以上设置完成后，点击 节点 选项卡，然后点 增加节点。

![](/img/15.png)

依次选择控制节点、计算节点、和存储节点。

![](/img/16.png)

![](/img/17.png)

选择完成后配置各角色 磁盘，以存储节点为例：

![](/img/c.png)

![](/img/d.png)



配置各角色 网络，以存储节点为例：

![](/img/21.png)

![](/img/22.png)

![](/img/23-1.png)

根据上图配置网络，配置完成后返回 节点 选项卡，点 部署变更，开始部署节点：

部署操作系统：

![](/img/23.png)

部署Openstack：

![](/img/24.png)

完成部署：

![](/img/25.png)

可以点击日志选项卡，查看安装日志；点击健康检查选项卡进行健康检查：

![](/img/26.png)

访问Openstack 管理控制台http://192.168.100.10/：

![](/img/e.png)