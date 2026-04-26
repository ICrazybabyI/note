
| **云主机** | **环境名称** |    **IPv4地址**     |    **完全合格域名**     |
| :-----: | :------: | :---------------: | :---------------: |
| linux1  |  linux1  | 192.168.31.221/24 | linux1.skills.lan |
| linux2  |  linux2  | 192.168.31.222/24 | linux2.skills.lan |
| linux3  |  linux3  | 192.168.31.223/24 | linux3.skills.lan |
| linux4  |  linux4  | 192.168.31.224/24 | linux4.skills.lan |
| linux5  |  linux5  | 192.168.31.225/24 | linux5.skills.lan |
| linux6  |  linux6  | 192.168.31.226/24 | linux6.skills.lan |
## 做题准备:
先配置ip  
`nmtui`  
![222](images/Pasted%20image%2020260424111527.png)回车选择第一项  
![](images/Pasted%20image%2020260424111552.png)回车选择网卡编辑  
![406](images/Pasted%20image%2020260424111758.png)配置完成后用pagedown快速去到OK回车保存    

esc退出到主菜单  
![](images/Pasted%20image%2020260424111921.png)选择第二项  

![](images/Pasted%20image%2020260424112007.png)回车两次刷新配置  

![](images/Pasted%20image%2020260424112032.png)选择第三项设置系统名称  

![374](images/Pasted%20image%2020260424112118.png)回车确认  

以上步骤都使用该方法配置其他的主机名与ip   
配置完ip后编辑本地仓库源    
`win + x  a`              //使用windows的ssh工具连接6台主机  
ssh: `ssh -p 22 root@192.168.31.221`  
linux1-6:  
`rm -rf  /etc/yum.repos.d/*.repo`    //删除默认的网络源  
linux1:  
`vi /etc/yum.repos.d/1.repo`      //编写本地仓的配置__vim使用语法移步到[vim语法](vim语法.md)  
```bash
[1]
name=1
enable=1
baseurl=file:///mnt/1/BaseOS
gpgcheck=0
[2]
name=2
enable=1
baseurl=file:///mnt/1/AppStream
gpgcheck=0
```
:wq      //退出保存  
`cat /etc/yum.repos.d/1.repo`      //把输出的选中复制到linux2-6  
linux2-6:  
`vi /etc/yum.repos.d/1.repo`  
ctrl + v  
:wq  
linux1-6:  
`mkdir /mnt/1`  
`mount Rocky-9.2-x86_64-dvd.iso /mnt/1/`  
`dnf install bash* vim -y`
## 2.dns服务
## （1）所有linux主机启用防火墙，防火墙区域为public，在防火墙中放行对应服务端口。
## **默认开启，只需在做服务时放行其端口**

## （2）利用chrony，配置linux1为其他linux主机提供NTP服务。  
`--123/tcp/udp`	

**<u>先做ssh 再做这个</u>**

## linux1:
[root@linux1 ~]#`vi /etc/chrony.conf`

```bash
# Use public servers from the pool.ntp.org project.
# Please consider joining the pool (https://www.pool.ntp.org/join.html).
#pool 2.rocky.pool.ntp.org iburst			#注释掉

# Use NTP servers from DHCP.
sourcedir /run/chrony-dhcp

# Record the rate at which the system clock gains/losses time.
driftfile /var/lib/chrony/drift

# Allow the system clock to be stepped in the first three updates
# if its offset is larger than 1 second.
makestep 1.0 3

# Enable kernel synchronization of the real-time clock (RTC).
rtcsync

# Enable hardware timestamping on all interfaces that support it.
#hwtimestamp *

# Increase the minimum number of selectable sources required to adjust
# the system clock.
#minsources 2

# Allow NTP client access from local network.
allow 192.168.31.0/24								#写所在的网段

# Serve time even if not synchronized to a time source.
local stratum 10										#取消注释
```

[root@ls1 ~]#`systemctl restart chronyd`重启服务,应用配置

## linux2-9:
**vi /etc/chrony.conf**

```bash
3	 server 192.168.31.231 iburst
```

#### 编写脚本chrony.sh
```bash
for i in {3..9}
do
  scp /etc/chrony.conf 192.168.31.23$i:/etc/
done
```

---

## （3）所有linux主机之间（包含本主机）root用户实现密钥ssh认证，禁用密码认证。  

### linux1-9生成并发送ssh密钥：
```bash
ssh-keygen
ssh-copy-id **.**.**.**9              #密钥全部发送给一台主机，这台主机也要发给自己
scp .ssh/authorized_keys **.**.**.**:/root/.ssh/ #分发给各个主机
```



`vi /etc/ssh/sshd_config`

```bash
42 PermitRootLogin yes
45 PubkeyAuthentication yes
#允许公钥登录
65 PasswordAuthenticaation  no
#允许密码登录
```

## （4）利用bind，配置linux1为主DNS服务器，linux2为备用DNS服务器。为所有linux主机提供冗余DNS正反向解析服务。
``--53/tcp/udp``

## 主服务器：
``dnf install bind -y``  
``vi /etc/named.conf``

```c
11         listen-on port 53 { any; };
19         allow-query     { any; };
```

``vi /etc/named.rfc1912.zones``

```c
 17 zone "skills.lan" IN {
 18         type master;
 19         file "named.localhost";
 20         allow-update { 192.168.31.232; };
 21 };
 22 
 23 zone "31.168.192.in-addr.arpa" IN {
 24         type master;
 25         file "named.loopback";
 26         allow-update { 192.168.31.232; };
 27 };
```

