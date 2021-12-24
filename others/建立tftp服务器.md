# 在ubuntu上安装tftp服务

```
#安装相关软件包
sudo apt-get install tftp-hpa tftpd-hpa xinetd

#创建tftp文件价，并设置权限
sudo mkdir /tftpboot
sudo chmod 777 /tftpboot

#进入目录 /etc/xinetd.d/，并在其中新建文件tftp，加入指定内容
cd /etc/xinetd.d/
sudo nano tftp

#加入以下内容
service tftp  
{  
	disable = no 138  
	socket_type = dgram  
	protocol = udp  
	wait = yes  
	user = root  
	server = /usr/sbin/in.tftpd  
	server_args = -s /tftpboot -c  
	per_source = 11  
	cps = 100 2  
}
	
#修改配置文件 /etc/default/tftpd-hpa，修改为
TFTP_USERNAME="tftp"
TFTP_DIRECTORY="/tftpboot"
TFTP_ADDRESS="0.0.0.0:69"
TFTP_OPTIONS="--secure -l -c -s"
	
#重新启动服务
sudo /etc/init.d/xinetd reload
sudo /etc/init.d/xinetd restart
sudo /etc/init.d/tftpd-hpa restart
```

