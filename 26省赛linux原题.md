<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/53941428/1774401559339-2979794a-0697-4807-994e-6ef29968f295.png)

**<font style="color:black;">（一）检查实例环境</font>**

<font style="color:black;">1.</font><font style="color:black;">网络管理信息表：</font>

| **<font style="color:black;">网络名称</font>** | **<font style="color:black;">协议版本</font>** | | **<font style="color:black;">子网网段</font>** | **<font style="color:black;">网关</font>** | **<font style="color:black;">ipv4</font>****<font style="color:black;">地址池</font>** |
| :---: | :---: | --- | :---: | :---: | :---: |
| **<font style="color:black;">LinNetInt</font>** | **<font style="color:black;">IPv4</font>** | <font style="color:black;">172.16.20.0/24</font> | | <font style="color:black;">172.16.20.1</font> | <font style="color:black;">172.16.20.2-172.16.20.252</font> |


<font style="color:black;">2.</font><font style="color:black;">云主机信息表：</font>

| **<font style="color:black;">云主机</font>** | **<font style="color:black;">环境名称</font>** | **<font style="color:black;">IPv4</font>****<font style="color:black;">地址</font>** | **<font style="color:black;">完全合格域名</font>** |
| :---: | :---: | :---: | :---: |
| <font style="color:black;">Rocky1</font> | <font style="color:black;">Rocky1</font> | <font style="color:black;">LinNetInt</font><font style="color:black;">：</font><font style="color:black;">Auto</font> | <font style="color:black;">ls1.gdskills.cn</font> |
| <font style="color:black;">Rocky2</font> | <font style="color:black;">Rocky2</font> | <font style="color:black;">LinNetInt</font><font style="color:black;">：</font><font style="color:black;">Auto</font> | <font style="color:black;">ls2.gdskills.cn</font> |
| <font style="color:black;">Rocky3</font> | <font style="color:black;">Rocky3</font> | <font style="color:black;">LinNetInt</font><font style="color:black;">：</font><font style="color:black;">Auto</font> | <font style="color:black;">ls3.gdskills.cn</font> |
| <font style="color:black;">Rocky4</font> | <font style="color:black;">Rocky4</font> | <font style="color:black;">LinNetInt</font><font style="color:black;">：</font><font style="color:black;">Auto</font> | <font style="color:black;">ls4.gdskills.cn</font> |
| <font style="color:black;">Kylin1</font> | <font style="color:black;">KylinV10</font> | <font style="color:black;">LinNetInt</font><font style="color:black;">：</font><font style="color:black;">Auto</font> | <font style="color:black;">ls5.gdskills.cn</font> |
| <font style="color:black;">Kylin2</font> | <font style="color:black;">KylinV10</font> | <font style="color:black;">LinNetInt</font><font style="color:black;">：</font><font style="color:black;">Auto</font> | <font style="color:black;">ls6.gdskills.cn</font> |


<font style="color:black;">本任务已提供操作系统镜像以及相关环境，所需软件安装包均在</font><font style="color:black;">/root/</font><font style="color:black;">目录下。涉及配置密码的步骤，除特殊说明外，密码均设置为</font><font style="color:black;">Key-1122</font><font style="color:black;">，创建完成云主机后，将所有实例从</font><font style="color:black;">DHCP</font><font style="color:black;">获取的地址改为静态地址。</font>

**<font style="color:black;">（二）</font>****<font style="color:black;">DNS</font>****<font style="color:black;">服务</font>**

<font style="color:black;">任务描述：创建</font><font style="color:black;">dns</font><font style="color:black;">服务器，实现企业域名访问。</font>

<font style="color:black;">1.</font><font style="color:black;">配置所有</font><font style="color:black;">rocky</font><font style="color:black;">主机和</font><font style="color:black;">kylin</font><font style="color:black;">主机的</font><font style="color:black;">IP</font><font style="color:black;">地址和主机名称。</font>