cd /var/named/  
``vi dns.sh``

```bash
 for i in {1..9}
  do
echo "linux$i A 192.168.31.23$i" >> named.localhost 
echo "23$i PTR linux$i.skills.lan" >> named.loopback
  done
```

## 从服务器：
``dnf install bind -y``  
``vi /etc/named.conf``

```plain
11         listen-on port 53 { any; };
19         allow-query     { any; };
```

``vi /etc/named.rfc1912.zones`

`rndc retransfer skills.lan`  #重新同步指针

`systemctl restart named && systemctl enable named`



## （5）配置linux1为CA服务器,为linux主机颁发证书。证书颁发机构有效期10年，公用名为linux1.skills.lan。申请并颁发一张供linux服务器使用的证书，证书信息：有效期=5年，公用名=skills.lan，国家=CN，省=Beijing，城市=Beijing，组织=skills，组织单位=system，使用者可选名称=*.skills.lan和skills.lan。将证书skills.crt和私钥skills.key复制到需要证书的linux服务器/etc/ssl目录。浏览器访问https网站时，不出现证书警告信息。
## 做法一：


`dnf install openssl* -y`  
`cd /etc/pki/CA`  
`vi file.txt`

```plain
subjectAltName=DNS.1:*.skills.lan,DNS.2:skills.lan
```

`touch index.txt`  
`echo 00 > serial`

`openssl req -new -x509 -nodes -days 3650 -out cacert.pem -keyout private/cakey.pem -subj “/CN=linux1.skills.lan/C=CN/ST=Beijing/L=Beijing/O=skills/OU=system”`

`openssl req -new -days 1825 -out skills.csr -keyout skills.key -subj“/CN=skills.lan/C=CN/ST=Beijing/L=Beijing/O=skills/OU=system”`

`openssl ca -in skills.csr -out skills.crt -days 1825 -extfile file.txt`



## 做法二：
`dnf install openssl* -y`

`cd /etc/pki/CA/`

`touch index.txt`

`echo 00 > serial`

`openssl genrsa -out ca.key 2048`

`openssl req -new -out ca.csr -key ca.key`

依次输入：

国家：`CN`

省份：`Beijing`

城市：`Beijing`

组织：`skills`

组织单位：`system`

主机名：`linux1.skills.lan`

回车

回车

回车

`openssl x509 -req -days 3650 -in ca.csr -signkey ca.key -out ca.crt`

`vi /etc/pki/tls/openssl.cnf` ----正常模式下   冒号 + 行号   跳转

```plain
取消注释167 # req_extensions = v3_req # The extensions to add to a certificate request
       213 basicConstraints= CA:TRUE
       236 subjectAltName=@alt_names
       239 basicConstraints = CA:TRUE
       242 [alt_names]
       243 DNS.1=*.skills.lan
       244 DNS.2=skills.lan
```

<font style="color:rgb(201, 209, 217);background-color:rgb(16, 24, 40);">index.txt 是 OpenSSL CA 的证书数据库文件，</font>**<font style="color:rgb(201, 209, 217);background-color:rgb(16, 24, 40);">每颁发或吊销一个证书，都会在此文件中追加一条记录</font>**<font style="color:rgb(201, 209, 217);background-color:rgb(16, 24, 40);">。</font>

<font style="background-color:#FBDE28;">signkey</font><font style="color:rgb(201, 209, 217);background-color:rgb(16, 24, 40);">参数用于 </font>**<font style="color:rgb(201, 209, 217);background-color:rgb(16, 24, 40);">直接指定私钥来签署证书请求（CSR）</font>**<font style="color:rgb(201, 209, 217);background-color:rgb(16, 24, 40);"> ，通常用于生成 </font>**<font style="color:rgb(201, 209, 217);background-color:rgb(16, 24, 40);">自签名证书</font>**<font style="color:rgb(201, 209, 217);background-color:rgb(16, 24, 40);">。</font>

+ `**<font style="color:rgb(201, 209, 217);background-color:rgb(16, 24, 40);">CA:TRUE</font>**`<font style="color:rgb(201, 209, 217);background-color:rgb(16, 24, 40);">：声明该证书为可信的证书颁发机构，允许其签发下级证书或吊销列表（CRL）</font><font style="color:rgb(201, 209, 217);background-color:rgb(16, 24, 40);">。</font>
+ `**<font style="color:rgb(201, 209, 217);background-color:rgb(16, 24, 40);">CA:FALSE</font>**`<font style="color:rgb(201, 209, 217);background-color:rgb(16, 24, 40);">：限制证书仅作为终端实体（如Web服务器、客户端设备），禁止签发其他证书。</font>
+ `**<font style="color:rgb(201, 209, 217);background-color:rgb(16, 24, 40);">subjectAltName=@alt_names</font>**`<font style="color:rgb(201, 209, 217);background-color:rgb(16, 24, 40);"> 是一个用于定义 </font>**<font style="color:rgb(201, 209, 217);background-color:rgb(16, 24, 40);">主题备用名称（Subject Alternative Name, SAcatN）</font>**<font style="color:rgb(201, 209, 217);background-color:rgb(16, 24, 40);"> 的关键指令，它允许证书支持多个域名或 IP 地址，是现代化证书的必备扩展。</font>
+ <font style="color:rgb(201, 209, 217);background-color:rgb(16, 24, 40);">使用CA的证书和私钥签署服务器CSR</font>

`openssl genrsa -out skills.key 2048`

`openssl req -new -key skills.key -out skills.csr -config /etc/pki/tls/openssl.cnf -extensions v3_req`  
依次输入：

国家：`CN`

省份：`Beijing`

城市：`Beijing`

组织：`skills`

组织单位：`system`

主机名：`skills.lan`

回车

回车

回车

`openssl ca -in skills.csr -out skills.crt -cert ca.crt -keyfile ca.key -extensions v3_req -days 1825 -config /etc/pki/tls/openssl.cnf`

y

y



## 3.ansible服务  
任务描述：请采用ansible，实现自动化运维。  
（1）在linux1上安装ansible，作为ansible的控制节点。linux2-linux9作为ansible的受控节点。
``dnf  install ansible* -y``  
vi /etc/ansible/hosts

```plain
#末尾加
[1]
被控的ip
```

ansible -m ping all     #测试

``有可能会遇到本地语言问题 ，需要再下载en语言包``  
dnf install glibc-langpack-en -y  
localectl set-locale LANG="en_US.UTF-8"



## 4.apache2服务  
任务描述：请采用Apache搭建企业网站。  
（1）配置linux1为Apache2服务器，使用skills.lan或any.skills.lan（any代表任意网址前缀，用linux1.skills.lan和web.skills.lan测试）访问时，自动跳转到www.skills.lan。禁止使用IP地址访问，默认首页文档/var/www/html/index.html的内容为&quot;apache&quot;。
## --{80,443}/tcp
`dnf install httpd* mod_ssl -y`  
``cd /etc/httpd/conf.d/``  
``cp ssl.conf 1.conf``    --择善而从，不用背的多

`vim 1.conf`

```plain
<virtualhost *:80>
servername www.skills.lan
serveralias *.skills.lan
rewriteengine on
rewriterule ^(.*)$ https://www.skills.lan/$1 [L,R=301]
</virtualhost>

