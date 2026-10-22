## 1.基本配置
## 本任务需要使用root用户完成相关配置，已提供操作系统镜像及需要配置前置环境，所需软件安装包均在/root/目录下。命令中要求使用绝对路径，具体要求如下:
## 1. 在AI-SRV中将操作系统镜像挂载至/media/cdrom，并实现开机自启。
启动openEuler系统

`mkdir /media/cdrom/`

`echo "openEuler-22.03-LTS-everything-x86_64-dvd.iso  /media/cdrom  iso9660 defaults 0 0"  >> /etc/fstab`

`mount -a`

## 2. 在AI-SRV中完成软件源配置文件的编写，禁用GPG校验，并为源数据建立缓存。 
`vi /etc/yum.repos.d/1.repo`

```plain
[1]
name=1
baseurl=file:///media/cdrom
gpgcheck=0
enable=1
```

`dnf install bash* -y`

`bash`

#用补全

`yum makecache`

`dnf install docker* vim -y`

## 2.环境搭建
## 本任务需要使用root用户完成相关配置，已提供ollama安装包及需要配置前置环境。命令中要求使用绝对路径，具体要求如下:
## 1. 在AI-SRV中为ollama配置用户/组，该用户仅用于运行服务，禁止其登录系统，家目录位于/usrlshare/ollama。 
`useradd -s /bin/false -d /usrlshare/ollama ollama`

## 2. 在AI-SRV中为root用户加入附加组ollama。
`usermod -aG ollama root`

## 3. 在AI-SRV中将文件ollama-linux-amd64.tgz解压安装至/usr目录下。
`tar zxvf ollama-linux-amd64.tgz -C /usr/`

## 4. 在AI-SRV中编写SystemD服务配置，使用服务运行专用用户完成启动，无论发生任何意外情况，服务都会进行自动重启（冷却3秒)，以保证服务可靠运行，服务需要监听所有地址。
`vi /etc/systemd/system/ollama.service`

```plain
[Unit]
Description=Ollama Service
After=network-online.target

[Service]
ExecStart=/usr/bin/ollama serve
User=ollama
Group=ollama
Restart=always
RestartSec=3
Environment="OLLAMA_HOST=0.0.0.0:11434"

[Install]
WantedBy=default.target
```

## 5. 在AI-SRV中启动ollama服务，并完成开机启动配置，最终启动ollama服务。
`systemctl start ollama`

`systemctl enable ollama`

## 6. 在AI-SRV中导入1.5b.tgz压缩文件模型至ollama。
`tar zxvf manifests.tar.gz -C /usrlshare/ollama/.ollama/models/`

`tar zxvf ~/deepseek-r1.tar.gz -C /usrlshare/ollama/.ollama/models/blobs/`

ollama run deepseek-r1:1.5b

## 3.Dify运行环境搭建
## 本任务需要使用root用户完成相关配置，已提供安装dify及需要配置前置环境。命令中要求使用绝对路径，具体要求如下:
## 1．在AI-SRV中完成Docker环境的安装，并查看Docker版本信息。
docker version

## 2. 在AlI-SRV中将文件 dify-0.14.2.tar.gz解压至/opt路径中。
`tar zxvf dify-0.14.2.tar.gz -C /opt/`

## 3. 在AI-SRV中完成dify镜像的导入，相关文件位于dify-images目录中，完成后检查镜像列表。 
`tar zxvf dify-image.tar.gz`

`docker load < dify-image/dify-api.tar`

`docker load < dify-image/dify-web.tar`

`docker load < dify-image/squid.tar`

`docker load < dify-image/weaviate.tar`

`docker load < dify-image/nginx.tar`

`docker load < dify-image/dify-sandbox.tar`

`docker load < dify-image/postgres.tar`

`docker load < dify-image/redis.tar`

docker image list

## 4. 在AI-SRV中完成dify 的启动，并在客户机中访问AI-SRV中的dify web服务。
`cd /opt/`

`vi dify-0.14.2/docker-legacy/docker-compose.yaml`

#寻找

/JOB

/0.2.1

```plain
 387       CREATE_TIDB_SERVICE_JOB_ENABLED: "false"      #双引号注释掉
 484     image: langgenius/dify-sandbox:0.2.10          #加个0
```



`docker-compose up -d`

#如果遇到iptables failed 

docker-compose down

iptables -F

systemctl restart docker

docker-compose up -d

## 5. 在客户机中访问dify，完成用户注册，邮箱为admin@dify.ai，用户名为admin，密码为Qwer1234，注册后登录dify管理页面。 
`firewall-cmd --add-port=80/tcp --permanent`

`firewall-cmd --reload`

太简单，省略

## 6. 在客户机中访问dify，完成对ollama模型提供商的注册。
![](https://cdn.nlark.com/yuque/0/2025/png/53941428/1741318690397-0d4dfc9f-a148-4256-9368-faf94441502d.png)![](https://cdn.nlark.com/yuque/0/2025/png/53941428/1741318730600-958ae127-7db2-4cc2-9ca4-16295d2f54a0.png)

![](https://cdn.nlark.com/yuque/0/2025/png/53941428/1741318786355-767a604e-dfba-4342-a4e3-f91cc12023d2.png)





## 4.语言翻译器构建
## <font style="color:black;">本任务需要基于前置环境完成，选手</font><font style="color:black;">须先自行完成配置，具体要求如下：</font>
## <font style="color:black;">1. </font><font style="color:black;">根据场景需要，创建空白应用</font><font style="color:black;"> “Translator”</font><font style="color:black;">。</font>
## <font style="color:black;">2. </font><font style="color:black;">为用</font><font style="color:black;">户提供翻译功能，其中参数包含</font><font style="color:black;"> Query, Language </font><font style="color:black;">两项，要求支持</font><font style="color:black;">语言（中文、</font><font style="color:black;">English</font><font style="color:black;">），提示词自拟</font><font style="color:black;">。</font>
## <font style="color:black;">3. </font><font style="color:black;">为确保结果准确性，需要为</font><font style="color:black;"> Web App </font><font style="color:black;">提供翻译结果多版本对照支</font><font style="color:black;">持。</font>
## <font style="color:black;">4. 运行应用，翻译内容：“人工智能改变世界”，“AI has changed the world now.”，并为翻译内容创建候选。</font>


[root@localhost ~]# `tar xvf cpython-3.11.6+20231002-x86_64_v2-unknown-linux-gnu-install_only.tar.gz -C /opt/`

[root@localhost vllm-offline]# `ln -s /opt/python/bin/python3.11 /usr/local/bin/python3.11`

[root@localhost vllm-offline]# `ln -s /opt/python/bin/pip /usr/local/bin/pip`

[root@localhost ~]# `tar xzvf vllm-offline.tar.gz`

[root@localhost vllm-offline]# `/opt/python/bin/pip install --find-links wheels/ -r requirements.txt  --no-index`

[root@localhost vllm-offline]#  `python3.11 -c "import vllm; print(vllm.__version__)"`

0.14.1