<font style="color:black;">2.</font><font style="color:black;">所有</font><font style="color:black;">Rocky(ls3-ls4</font><font style="color:black;">除外</font><font style="color:black;">)</font><font style="color:black;">和</font><font style="color:black;">Kylin(ls5</font><font style="color:black;">除外</font><font style="color:black;">)</font><font style="color:black;">主机启用防火墙防火墙区域为</font><font style="color:black;">public</font><font style="color:black;">，在防火墙中放行对应服务端口。</font>

<font style="color:black;"></font>

<font style="color:black;">3.</font><font style="color:black;">利用</font><font style="color:black;">chrony</font><font style="color:black;">，配置</font><font style="color:black;">ls1</font><font style="color:black;">为其他</font><font style="color:black;">linux</font><font style="color:black;">主机提供</font><font style="color:black;">ntp</font><font style="color:black;">服务。</font>



<font style="color:black;">4.</font><font style="color:black;">所有</font><font style="color:black;">linux</font><font style="color:black;">主机</font><font style="color:black;">root</font><font style="color:black;">用户使用完全合格域名免密码</font><font style="color:black;">ssh</font><font style="color:black;">登录到其他</font><font style="color:black;">linux</font><font style="color:black;">主机。</font>

<font style="color:black;"></font>

<font style="color:black;">5.</font><font style="color:black;">利用</font><font style="color:black;">bind</font><font style="color:black;">和</font><font style="color:black;">bind-utils</font><font style="color:black;">，配置</font><font style="color:black;">ls1</font><font style="color:black;">为主</font><font style="color:black;">dns</font><font style="color:black;">根服务器，区域文件为</font><font style="color:black;">/var/named/named.root</font><font style="color:black;">，</font><font style="color:black;">ls2</font><font style="color:black;">为备用</font><font style="color:black;">dns</font><font style="color:black;">服务器。为所有</font><font style="color:black;">linux</font><font style="color:black;">主机提供冗余</font><font style="color:black;">dns</font><font style="color:black;">正反向解析服务。正向区域文件均为</font><font style="color:black;">/var/named/named.gdskills</font><font style="color:black;">，反向区域文件均为</font><font style="color:black;">/var/named/named.20</font><font style="color:black;">。</font>

<font style="color:black;"></font>

<font style="color:black;">6.</font><font style="color:black;">创建</font><font style="color:black;">DNS</font><font style="color:black;">响应区域，区域名称为</font><font style="color:black;">rpz.zone</font><font style="color:black;">，响应区域文件名称为</font><font style="color:black;">/var/named/rpz.zone</font><font style="color:black;">。当客户端解析</font>                          <font style="color:black;">www.yellow.com</font><font style="color:black;">、</font><font style="color:black;">www.dubo.com</font><font style="color:black;">、</font><font style="color:black;">www.duping.com</font><font style="color:black;">时均返回</font><font style="color:black;">NXDOMAIN</font><font style="color:black;">。</font>

<font style="color:black;"></font>

<font style="color:black;">6.</font><font style="color:black;">配置</font><font style="color:black;">ls1</font><font style="color:black;">为</font><font style="color:black;">CA</font><font style="color:black;">服务器</font><font style="color:black;">,</font><font style="color:black;">为</font><font style="color:black;">linux</font><font style="color:black;">主机颁发证书。证书颁发机构有效期</font><font style="color:black;">10</font><font style="color:black;">年，公用名为：</font><font style="color:black;">“GDGlobalSignROOTCA”</font><font style="color:black;">。申请并颁发一张供</font><font style="color:black;">linux</font><font style="color:black;">服务器使用的证书，证书信息：有效期</font><font style="color:black;">=5</font><font style="color:black;">年，公用名</font><font style="color:black;">=gdskills.cn</font><font style="color:black;">，国家</font><font style="color:black;">=CN</font><font style="color:black;">，省</font><font style="color:black;">=Guangdong</font><font style="color:black;">，城市</font><font style="color:black;">=Guangzhou</font><font style="color:black;">，组织</font><font style="color:black;">=gdskills</font><font style="color:black;">，组织单位</font><font style="color:black;">=system</font><font style="color:black;">，使用者可选名称</font><font style="color:black;">=*.gdskills.cn</font><font style="color:black;">和</font><font style="color:black;">gdskills.cn</font><font style="color:black;">。将证书</font><font style="color:black;">skills.crt</font><font style="color:black;">和私钥</font><font style="color:black;">skills.key</font><font style="color:black;">复制到需要证书的</font><font style="color:black;">linux</font><font style="color:black;">服务器</font><font style="color:black;">/etc/pki/tls</font><font style="color:black;">目录。浏览器访问</font><font style="color:black;">https</font><font style="color:black;">网站时，不出现证书警告信息。</font>