<virtualhost *:80>
servername 10.1.220.102
<location />
order allow,deny
deny from all
</location>

<VirtualHost *:443>
servername www.skills.lan
serveralias *.skills.lan
DocumentRoot "/var/www/html"
SSLEngine on
SSLCertificateFile /etc/ssl/skills.crt
SSLCertificateKeyFile /etc/ssl/skills.key
SSLCACertificateFile /etc/pki/CA/cacert.pem
SSLVerifyClient require
SSLVerifyDepth  10
</VirtualHost>

<virtualhost *:443>
servername 10.1.220.102
SSLCertificateFile /etc/ssl/skills.crt
SSLCertificateKeyFile /etc/ssl/skills.key
<location />
order allow,deny
deny from all
</location>
</virtualhost>
```



`systemctl enable httpd --now`  		#开机自启动httpd服务,现在就启动



如果要用域名访问要去DNS修改正向文件

## （2）把/etc/ssl/skills.crt证书文件和/etc/ssl/skills.key私钥文件转换成含有证书和私钥的/etc/ssl/skills.pfx文件；然后把/etc/ssl/skills.pfx转换为含有证书和私钥的/etc/ssl/skills.pem文件，再从/etc/ssl/skills.pem文件中提取证书和私钥分别到/etc/ssl/apache.crt和/etc/ssl/apache.key。
cd  /etc/ssl/  
`openssl pkcs12 -export -in skills.crt -inkey skills.key -out skills.pfx`  
设置密码 Pass-1234     ``#后面要用``  
`openssl  pkcs12 -nodes  -in skills.pfx -out skills.pem`  
无需密码  
`openssl rsa -in skills.pem -out apache.key`  
`openssl x509 -in skills.pem -out apache.crt`

### 
## （3）客户端访问Apache服务时，必需有ssl证书。
`curl --cert /etc/ssl/apache.crt --key /etc/ssl/apache.key https://web.skills.lan`  
`curl --cert /etc/ssl/apache.crt --key /etc/ssl/apache.key http://web.skills.lan -L`



## （2）把/etc/ssl/skills.crt证书文件和/etc/ssl/skills.key私钥文件转换成含有证书和私钥的/etc/ssl/skills.pfx文件；然后把/etc/ssl/skills.pfx转换为含有证书和私钥的/etc/ssl/skills.pem文件，再从/etc/ssl/skills.pem文件中提取证书和私钥分别到/etc/ssl/apache.crt和/etc/ssl/apache.key。
cd  /etc/ssl/  
``openssl pkcs12 -export -in skills.crt -inkey skills.key -out skills.pfx``  
设置密码 Key-1122     ``#后面要用``  
``openssl  pkcs12 -nodes  -in skills.pfx -out skills.pem``  
  
``openssl rsa -in skills.pem -out apache.key``  
``openssl x509 -in skills.pem -out apache.crt``

### 还需信任ca机构
`cp /etc/ssl/skills.pem /etc/pki/ca-trust/source/anchors/`  
`update-ca-trust`

## （3）客户端访问Apache服务时，必需有ssl证书。
`Curl -E /etc/ssl/skills.pem https://web.skills.lan`  
`Curl -E /etc/ssl/skills.pem http://web.skills.lan -L`

## 5.tomcat服务  
任务描述：采用Tomcat搭建动态网站。  
（1）配置linux2为nginx服务器，默认文档index.html的内容为“hellonginx”；仅允许使用域名访问，http访问自动跳转到https。
## ``--80/tcp,443/tcp``
`dnf install nginx* -y`

