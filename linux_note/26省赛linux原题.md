
**（一）检查实例环境**

1.网络管理信息表：

|   **网络名称**    | **协议版本** |    **子网网段**    |   **网关**    |          ipv4地址池          |
| :-----------: | :------: | :------------: | :---------: | :-----------------------: |
| **LinNetInt** | **IPv4** | 172.16.20.0/24 | 172.16.20.1 | 172.16.20.2-172.16.20.252 |


2.云主机信息表：

| **云主机** | **环境名称** |   **IPv4地址**   |   **完全合格域名**    |
| :-----: | :------: | :------------: | :-------------: |
| Rocky1  |  Rocky1  | LinNetInt：Auto | ls1.gdskills.cn |
| Rocky2  |  Rocky2  | LinNetInt：Auto | ls2.gdskills.cn |
| Rocky3  |  Rocky3  | LinNetInt：Auto | ls3.gdskills.cn |
| Rocky4  |  Rocky4  | LinNetInt：Auto | ls4.gdskills.cn |
| Kylin1  | KylinV10 | LinNetInt：Auto | ls5.gdskills.cn |
| Kylin2  | KylinV10 | LinNetInt：Auto | ls6.gdskills.cn |


本任务已提供操作系统镜像以及相关环境，所需软件安装包均在/root/目录下。涉及配置密码的步骤，除特殊说明外，密码均设置为Key-1122，创建完成云主机后，将所有实例从DHCP获取的地址改为静态地址。

**（二）DNS服务**

任务描述：创建dns服务器，实现企业域名访问。

1.配置所有rocky主机和kylin主机的IP地址和主机名称。



2.所有Rocky(ls3-ls4除外)和Kylin(ls5除外)主机启用防火墙防火墙区域为public，在防火墙中放行对应服务端口。



3.利用chrony，配置ls1为其他linux主机提供ntp服务。



4.所有linux主机root用户使用完全合格域名免密码ssh登录到其他linux主机。



5.利用bind和bind-utils，配置ls1为主dns根服务器，区域文件为/var/named/named.root，ls2为备用dns服务器。为所有linux主机提供冗余dns正反向解析服务。正向区域文件均为/var/named/named.gdskills，反向区域文件均为/var/named/named.20。



6.创建DNS响应区域，区域名称为rpz.zone，响应区域文件名称为/var/named/rpz.zone。当客户端解析 /www.yellow.com /www.dubo.com /www.duping.com 时均返回NXDOMAIN。



6.配置ls1为CA服务器,为linux主机颁发证书。证书颁发机构有效期10年，公用名为：“GDGlobalSignROOTCA”。申请并颁发一张供linux服务器使用的证书，证书信息：有效期=5年，公用名=gdskills.cn，国家=CN，省=Guangdong，城市=Guangzhou，组织=gdskills，组织单位=system，使用者可选名称=*.gdskills.cn和gdskills.cn。将证书skills.crt和私钥skills.key复制到需要证书的linux服务器/etc/pki/tls目录。浏览器访问https网站时，不出现证书警告信息。



7.在ls1上安装系统自带的ansible-core，作为ansible的控制节点。ls2-ls6作为ansible的受控节点，受控节点组名称为web，设置ansible_python_interpreter为/usr/bin/python3。



8.在ls1编写/root/web-deploy.yml剧本（提示：可以使用ansible-doc命令查询相关模块），在所有受控主机上安装httpd服务，将监听端口修改为8081，启动并设置为开机自启；各主机站点首页内容统一为：Hello,thisis"hostname"site!!!（例如：Hello, thisis"ls2.gdskills.cn"site!!!）



**（三）高可用服务**

任务描述：利用高可用架构，搭建Tomcat动态网站。

1.配置ls3和ls4为tomcat服务器，网站默认首页内容分别为“TomcatA”和“TomcatB”，Tomcat采用修改配置文件以HTTP 80端口的方式运行。



3.ls1和ls2安装并配置Keepalived。ls1的路由ID设置为ha1，ls2为ha2；指定用于VIP漂移的网卡名称；虚拟路由ID为26；身份验证方式为PASS，密码为Key-1122；设置ls1的优先级为100，ls2为90；防火墙放行相关协议。



4.ls1和ls2安装并配置HAProxy，实现后端Tomcat服务器的负载均衡。设置健康检查间隔为3000ms，当连续3次健康检查成功时，认为该Tomcat服务器可用；当连续5次健康检查失败时，认为该服务器不可用。启用redispatch功能，允许在服务器故障时重新分配请求。配置基于cookie的会话保持，cookie名称为SERVERID，为后端服务器ls3设置cookie值为TomcatServerA，为ls4设置cookie值为TomcatServerB。



5.对外仅允许tomcat.gdskills.cn域名访问，配置HTTP到HTTPS的永久重定向。当ls1或ls2任一节点故障时，VIP（172.16.20. 200/24）及负载均衡服务能自动切换至另一正常节点，保证服务连续性。