<font style="color:black;"></font>

<font style="color:black;">7.</font><font style="color:black;">在</font><font style="color:black;">ls1</font><font style="color:black;">上安装系统自带的</font><font style="color:black;">ansible-core</font><font style="color:black;">，作为</font><font style="color:black;">ansible</font><font style="color:black;">的控制节点。</font><font style="color:black;">ls2-ls6</font><font style="color:black;">作为</font><font style="color:black;">ansible</font><font style="color:black;">的受控节点，受控节点组名称为</font><font style="color:black;">web</font><font style="color:black;">，设置</font><font style="color:black;">ansible_python_interpreter</font><font style="color:black;">为</font><font style="color:black;">/usr/bin/python3</font><font style="color:black;">。</font>

<font style="color:black;"></font>

<font style="color:black;">8.在ls1编写/root/web-deploy.yml剧本（提示：可以使用ansible-doc命令查询相关模块），在所有受控主机上安装httpd服务，将监听端口修改为8081，启动并设置为开机自启；各主机站点首页内容统一为：Hello,thisis"hostname"site!!!（例如：Hello, thisis"ls2.gdskills.cn"site!!!）</font>

<font style="color:black;"></font>

**<font style="color:black;">（三）高可用服务</font>**

<font style="color:black;">任务描述：利用高可用架构，搭建</font><font style="color:black;">Tomcat</font><font style="color:black;">动态网站。</font>

<font style="color:black;">1.</font><font style="color:black;">配置</font><font style="color:black;">ls3</font><font style="color:black;">和</font><font style="color:black;">ls4</font><font style="color:black;">为</font><font style="color:black;">tomcat</font><font style="color:black;">服务器，网站默认首页内容分别为</font><font style="color:black;">“TomcatA”</font><font style="color:black;">和</font><font style="color:black;">“TomcatB”</font><font style="color:black;">，</font><font style="color:black;">Tomcat</font><font style="color:black;">采用修改配置文件以</font><font style="color:black;">HTTP 80</font><font style="color:black;">端口的方式运行。</font>

<font style="color:black;"></font>

<font style="color:black;">3.ls1</font><font style="color:black;">和</font><font style="color:black;">ls2</font><font style="color:black;">安装并配置</font><font style="color:black;">Keepalived</font><font style="color:black;">。</font><font style="color:black;">ls1</font><font style="color:black;">的路由</font><font style="color:black;">ID</font><font style="color:black;">设置为</font><font style="color:black;">ha1</font><font style="color:black;">，</font><font style="color:black;">ls2</font><font style="color:black;">为</font><font style="color:black;">ha2</font><font style="color:black;">；指定用于</font><font style="color:black;">VIP</font><font style="color:black;">漂移的网卡名称；虚拟路由</font><font style="color:black;">ID</font><font style="color:black;">为</font><font style="color:black;">26</font><font style="color:black;">；身份验证方式为</font><font style="color:black;">PASS</font><font style="color:black;">，密码为</font><font style="color:black;">Key-1122</font><font style="color:black;">；设置</font><font style="color:black;">ls1</font><font style="color:black;">的优先级为</font><font style="color:black;">100</font><font style="color:black;">，</font><font style="color:black;">ls2</font><font style="color:black;">为</font><font style="color:black;">90</font><font style="color:black;">；防火墙放行相关协议。</font>