`vi /etc/nginx/nginx.conf`

```plain
 38     server {
 39         listen       80;
 40      #   listen       [::]:80;
 41         server_name  linux2.skills.lan;
 42         return 301 https://linux2.skills.lan;
 43         root         /usr/share/nginx/html;
 44 
 45         # Load configuration files for the default server block.
 46     #    include /etc/nginx/default.d/*.conf;
 47 
 48    #     error_page 404 /404.html;
 49   #      location = /404.html {
 50  #       }
 51 #
 52         error_page 500 502 503 504 /50x.html;
 53         location = /50x.html {
 54         }
 55     }
 56 
    server {														#添加两个拒绝未定义的server_name进行连接
    listen 80 default_server;
    return 403;
}
        server{
        listen 443 http2 ssl default_server;
        ssl_certificate "/etc/ssl/skills.crt";
        ssl_certificate_key "/etc/ssl/skills.key";
        return 403;
        }
 57 # Settings for a TLS enabled server.
 58 #
 59     server {
 60         listen       443 ssl http2;
 61 #        listen       [::]:443 ssl http2;
 62         server_name  linux2.skills.lan;
 63 #        root         /usr/share/nginx/html;
 64 #
 65         ssl_certificate "/etc/ssl/skills.crt";
 66         ssl_certificate_key "/etc/ssl/skills.key";
 67 #        ssl_session_cache shared:SSL:1m;
 68 #        ssl_session_timeout  10m;
```

## （2）利用nginx反向代理，实现linux3和linux4的tomcat负载均衡，通过[https://tomcat.skills.lan加密访问Tomcat，http访问通过301自动跳转到https。](https://tomcat.skills.lan加密访问Tomcat，http访问通过301自动跳转到https。)
`vi /etc/nginx/conf.d/1.conf`

```plain
upstream tomcat{
        server linux3.skills.lan:443;
        server linux4.skills.lan:443;
}
server{
        listen 80;
        server_name tomcat.skills.lan;
        return 301 https://tomcat.skills.lan;
}
server {
        listen 443 ssl;
        server_name tomcat.skills.lan;
        ssl_certificate "/etc/ssl/skills.crt";
        ssl_certificate_key "/etc/ssl/skills.key";
        location / {
        proxy_pass https://tomcat;
        }
}
```

`setsebool -P httpd_can_network_connect on`

``systemctl restart nginx && systemctl enable nginx``

---

## （3）配置linux3和linux4为tomcat服务器，网站默认首页内容分别为“tomcatA”和“tomcatB”，仅使用域名访问80端口http和443端口https；证书路径均为/etc/ssl/skills.jks。
`keytool -importkeystore -srckeystore /etc/pki/tls/skills.pfx -destkeystore /etc/pki/tls/skills.jks -deststoretype JKS`

密码:Pass-1234

再次输入:Pass-1234

输入源密码:Pass-1234  
<font style="background-color:#FBF5CB;">`keytool -importkeystore (密钥库) -srckeystore 源密钥的路径-destkeystore 导出的密钥路径 -deststoretype 指定的类型`</font>

`<font style="background-color:rgba(255, 255, 255, 0);">dnf install tomcat* -y</font>`

`vi /usr/lib/systemd/system/tomcat.service `

```plain
[Service]
Type=simple
EnvironmentFile=/etc/tomcat/tomcat.conf
Environment="NAME="
EnvironmentFile=-/etc/sysconfig/tomcat
ExecStart=/usr/libexec/tomcat/server start
SuccessExitStatus=143
User=root									#改这里
```

<!-- 这是一张图片，ocr 内容为： -->
![](:storage\f7wg4917w6k73nmi.png)`vi /etc/tomcat/server.xml`

```plain
 69     <Connector port="80" protocol="HTTP/1.1"
 70                connectionTimeout="20000"
 71                redirectPort="443" />
 
 86     <Connector port="443" protocol="org.apache.coyote.http11.Http11NioProt    ocol"
 87                maxThreads="150" SSLEnabled="true">
 88         <SSLHostConfig>
 89                 <Certificate certificateKeystoreFile="/etc/ssl/skills.jks"
 90                              certificateKeystorePassword="Pass-1234"
 91                          type="RSA" />
 92         </SSLHostConfig>

130    <Engine name="Catalina" defaultHost="linux3.skills.lan">

150      <Host name="linux3.skills.lan"  appBase="webapps"
151            unpackWARs="true" autoDeploy="true">
152      </Host>

167      <Host name="192.168.31.213"  appBase="ipapps"
168            unpackWARs="false" autoDeploy="false">
169      </Host>
170    </Engine>
```

  
`#改完直接scp /etc/tomcat/server.xml到linux4,再改对应主机名和IP`  
`#ss -tunlup查看端口是否开启`  
`linux3/4:`  
`systemctl restart tomcat && systemctl enable tomcat`

---

## 6.samba服务  
任务描述：请采用samba服务，实现资源共享。  
（1）在linux3上创建user00-user19等20个用户；user00和user01添加到manager组，user02和user03添加到dev组。把用户user00-user03添加到samba用户。
``--445/tcp,139/tcp``  
``dnf install samba* -y``

`vi user.sh`

