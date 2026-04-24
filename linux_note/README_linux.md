首次学习linux,推荐先阅读[菜鸟教程](https://www.runoob.com/linux/linux-tutorial.html),了解linux的作用 用法 为什么要用linux的原因.  
然后看一看[linux的目录结构](https://www.runoob.com/linux/linux-system-contents.html)linux是由哪几个文件夹组成的,和linux一切皆文件的道理,  
最后看[文件权限](https://www.runoob.com/linux/linux-file-attr-permission.html)和[目录相关](https://www.runoob.com/linux/linux-file-content-manage.html)的.  
熟悉相关知识后开始学习如何在linux中做各种操作,这里要边看边练,不懂的问ai直接贴个报错和问题  
例:  
![515](images/Pasted%20image%2020260420210613.png)  
问了之后要在虚拟机里实验加深记忆.  
提到虚拟机,这里可以使用cockpit自带的[web虚拟机控制台](https://192.168.31.245:9090),  
这里更加推荐使用戴尔服务器的[IDRC](https://192.168.31.246/)在里面使用kvm管理器  
新建虚拟机和编辑虚拟机的参数,因为这样可以更改更具体的配置    
IDRC使用教程:   
![](images/bandicam%202026-04-20%2021-39-21-641.mp4)  
cockpit使用教程:  
![](images/bandicam%202026-04-20%2021-42-21-893.mp4)  
打好基础后,看[linux](linux.md)这篇教程,跟着做  
掌握做题的方法,试着自己解决各种报错,可以使用`systemctl status 服务名`或者  
`journalctl -u 服务名 --no-pager -n 50` 看服务报错. 实在不行就贴报错给ai看  
### 关于服务的笔记:
[做题准备](linux.md#做题准备)   --[视频](http://192.168.31.245:8989/crazybaby/linux_video/-/raw/main/%E5%81%9A%E9%A2%98%E5%87%86%E5%A4%87.mp4?ref_type=heads)  
[chrony服务](linux.md#-2-利用chrony-配置linux1为其他linux主机提供ntp服务-)   