<font style="color:black;"></font>

<font style="color:black;">4.ls1</font><font style="color:black;">和</font><font style="color:black;">ls2</font><font style="color:black;">安装并配置</font><font style="color:black;">HAProxy</font><font style="color:black;">，实现后端</font><font style="color:black;">Tomcat</font><font style="color:black;">服务器的负载均衡。设置健康检查间隔为</font><font style="color:black;">3000ms</font><font style="color:black;">，当连续</font><font style="color:black;">3</font><font style="color:black;">次健康检查成功时，认为该</font><font style="color:black;">Tomcat</font><font style="color:black;">服务器可用；当连续</font><font style="color:black;">5</font><font style="color:black;">次健康检查失败时，认为该服务器不可用。启用</font><font style="color:black;">redispatch</font><font style="color:black;">功能，允许在服务器故障时重新分配请求。配置基于</font><font style="color:black;">cookie</font><font style="color:black;">的会话保持，</font><font style="color:black;">cookie</font><font style="color:black;">名称为</font><font style="color:black;">SERVERID</font><font style="color:black;">，为后端服务器</font><font style="color:black;">ls3</font><font style="color:black;">设置</font><font style="color:black;">cookie</font><font style="color:black;">值为</font><font style="color:black;">TomcatServerA</font><font style="color:black;">，为</font><font style="color:black;">ls4</font><font style="color:black;">设置</font><font style="color:black;">cookie</font><font style="color:black;">值为</font><font style="color:black;">TomcatServerB</font><font style="color:black;">。</font>

<font style="color:black;"></font>

<font style="color:black;">5.</font><font style="color:black;">对外仅允许</font><font style="color:black;">tomcat.gdskills.cn</font><font style="color:black;">域名访问，配置</font><font style="color:black;">HTTP</font><font style="color:black;">到</font><font style="color:black;">HTTPS</font><font style="color:black;">的永久重定向。当</font><font style="color:black;">ls1</font><font style="color:black;">或</font><font style="color:black;">ls2</font><font style="color:black;">任一节点故障时，</font><font style="color:black;">VIP</font><font style="color:black;">（</font><font style="color:black;">172.16.20. 200/24</font><font style="color:black;">）及负载均衡服务能自动切换至另一正常节点，保证服务连续性。</font>

<font style="color:black;"></font>

**<font style="color:black;">（四）iSCSI服务</font>**

<font style="color:black;">任务描述：请采用</font><font style="color:black;">iSCSI</font><font style="color:black;">，实现集中管理存储。</font>

<font style="color:black;">1.</font><font style="color:black;">在</font><font style="color:black;">ls2</font><font style="color:black;">上使用</font><font style="color:black;">dd</font><font style="color:black;">命令在</font><font style="color:black;">/opt</font><font style="color:black;">目录下创建两个大小为</font><font style="color:black;">5G</font><font style="color:black;">，名称为</font><font style="color:black;">file1</font><font style="color:black;">和</font><font style="color:black;">file2</font><font style="color:black;">的文件，并将其以</font><font style="color:black;">/dev/loop10</font><font style="color:black;">、</font><font style="color:black;">/dev/loop11</font><font style="color:black;">设备进行挂载，利用</font><font style="color:black;">lvm2</font><font style="color:black;">创建</font><font style="color:black;">lvm</font><font style="color:black;">，卷组名称为</font><font style="color:black;">vg1</font><font style="color:black;">，逻辑卷名称为</font><font style="color:black;">lv1</font><font style="color:black;">，容量为全部，格式化为</font><font style="color:black;">ext4</font><font style="color:black;">格式。使用</font><font style="color:black;">/dev/vg1/lv1</font><font style="color:black;">配置为</font><font style="color:black;">iscsi</font><font style="color:black;">目标服务器。</font>