```plain
for i in {00..19}
do
useradd user$i
echo "user$i" | passwd --stdin user$i
done
groupadd manager
groupadd dev
usermod -aG manager user00 
usermod -aG manager user01 
usermod -aG dev user02 
usermod -aG dev user03 
```

`sh user.sh`

`smbpasswd -a user00到user03`    #全部设置一遍密码

---

##   
（2）配置linux3为samba服务器,建立共享目录/srv/sharesmb，共享名与目录名相同。manager组用户对sharesmb共享有读写权限，   dev组对sharesmb共享有只读权限；用户对自己新建的文件有完全权限，对其他用户的文件只有读权限，且不能删除别人的文件。在本机用smbclient命令测试。
!!!

#### `<font style="color:#DF2A3F;">setsebool -P samba_enable_home_dirs on</font>`
#### `<font style="color:#DF2A3F;">setsebool -P samba_export_all_rw on</font>`<font style="color:#DF2A3F;"> 	</font>
!!!  
`mkdir /srv/sharesmb`  
`chmod 777 /srv/sharesmb`			#设置目录的u g o 均为777  
`smbpasswd -a user00 – user03`	#新建user00到user03  
`chmod o+t  /srv/sharesmb`		#再给o一个粘滞位

`vi /etc/samba/smb.conf`



```plain
[sharesmb]
        comment = sharesmb
        path = /srv/sharesmb
        valid users = @manager,@dev
        write list = @manager
        read list = @dev,@manager
        create mask = 1744
```

smbclient //10.93.123.33/sharesmb -U user00

#随便put一个文件上去测试写入  

put .bash_history 			--失败

NT_STATUS_ACCESS_DENIED opening remote file \.bash_history

                                 

put .cshrc 				--成功

putting file .cshrc as \.cshrc (48.8 kb/s) (average 48.8 kb/s)

exit

#测试粘滞位有没有作用,登录别的用户删除刚put的文件

smbclient //10.93.123.33/sharesmb -U user01

smb: \> rm .lesshst 		--起作用了

NT_STATUS_ACCESS_DENIED deleting remote file \.lesshst



  
<!-- 这是一张图片，ocr 内容为： -->
![](:storage\xc4u7inpfga54s4i.png)

---

## （3）在linux4修改/etc/fstab,使用用户user00实现自动挂载linux3的sharesmb共享到/sharesmb。
``dnf install samba-cl* cifs-utils-7.0-1.el9.x86_64 -y``  
``vi /etc/fstab``

```plain
//192.168.31.233/sharesmb /sharesmb cifs username=user00 ,password=Key-1122	0 0
```

``systemctl daemon-reload``  
``mount -a``

---

## 7.nfs服务  
任务描述：请采用nfs，实现共享资源的安全访问。  
（1）配置linux2为kdc服务器，负责linux3和linux4的验证。
``--{88,464,749}/tcp/udp``

<u>linux2:</u>

`dnf install krb5-* -y`		  
``vi /etc/krb5.conf``

```plain
    default_realm = SKILLS.LAN
    default_ccache_name = KEYRING:persistent:%{uid}

[realms]
 SKILLS.LAN = {
     kdc = linux2.skills.lan
     admin_server = linux2.skills.lan
 }

[domain_realm]
 .skills.lan = SKILLS.LAN
 skills.lan = SKILLS.LAN
```

`#删掉注释,把配置文件都改成自己的域,中间还有一段的，找仔细点,该大写的大写`  
``scp /etc/krb5.conf linux3:/etc/``

``scp /etc/krb5.conf linux4:/etc/``

``vi /var/kerberos/krb5kdc/kadm5.acl``

```plain
*/admin@SKILLS.LAN      *
```

`vi /var/kerberos/krb5kdc/kdc.conf`

```plain
[realms]
SKILLS.LAN = {
     master_key_type = aes256-cts-hmac-sha384-192
     acl_file = /var/kerberos/krb5kdc/kadm5.acl
```

``kdb5_util create -s`` #创建数据库并设置密码Key-1122

#开机自启服务  
`systemctl enable krb5kdc kadmin --now`  
``kadmin.local``        #管理数据库  
kadmin.local:``addprinc root/admin `` #输入密码 Key-1122  
kadmin.local: ``addprinc -randkey	nfs/linux3.skills.lan`` #-randkey 随机创建密码  
kadmin.local: ``addprinc -randkey	nfs/linux4.skills.lan``  
kadmin.local: ``listprincs`` #列出创建的数据

kadmin.local:  `exit`

---

## （2）在linux3上，创建用户，用户名为xiao，uid=222，gid=222，家目录为/home/xiaodir。


  
`groupadd -g 222 xiao`  
``useradd -u 222 -g 222 -d /home/xiaodir xiao``

---

## （3）配置linux3为nfs服务器，目录/srv/sharenfs的共享要求为：linux服务器所在网络用户有读写权限，所有用户映射为xiao，kdc加密方式为krb5p。
``--{111,20048,2049}/tcp/udp``  
    linux3:  
``dnf install krb5-work* nfs-ut* -y``  
``kadmin``  
``ktadd nfs/linux3.skills.lan@SKILLS.LAN``

`exit`

`mkdir /srv/sharenfs`

`chmod 777 /srv/sharenfs`

`vi /etc/exports`

```plain
/srv/sharenfs   192.168.31.0/24(rw,anonuid=222,anongid=222,sec=krb5p)
```

`exportfs -arv`

## （4）配置linux4为nfs客户端，利用autofs按需挂载linux3上的/srv/sharenfs到/sharenfs目录，挂载成功后在该目录创建test目录。
``--{111,20048,2049}/tcp/udp``

