# jump  
## [26vllm样题](#26vllm样题)

------

# 26省赛ollama样题

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

\#用补全

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

\#寻找

/JOB

/0.2.1

```plain
 387       CREATE_TIDB_SERVICE_JOB_ENABLED: "false"      #双引号注释掉
 484     image: langgenius/dify-sandbox:0.2.10          #加个0
```



`docker-compose up -d`

\#如果遇到iptables failed 

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
## 本任务需要基于前置环境完成，选手须先自行完成配置，具体要求如下：
## 1. 根据场景需要，创建空白应用 “Translator”。
## 2. 为用户提供翻译功能，其中参数包含 Query, Language 两项，要求支持语言（中文、English），提示词自拟。
## 3. 为确保结果准确性，需要为 Web App 提供翻译结果多版本对照支持。
## 4. 运行应用，翻译内容：“人工智能改变世界”，“AI has changed the world now.”，并为翻译内容创建候选。

----

# 26vllm样题  

[⬆️top⬆️](#jump)

# (一）基础环境配置   
# 本任务需要使用 root 用户完成相关配置，已提供操作系统镜像及需要配置前置环境，所需软件安装包均在/root/目录下。命令中要求使用绝对路径，具体要求如下:   

## 1.在 AI-SRV 中将操作系统镜像挂载至/media/cdrom，并实现开机自启。  

## 2.在 AI-SRV 中完成软件源配置文件的编写，禁用GPG 校验，并为源数据建立缓存。   


# （二）环境搭建 
# 本任务需要使用 root 用户完成相关配置，已提供transformers安装包及需要配置前置环境。 命令中要求使用绝对路径，具体要求如下:  

## 1.在 AI-SRV 中解压对应的 Python pip 包，安装gcc、gcc-c++。使用 python 在/root 目录中创建虚拟环境venv-tokenizers。

## 2.在 AI-SRV 中解压 test_tokenizers.tar.gz 至/opt 目录，编写 如下 test_tokenizer.py 脚本测试分词器是否正常运行：
```py

from transformers import AutoTokenizer 
tokenizer
AutoTokenizer.from_pretrained("/opt/test_tokenizers")
instruct_str = "欢迎来到 2025-2026 年度网络建设与运维赛项！"
ids = tokenizer.encode(instruct_str)
text = tokenizer.decode(ids)
print(f"字符串对应的 ID: {ids}")
print(f"Token ID 对应的字符串: {text}")

```
# （三）Dify运行环境搭建 
# 本任务需要使用 root 用户完成相关配置，已提供安装dify及需要配置前置环境。命令中要求使用绝对路径，具体要求如下:  
## 1．在 AI-SRV 中完成 Docker 环境的安装，并查看Docker 版本信息。 

## 2.在AlI-SRV中将文件 dify-v1.11.2.tar.gz解压至/opt路径中。  

## 3.在 AI-SRV 中完成 dify 镜像的导入，相关文件位于dify-images目录中，完成后检查镜像列表。   

## 4.在 AI-SRV 中完成 dify 的启动，并在客户机中访问AI-SRV中的 dify web 服务。   

## 5. 在 客 户 机 中 访 问 dify ， 完成用户注册，邮箱为admin@gdskills.ai，用户名为 admin，密码为GDSkills2026，注册后登录 dify 管理页面。   

## 6.在客户机中访问 dify，通过 dify 扩展包安装vLLM-openai扩展组件，完成对 vLLM 模型提供商的注册。  

# （四）大模型部署与功能验证 本任务需要使用 root 用户完成相关配置，已提供安装vllm-cpu及需要配置前置环境。命令中要求使用绝对路径，具体要求如下:

## 1.在 AI-SRV 中解压对应的 Python pip 包，安装gcc、gcc-c++。使用 python 创建虚拟环境 venv-vllm-cpu。  

## 2.在/root 目录中存放一个 Yuezhi100M.tar.gz，该目录中提供了一个模型权重文件，解压模型文件至/opt 目录并复制任务三训练好的分词器存放在模型目录中。 

## 3.使用 vLLM 部署 Yuezhi-100M 大模型，设置vLLM_CPU_KVCACHE为 4G，防止过多的 kv_cache 占用导致内存不足；设置服务模型名称为Yuezhi，apiKey 为 GDSkills2026!@#$，监听所有接口，API 服务监听端口为 16384，张量并行为 1，dtype 为 float32，最大模型序列长度为2048,最大请求序列为 8。 

## 4.在 dify 中创建创建空白应用 YuezhiChat。为改善对话体验，须为助手配置初始问候语，内容为： “你好，我是智能助手，有什么能够帮助你？”，并添加以下常用问题 “现在几点了？”、“查询当前时间”。

## 5.使用客服助手生成一个 shell 脚本，实现批量创建20 个文件file00 至 file19;每个文件的内容同文件名，如file00 文件的内容为“file00”。将 shell 脚本保存命名为 createfile.sh。  

[root@localhost ~]# `tar xvf cpython-3.11.6+20231002-x86_64_v2-unknown-linux-gnu-install_only.tar.gz -C /opt/`

[root@localhost vllm-offline]# `ln -s /opt/python/bin/python3.11 /usr/local/bin/python3.11`

[root@localhost vllm-offline]# `ln -s /opt/python/bin/pip /usr/local/bin/pip`

[root@localhost ~]# `tar xzvf vllm-offline.tar.gz`

[root@localhost vllm-offline]# `/opt/python/bin/pip install --find-links wheels/ -r requirements.txt  --no-index`

[root@localhost vllm-offline]#  `python3.11 -c "import vllm; print(vllm.__version__)"`

0.14.1