<font style="color:black;"></font>

<font style="color:black;">2.iscsi</font><font style="color:black;">目标端的</font><font style="color:black;">wwn</font><font style="color:black;">为</font><font style="color:black;">iqn.2026-03.cn.gdskills:server, iscsi</font><font style="color:black;">发起端的</font><font style="color:black;">wwn</font><font style="color:black;">为</font><font style="color:black;">iqn.2026-03.cn.gdskills:client</font><font style="color:black;">。</font>

<font style="color:black;"></font>

<font style="color:black;">3.ls6</font><font style="color:black;">连接</font><font style="color:black;">ls2</font><font style="color:black;">上的</font><font style="color:black;">iscsi</font><font style="color:black;">磁盘，修改</font><font style="color:black;">/etc/rc.d/rc.local</font><font style="color:black;">文件，实现开机自动挂载</font><font style="color:black;">ls2</font><font style="color:black;">上的</font><font style="color:black;">iscsi</font><font style="color:black;">磁盘到</font><font style="color:black;">/shareiscsi</font><font style="color:black;">目录。</font>

<font style="color:black;"></font>

**<font style="color:black;">（五）邮件服务</font>**

<font style="color:black;">任务描述：请采用</font><font style="color:black;">postfix</font><font style="color:black;">邮件服务器，实现安全的邮件服务。</font>

<font style="color:black;">1.</font><font style="color:black;">配置</font><font style="color:black;">ls3</font><font style="color:black;">为邮件服务器，安装</font><font style="color:black;">postfix</font><font style="color:black;">和</font><font style="color:black;">dovecot</font><font style="color:black;">。仅允许</font><font style="color:black;">smtps</font><font style="color:black;">和</font><font style="color:black;">pop3s</font><font style="color:black;">连接。向</font><font style="color:black;">all@gdskills.cn</font><font style="color:black;">发送邮件时，</font><font style="color:black;">mail1</font><font style="color:black;">和</font><font style="color:black;">mail2</font><font style="color:black;">用户都会收到。</font>

<font style="color:black;">2.</font><font style="color:black;">使用本机测试。</font>

<font style="color:black;"></font>

**<font style="color:black;">（六）</font>****<font style="color:black;">Mariadb</font>****<font style="color:black;">服务</font>**

<font style="color:black;">任务描述：请采用</font><font style="color:black;">Mariadb</font><font style="color:black;">数据库，实现数据的高效存储与处理。</font>

<font style="color:black;">1.</font><font style="color:black;">配置</font><font style="color:black;">ls5</font><font style="color:black;">和</font><font style="color:black;">ls6</font><font style="color:black;">为</font><font style="color:black;">mariadb</font><font style="color:black;">主从服务器，创建数据库用户</font><font style="color:black;">xiao</font><font style="color:black;">，只允许对</font><font style="color:black;">userdb</font><font style="color:black;">数据库拥有完全权限。设置</font><font style="color:black;">ls5</font><font style="color:black;">服务器</font><font style="color:black;">ID</font><font style="color:black;">为</font><font style="color:black;">1</font><font style="color:black;">，</font><font style="color:black;">ls6</font><font style="color:black;">为</font><font style="color:black;">2</font><font style="color:black;">。</font>

<font style="color:black;"></font>

<font style="color:black;">2.创建数据库userdb；在数据库中创建表userinfo，在表中插入2条记录，分别为(1,user1,1.61,2026-01-12,F)，(2,user2,1.62,2026-01-13,M)，口令与用户名相同，password字段用md5函数加密，</font>

<font style="color:black;">表结构如下：</font>