`dnf install krb5-wo* autofs nfsv4-cl* <font style="background-color:rgba(255, 255, 255, 0.05);">nfs-utils</font> -y`

`kinit -kt /etc/krb5.keytab nfs/linux4.skills.lan@SKILLS.LAN`

[root@linux4 sharenfs]# `kvno nfs/linux3.skills.lan@SKILLS.LAN` #测试是否打通<font style="background-color:rgba(255, 255, 255, 0);">Kerberos认证链路</font>

nfs/linux3.skills.lan@SKILLS.LAN: kvno = 2

`systemctl enable autofs <font style="background-color:rgba(255, 255, 255, 0.05);">rpc-gssd</font> --now`!!!

`vi /etc/auto.master`

#添加

```plain
/-		/etc/auto.nfs
```

`vi /etc/auto.nfs`

```plain
/sharenfs 	-fstype=nfs,rw,sec=krb5p   linux3.skills.lan:/srv/sharenfs		#顺序不要错
```

`cd /sharenfs`

`df`

mkdir test

---

## 8.ftp服务
## 任务描述：请采用FTP服务器，实现文件安全传输。
## --21/tcp --20/tcp
## （1）配置linux2为FTP服务器，安装vsftpd，新建本地用户test，本地用户登陆ftp后的目录为/var/ftp/pub，可以上传下载。
[root@linux2 ~]# `yum install -y vsftpd libdb-utils ftp` 

创建用户

[root@linux2 ~]# `useradd test`  
[root@linux2 ~]# `passwd test`  
Changing password for user test.  
New password:   
BAD PASSWORD: The password is shorter than 8 characters  
Retype new password:   
passwd: all authentication tokens updated successfully.  
[root@linux2 ~]# `chmod 777 /var/ftp/pub/ -R `

编辑配置文件

[root@linux2 ~]#` vim /etc/vsftpd/vsftpd.conf` 

追加以下内容

`local_root=/var/ftp/pub`   	# 指定用户本地文件系统上的根目录  
`allow_writeable_chroot=YES`	# 所有访问将被限制在该目录下，用 户将无法访问除该目录之外的任何内容。

## （2）配置ftp虚拟用户认证模式，虚拟用户ftp1和ftp2映射为ftp，ftp1登录ftp后的目录为/var/ftp/vdir/ftp1，可以上传下载,禁止上传后缀名为.docx的文件；ftp2登录ftp后的目录为/var/ftp/vdir/ftp2，仅有下载权限。
创建虚拟用户数据库文件

[root@linux2 ~]# cd /etc/vsftpd/

[root@linux2 vsftpd]# `vim vuser`

```plain
ftp1
123
ftp2
123
```

[root@linux2 vsftpd]# `db_load -T -t hash -f vuser vuser.db`  
[root@linux2 vsftpd]# `chmod 600  vuser.db` 

建立支持虚拟用户的 PAM 认证文件

[root@linux2 ~]# `vim /etc/pam.d/vsftpd` 

全都注释，只留这两行 ；sufficient 表示充分条件，可以让 vsftpd 同时支持虚拟用户和本地用户

```plain
auth sufficient pam_userdb.so db=/etc/vsftpd/vuser
account sufficient pam_userdb.so db=/etc/vsftpd/vuser
```

编辑配置文件

[root@linux2 ~]# `vim /etc/vsftpd/vsftpd.conf` 

```plain
47 anon_upload_enable=YES	       # 解除注释
32 anon_mkdir_write_enable=YES    # 解除注释
# 添加一下内容
user_config_dir=/etc/vsftpd/vuser_profile
file_open_mode=0755
```

修改虚拟目录权限

[root@linux2 ~]# `mkdir /etc/vsftpd/vuser_profile`  
[root@linux2 ~]# `vim /etc/vsftpd/vuser_profile/ftp1`

```plain
guest_enable=YES
guest_username=ftp
anon_world_readable_only=NO
anon_upload_enable=YES
anon_mkdir_write_enable=YES
anon_other_write_enable=YES
deny_file={*.docx}
local_root=/var/ftp/vdir/ftp1
```

[root@linux2 ~]# `vim /etc/vsftpd/vuser_profile/ftp2`

```plain
guest_enable=YES
guest_username=ftp
anon_world_readable_only=NO
anon_upload_enable=NO
anon_mkdir_write_enable=NO
anon_other_write_enable=NO
local_root=/var/ftp/vdir/ftp2
```

创建根目录及虚拟用户映射的系统用户

[root@linux2 ~]# `mkdir /var/ftp/vdir`  
[root@linux2 ~]# `usermod -d /var/ftp/vdir ftp`

创建虚拟 ftp 用户

[root@linux2 ~]# `mkdir /var/ftp/vdir/{ftp1,ftp2}`  
[root@linux2 ~]# `chown ftp:ftp /var/ftp/vdir/ -R`  
[root@linux2 ~]# `systemctl restart vsftpd`

## （3）使用ftp命令在本机验证。
`cd  /var/ftp/vdir/ftp1`

`touch 1.txt`

`cd  /var/ftp/vdir/ftp2`

`touch 2.txt`

`ftp 127.0.0.1`

Connected to 127.0.0.1 (127.0.0.1).

220 (vsFTPd 3.0.5)

`Name (127.0.0.1:root): ftp1`

331 Please specify the password.