**（四）iSCSI服务**

任务描述：请采用iSCSI，实现集中管理存储。

1.在ls2上使用dd命令在/opt目录下创建两个大小为5G，名称为file1和file2的文件，并将其以/dev/loop10、/dev/loop11设备进行挂载，利用lvm2创建lvm，卷组名称为vg1，逻辑卷名称为lv1，容量为全部，格式化为ext4格式。使用/dev/vg1/lv1配置为iscsi目标服务器。



2.iscsi目标端的wwn为iqn.2026-03.cn.gdskills:server, iscsi发起端的wwn为iqn.2026-03.cn.gdskills:client。



3.ls6连接ls2上的iscsi磁盘，修改/etc/rc.d/rc.local文件，实现开机自动挂载ls2上的iscsi磁盘到/shareiscsi目录。



**（五）邮件服务**

任务描述：请采用postfix邮件服务器，实现安全的邮件服务。

1.配置ls3为邮件服务器，安装postfix和dovecot。仅允许smtps和pop3s连接。向 all@gdskills.cn发送邮件时，mail1和mail2用户都会收到。

2.使用本机测试。



**（六）****Mariadb****服务**

任务描述：请采用Mariadb数据库，实现数据的高效存储与处理。

1.配置ls5和ls6为mariadb主从服务器，创建数据库用户xiao，只允许对userdb数据库拥有完全权限。设置ls5服务器ID为1，ls6为2。



2.创建数据库userdb；在数据库中创建表userinfo，在表中插入2条记录，分别为(1,user1,1.61,2026-01-12,F)，(2,user2,1.62,2026-01-13,M)，口令与用户名相同，password字段用md5函数加密，

表结构如下：

| **字段名** | **数据类型** | **主键** | **自增** |
| :---: | :---: | :---: | :---: |
| id | int | 是 | 是 |
| name | varchar(10) | 否 | 否 |
| height | float | 否 | 否 |
| birthday | datetime | 否 | 否 |
| sex | varchar(5) | 否 | 否 |
| password | char(200) | 否 | 否 |


3.新建/var/mariadb/userinfo.txt文件，文件内容如下，然后

将文件内容导入到userinfo表中，password字段使用md5函数加密。

3,user3,1.63,2026-03-13,F,user3

4,user4,1.64,2026-03-14,M,user4

5,user5,1.65,2026-03-15,M,user5

6,user6,1.66,2026-03-16,F,user6

7,user7,1.67,2026-03-17,F,user7

8,user8,1.68,2026-03-18,M,user8

9,user9,1.69,2026-03-19,F,user9



4.将表userinfo中的记录导出，并存放到 /var/mariadb/userinfo.sql，字段之间使用','分隔。



5.为root用户创建计划任务（day使用数字表示），每周五凌晨2:00，备份数据库 userdb（含创建数据库命令）到 /var/mariadb/userdb.sql。（提示：为便于测试，请手动备份一次。）**（七）DHCP服务**

任务描述：请采用DHCP服务器，实现ip地址及其他网络参数动态分配。

1.在ls2上安装DHCP服务，地址范围为172.16.20.10-172.16.20.19，网关为172.16.20.1，dns为ls1和ls2，域名为gdskills.cn。

**（八）****Kubernetes****服务**

任务描述：请采用kubernetes和containerd，管理容器。

1.在ls3-ls5上安装containerd和kubernetes（提示，若ls5无法安装，请先安装kylin_extra中的软件包），ls3作为masternode，ls4和ls5作为worknode；containerd的namespace为k8s.io。



2.使用containerd.sock作为容器runtime-endpoint。初始化节点时，pod_network为10.244.0.0/16，service_cidr为10.96.0.0/16，为master节点配置calico，作为网络组件。



3.为节点添加标签，ls4添加标签为os=rocky，ls5为os=kylin。



4.导入rockylinux9和ubuntu24.04镜像；使用NodeSelector将rockylinux9和ubuntu24.04分别调度到kylin和rocky节点上，副本数为2。



**（九）审计服务**

任务描述：请采用audit，实现审计服务器的所有操作。

1.在ls4上安装audit。配置审计日志路径为 /var/log/audit/audit.log；最大日志文件大小为10M；保留10个副本。



**（十）开发服务**

任务描述：请配置开发环境，实现源代码集中管理。

1.在ls3上安装gcc、rust、golang，在/root/soft目录中将提供的main.c、main.rust、main.go依次编译为info_c、info_r、info_g二进制文件并将其存放至/usr/sbin目录中（提示：为便于测试，请分别手动执行一次）。



**（十一）备份服务**

任务描述：请采用rsync备份工具，实现数据备份与增量备份。

1.在ls2上安装rsync，并在/etc目录下创建排除文件rsync_exclude.txt，排除log/、debug/文件夹，以及log.txt、debug.txt文件。



2.将ls5和ls6的/var/lib/mysql目录备份到本地/backup/ls6/目录中，实现数据备份与增量备份，排除指定目录和文件。