| **<font style="color:black;">字段名</font>** | **<font style="color:black;">数据类型</font>** | **<font style="color:black;">主键</font>** | **<font style="color:black;">自增</font>** |
| :---: | :---: | :---: | :---: |
| <font style="color:black;">id</font> | <font style="color:black;">int</font> | <font style="color:black;">是</font> | <font style="color:black;">是</font> |
| <font style="color:black;">name</font> | <font style="color:black;">varchar(10)</font> | <font style="color:black;">否</font> | <font style="color:black;">否</font> |
| <font style="color:black;">height</font> | <font style="color:black;">float</font> | <font style="color:black;">否</font> | <font style="color:black;">否</font> |
| <font style="color:black;">birthday</font> | <font style="color:black;">datetime</font> | <font style="color:black;">否</font> | <font style="color:black;">否</font> |
| <font style="color:black;">sex</font> | <font style="color:black;">varchar(5)</font> | <font style="color:black;">否</font> | <font style="color:black;">否</font> |
| <font style="color:black;">password</font> | <font style="color:black;">char(200)</font> | <font style="color:black;">否</font> | <font style="color:black;">否</font> |


<font style="color:black;">3.</font><font style="color:black;">新建</font><font style="color:black;">/var/mariadb/userinfo.txt</font><font style="color:black;">文件，文件内容如下，然后</font>

<font style="color:black;">将文件内容导入到</font><font style="color:black;">userinfo</font><font style="color:black;">表中，</font><font style="color:black;">password</font><font style="color:black;">字段使用</font><font style="color:black;">md5</font><font style="color:black;">函数加密。</font>

<font style="color:black;">3,user3,1.63,2026-03-13,F,user3</font>

<font style="color:black;">4,user4,1.64,2026-03-14,M,user4</font>

<font style="color:black;">5,user5,1.65,2026-03-15,M,user5</font>

<font style="color:black;">6,user6,1.66,2026-03-16,F,user6</font>

<font style="color:black;">7,user7,1.67,2026-03-17,F,user7</font>

<font style="color:black;">8,user8,1.68,2026-03-18,M,user8</font>

<font style="color:black;">9,user9,1.69,2026-03-19,F,user9</font>

<font style="color:black;"></font>

<font style="color:black;">4.将表userinfo中的记录导出，并存放到 /var/mariadb/userinfo.sql，字段之间使用','分隔。</font>

<font style="color:black;"></font>

<font style="color:black;">5.为root用户创建计划任务（day使用数字表示），每周五凌晨2:00，备份数据库 userdb（含创建数据库命令）到 /var/mariadb/userdb.sql。（提示：为便于测试，请手动备份一次。）</font>**<font style="color:black;">（七）DHCP服务</font>**

<font style="color:black;">任务描述：请采用DHCP服务器，实现ip地址及其他网络参数动态分配。</font>

<font style="color:black;">1.在ls2上安装DHCP服务，地址范围为172.16.20.10-172.16.20.19，网关为172.16.20.1，dns为ls1和ls2，域名为gdskills.cn。</font>

**<font style="color:black;">（八）</font>****<font style="color:black;">Kubernetes</font>****<font style="color:black;">服务</font>**

<font style="color:black;">任务描述：请采用kubernetes和containerd，管理容器。</font>

<font style="color:black;">1.</font><font style="color:black;">在</font><font style="color:black;">ls3-ls5</font><font style="color:black;">上安装</font><font style="color:black;">containerd</font><font style="color:black;">和</font><font style="color:black;">kubernetes</font><font style="color:black;">（提示，若</font><font style="color:black;">ls5</font><font style="color:black;">无法安装，请先安装</font><font style="color:black;">kylin_extra</font><font style="color:black;">中的软件包），</font><font style="color:black;">ls3</font><font style="color:black;">作为</font><font style="color:black;">masternode</font><font style="color:black;">，</font><font style="color:black;">ls4</font><font style="color:black;">和</font><font style="color:black;">ls5</font><font style="color:black;">作为</font><font style="color:black;">worknode</font><font style="color:black;">；</font><font style="color:black;">containerd</font><font style="color:black;">的</font><font style="color:black;">namespace</font><font style="color:black;">为</font><font style="color:black;">k8s.io</font><font style="color:black;">。</font>

<font style="color:black;"></font>