`Password:`

230 Login successful.

Remote system type is UNIX.

Using binary mode to transfer files.

`ftp> ls`

227 Entering Passive Mode (127,0,0,1,148,75).

150 Here comes the directory listing.

-rw-r--r--    1 0        0               0 Mar 05 09:58 `1.txt`

drwx------    2 14    50            6 Mar 05 10:03 新文件夹

226 Directory send OK.



### 9.iscsi服务
### 任务描述：请采用iscsi，搭建存储服务。
### （1）为linux8添加4块硬盘，每块硬盘大小为5G，创建lvm卷，卷组名为vg1，逻辑卷名为lv1，容量为全部空间，格式化为ext4格式。使用/dev/vg1/lv1配置为iSCSI目标服务器，为linux9提供iSCSI服务。iSCSI目标端的wwn为iqn.2023-08.lan.skills:server, iSCSI发起端的wwn为iqn.2023-08.lan.skills:client。
### （2）配置linux9为iSCSI客户端，实现discovery chap和session chap双向认证，Target认证用户名为IncomingUser，密码为IncomingPass；Initiator认证用户名为OutgoingUser，密码为OutgoingPass。修改/etc/rc.d/rc.local文件开机自动挂载iscsi硬盘到/iscsi目录。
### 10.mysql服务
### -- 3360/tcp
## 任务描述：请安装mysql服务，建立数据表。
## （1）配置linux2为mysql服务器，创建数据库用户xiao，在任意机器上对所有数据库有完全权限。
`systemctl enable mysqld --now`

`mysql`

mysql>`create user xiao identified by 'Key-1122';`

mysql>`grant all on *_.* _to xiao;`

#查看是否创建成功  select user,host from mysql.user;

#删除用户  drop user 'username';

## （2）创建数据库userdb；在库中创建表userinfo，表结构   b 如下：
| ##### 字段名 | ##### 数据类型 | ##### 主键 | ##### 自增 |
| --- | --- | --- | --- |
| ##### id | ##### int | ##### 是 | ##### 是 |
| ##### name | ##### varchar(10) | ##### 否 | ##### 否 |
| ##### birthday | ##### datetime | ##### 否 | ##### 否 |
| ##### sex | ##### varchar(5) | ##### 否 | ##### 否 |
| ##### password | ##### varchar(200) | ##### 否 | ##### 否 |


mysql>`create database userdb;`

#查看创建了的数据库 show databases;

mysql>`use userdb;`

mysql>`create table userinfo(id int primary key auto_increment,name varchar(10),birthday datetime,sex varchar(5),password varchar(200));`

#查看表结构，desc userinfo;

## （3）在表中插入2条记录，分别为(1,user1，1999-07-01，男)，(2,user2，1999-07-02，女)，password与name相同，password字段用password函数加密。
mysql>`insert into userinfo values('1','user1','1999-07-01','男',MD5('user1')),('2','user2','1999-07-02','女',MD5('user2'));`

#查看表记录 select * from userinfo;

## （4）修改表userinfo的结构，在name字段后添加新字段height(数据类型为float)，更新user1和user2的height字段内容为1.61和1.62。
`mysql> alter table userinfo add column height float after name;`

`update userinfo set height = 1.61 where name = 'user1';`

`update userinfo set height = 1.62 where name = 'user2';`

#验证表结构 desc userinfo;

#验证表记录 select * from userinfo;

## （5）新建/var/mysqlbak/userinfo.txt文件，文件内容如下，然后将文件内容导入到userinfo表中，password字段用password函数加密。
##### 3,user3,1.63,1999-07-03,女,user3
##### 4,user4,1.64,1999-07-04,男,user4
##### 5,user5,1.65,1999-07-05,男,user5
##### 6,user6,1.66,1999-07-06,女,user6
##### 7,user7,1.67,1999-07-07,女,user7
##### 8,user8,1.68,1999-07-08,男,user8
##### 9,user9,1.69,1999-07-09,女,user9
#报错  ERROR 3948 (42000): Loading local data is disabled; this must be enabled on both the client and server sides

  `vi /etc/my.cnf`#添加字条

```plain
[mysqld]
local-infile=1					#允许本地文件导入
secure-file-priv="/var/mysqld/"				#允许导出到路径
```

`systemctl restart mysqld`

MariaDB [(userdb)]> `load data local infile '/var/mysqlbak/userinfo.txt' into table userinfo fields terminated by ',' lines terminated by '\n' (id,name,height,birthday,sex,@password)set password=MD5(@Password);`

## （6）将表userinfo的记录导出，存放到/var/databak/mysql.sql，字段之间用','分隔。
`systemctl restart mysqld`

`setenforce 0`

mysql>`select * from userinfo into outfile '/var/databak/mysql.sql' fields terminated by ',' lines terminated by '\n';`



## （7）每周五凌晨1:00以root用户身份备份数据库userdb到/var/databak/userdb.sql(含创建数据库命令)。
`cronrab -e`

```plain
01 ** 5 mysqldump -u root -pKey-1122 --databases userdb > /var/mariadb/userdb.sql
```

#这里要含创建数据的的命令是要加上 --databases的

#验证是否有创建数据库的命令

`cat /var/mysql/userdb.sql |grep "CREATE"`

## 11.mariadb  
(1).配置rocky5为mariadb服务器，创建数据库用户teacher，在任意机器上对所有数据库有完全控制权限，允许从任意地方登录数据库。
--3306/tcp