<font style="color:black;">2.</font><font style="color:black;">使用</font><font style="color:black;">containerd.sock</font><font style="color:black;">作为容器</font><font style="color:black;">runtime-endpoint</font><font style="color:black;">。初始化节点时，</font><font style="color:black;">pod_network</font><font style="color:black;">为</font><font style="color:black;">10.244.0.0/16</font><font style="color:black;">，</font><font style="color:black;">service_cidr</font><font style="color:black;">为</font><font style="color:black;">10.96.0.0/16</font><font style="color:black;">，为</font><font style="color:black;">master</font><font style="color:black;">节点配置</font><font style="color:black;">calico</font><font style="color:black;">，作为网络组件。</font>

<font style="color:black;"></font>

<font style="color:black;">3.</font><font style="color:black;">为节点添加标签，</font><font style="color:black;">ls4</font><font style="color:black;">添加标签为</font><font style="color:black;">os=rocky</font><font style="color:black;">，</font><font style="color:black;">ls5</font><font style="color:black;">为</font><font style="color:black;">os=kylin</font><font style="color:black;">。</font>

<font style="color:black;"></font>

<font style="color:black;">4.</font><font style="color:black;">导入</font><font style="color:black;">rockylinux9</font><font style="color:black;">和</font><font style="color:black;">ubuntu24.04</font><font style="color:black;">镜像；使用</font><font style="color:black;">NodeSelector</font><font style="color:black;">将</font><font style="color:black;">rockylinux9</font><font style="color:black;">和</font><font style="color:black;">ubuntu24.04</font><font style="color:black;">分别调度到</font><font style="color:black;">kylin</font><font style="color:black;">和</font><font style="color:black;">rocky</font><font style="color:black;">节点上，副本数为</font><font style="color:black;">2</font><font style="color:black;">。</font>

<font style="color:black;"></font>

**<font style="color:black;">（九）审计服务</font>**

<font style="color:black;">任务描述：请采用audit，实现审计服务器的所有操作。</font>

<font style="color:black;">1.在ls4上安装audit。配置审计日志路径为 /var/log/audit/audit.log；最大日志文件大小为10M；保留10个副本。</font>



**<font style="color:black;">（十）开发服务</font>**

<font style="color:black;">任务描述：请配置开发环境，实现源代码集中管理。</font>

<font style="color:black;">1.</font><font style="color:black;">在</font><font style="color:black;">ls3</font><font style="color:black;">上安装</font><font style="color:black;">gcc</font><font style="color:black;">、</font><font style="color:black;">rust</font><font style="color:black;">、</font><font style="color:black;">golang</font><font style="color:black;">，在</font><font style="color:black;">/root/soft</font><font style="color:black;">目录中将提供的</font><font style="color:black;">main.c</font><font style="color:black;">、</font><font style="color:black;">main.rust</font><font style="color:black;">、</font><font style="color:black;">main.go</font><font style="color:black;">依次编译为</font><font style="color:black;">info_c</font><font style="color:black;">、</font><font style="color:black;">info_r</font><font style="color:black;">、</font><font style="color:black;">info_g</font><font style="color:black;">二进制文件并将其存放至</font><font style="color:black;">/usr/sbin</font><font style="color:black;">目录中（提示：为便于测试，请分别手动执行一次）。</font>

<font style="color:black;"></font>

**<font style="color:black;">（十一）备份服务</font>**

<font style="color:black;">任务描述：请采用</font><font style="color:black;">rsync</font><font style="color:black;">备份工具，实现数据备份与增量备份。</font>

<font style="color:black;">1.在ls2上安装rsync，并在/etc目录下创建排除文件rsync_exclude.txt，排除log/、debug/文件夹，以及log.txt、debug.txt文件。</font>

<font style="color:black;"></font>

<font style="color:black;">2.将ls5和ls6的/var/lib/mysql目录备份到本地/backup/ls6/目录中，实现数据备份与增量备份，排除指定目录和文件。</font>