`systemctl restart mariadb`

`systemctl enable mariadb --now`

`mysql -u root -p`

Enter password: 

MariaDB [(none)]> `create user teacher identified by 'Key-1122';`

MariaDB [(none)]> `grant all on *_.* _to teacher;`

#查看是否创建成功  select user,host from mysql.user;

#删除用户  drop user 'teacher';

---

##   
(2).创建数据库studentdb;在库中创建表studentinfo，表结构如下  
| ##### 字段名 | ##### 数据类型 | ##### 主键 | ##### 自增 | ##### 约束 |
| --- | --- | --- | --- | --- |
| ##### sid | ##### int | ##### 是 | ##### 是 | ##### 非空 |
| ##### sname | ##### varchar(10) | ##### 否 | ##### 否 | ##### 非空 |
| ##### sheight | ##### float | ##### 否 | ##### 否 | #####  |
| ##### sbirthday | ##### datetime | ##### 否 | ##### 否 | #####  |
| ##### ssex | ##### varchar(5) | ##### 否 | ##### 否 | #####  |
| ##### password | ##### varchar(200) | ##### 否 | ##### 否 | #####  |


MariaDB [(none)]>`create database studentdb;`	<font style="color:#DF2A3F;">##看存储数据库的数据是否有中文!!!,假如有中文</font>

#设置可以存储中文数据库的类型	<font style="color:#DF2A3F;background-color:#FBDE28;">create database studentdb character set utf8mb4; </font><font style="color:#DF2A3F;">    #这个也要记下来</font>

#查看创建了的数据库 show databases;

MariaDB [(none)]>`use studentdb;`

MariaDB [(userdb)]>`create table studentinfo(sid int primary key auto_increment not null,sname varchar(10) not null,sheight float,sbirthday datetime,ssex varchar(5),password varchar(200));`

#查看表结构，desc studentinfo;	

---

## (3).在表中插入2条记录，分别为(1,xmy,1.65,2001-02-06，M)，(2,zhp,1.71.2001-10-21,M)，password字段与name字段相同，password字段用md5函数加密。新建/var/mariadb/studentinfo.txt文件，文件内容如下，然后将文件内容导入到studentinfo表中，password字段用md5函数加密。
##### 3,yxt,1.63,2004-05-05,F,yxt
##### 4,yzm,1.64,2000-07-04,F,yzm
##### 5,xxh,1.68,2003-09-21,M,xxh
##### 6,zqf,1.86,2001-07-06,M,zqf
##### 7,cmh,1.67,2009-01-18,M,cmh
##### 8,xzh,1.78,2004-05-18,M,xzh
##### 9,xmm,1.69,2000-07-09,F,xmm
MariaDB [(userdb)]>`insert into studentinfo values ('1','xmy','1.65','2001-02-06'，'M',MD5('xmy')),('2','zhp','1.71','2001-10-21','M',MD5('zhp'));`

#查看表记录 select * from studentinfo;

MariaDB [(userdb)]> `exit`

`mkdir /var/mariadb`

`chown mysql:mysql /var/mariadb`

`cd /var/mariadb`

`vi studentinfo.txt`

```plain
3,yxt,1.63,2004-05-05,F,yxt
4,yzm,1.64,2000-07-04,F,yzm
5,xxh,1.68,2003-09-21,M,xxh
6,zqf,1.86,2001-07-06,M,zqf
7,cmh,1.67,2009-01-18,M,cmh
8,xzh,1.78,2004-05-18,M,xzh
9,xmm,1.69,2000-07-09,F,xmm
```

#报错  ERROR 3948 (42000): Loading local data is disabled; this must be enabled on both the client and server sides

  `SET GLOBAL local_infile = ON;`

`setenforce 0`		#先关掉sel策略导出完再开

`mysql`

MariaDB [(none)]> `use userdb;`

MariaDB [(userdb)]> `load data local infile '/var/mariadb/studeninfo.txt' into table studentinfo fields terminated by ',' lines terminated by '\n' (sid,sname,sheight,sbirthday,ssex,@password)set password=MD5(@Password);`



##### 
---

## (4).将表studentinfo中的记录导出，并存放到/var/mariadb/sinfo.sql，字段之间用',分隔。利用cron为root用户创建计划任务(day用数字表示)，每周六凌晨1:00备份数据库studentdb(不含创建数据库命令)到/var/mariadb/sdb_bak.sql。(为便于测试，手动备份一次。)


MariaDB [userdb]> `select * from studentinfo into outfile'/var/mariadb/userinfo.sql' <font style="background-color:rgba(255, 255, 255, 0.05);">fields terminated by ',' lines terminated by '\n'</font>;`

`setenforce 0`

cronrab -e

```plain
0 1 * * 6 mysqldump -u teacher -pKey-1122 studentdb > /var/mariadb/sdb_bak.sql
```

  
<font style="color:#DF2A3F;">##这里不含create database的是这个,含create database的话</font>

mysqldump -u teacher -pKey-1122 --databases studentdb > /var/mariadb/sdb_bak.sql

---

## 12.shell脚本
## 任务描述：请采用shell脚本,实现快速批量的操作。
## （1）在linux4上编写/root/createfile.sh的shell脚本，创建20个文件/root/shell/file00至/root/shell/file19，如果文件存在，则删除再创建；每个文件的内容同文件名，如file00文件的内容为“file00”。用/root/createfile.sh命令测试。
