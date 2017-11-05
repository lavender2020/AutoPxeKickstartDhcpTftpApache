# Reference URL    
http://www.linuxidc.com/Linux/2017-09/146763.htm    
# Automatic Installation of Ubuntu Server    
## Environment    
system:Ubuntu 16.04.3 LTS    
dhcp:4.3.3-5ubuntu12.7        
Tftp:0.17-18ubuntu2      
Apache:2.4.18-2ubuntu3.5     
Ip address:192.168.0.105
## Installation     
### DHCP
#### Dhcp service installation    
apt-get install isc-dhcp-server -y    #如果提示E: 无法定位软件包 isc-dhcp-Server，执行此命令apt-get update    

#### 配置dhcp服务     
     a.vim /etc/default/isc-dhcp-server    
        INTERFACES="ens33"       # 指定的网络接口名字    
     b.vim /etc/dhcp/dhcpd.conf    #在文件末尾添加即可    
        subnet 192.168.193.0 netmask 255.255.255.0 {      #dhcpserver 分配ip的子网192.168.193网段，必须和PXE server的一个网卡同一个网段     
        range 192.168.193.100 192.168.193.200;     #为客户端分配ip范围     
        default-lease-time 600;    
        max-lease-time 7200;    
        filename "pxelinux.0";     #通过tftp找到pxelinux.0文件，并下载     
        next-server 192.168.193.128;    #指定tftp server的ip     
        }    
     配置完重启系统    
     systemctl restart isc-dhcp-server     
     查看服务    
     netstat -tunlp|grep 67    
     udp        0      0 0.0.0.0:67              0.0.0.0:*                           2119/dhcpd     
#### 安装tftp服务    
apt-get install tftpd-hpa -y      #安装完成就ok了，使用默认配置即可，tftp目录是 /var/lib/tftpboot/    
vim /etc/default/tftpd-hpa   #默认配置    
TFTP_USERNAME="tftp"    
TFTP_DIRECTORY="/var/lib/tftpboot"     
TFTP_ADDRESS=":69"      
TFTP_OPTIONS="--secure"     
#### 安装apache2    
apt-get install apache2 -y   #也是安装完就可以了，http根目录是 /var/www/html/     
#### 拷贝及修改所需文件     
mkdir /var/www/html/ubuntu    
rm -fr /var/www/html/index.html      
mount /dev/cdrom /mnt     
cp -r /mnt/* /var/www/html/ubuntu/    
cp -r /var/www/html/ubuntu/install/netboot/* /var/lib/tftpboot/    
cp /var/www/html/ubuntu/preseed/ubuntu-server.seed /var/www/html/     

vim /var/www/html/ubuntu-server.seed   #文件最后添加      
live-installer/net-image=http://192.168.193.128/ubuntu/install/filesystem.squashfs    

#### 安装kickstart      
kickstart需要GUI界面，我因为是安装的server，所以需要安装桌面（如果是desktop版本就不需要），如下安装     
apt-get install ubuntu-desktop -y     
apt-get install system-config-kickstart -y      
安装完之后，重启一下进入桌面     

#### 打开终端执行system-config-kickstart     
弹出下图     
进行配置     
选择安装的软件包，这里选择不了，生成ks文件之后，直接在ks文件里添加    
在命令行root家目录     
cp ks.cfg /var/www/html/     
vim /var/www/html/ks.cfg   #添加安装的软件包      
skipx    #下面添加需要安装的软件包    
%packages    
openssh-server    
vim     

#### 编辑txt.cfg    
vim /var/lib/tftpboot/ubuntu-installer/amd64/boot-screens/txt.cfg      
default install     
label install      
        menu label ^Install     
        menu default     
        kernel ubuntu-installer/amd64/linux     
        append ks=http://192.168.193.128/ks.cfg vga=788 initrd=ubuntu-installer/amd64/initrd.gz live-installer/net-image=http://192.168.193.128/ubuntu/install/filesystem.squashfs
label cli
        menu label ^Command-line install    
        kernel ubuntu-installer/amd64/linux    
        append tasks=standard pkgsel/language-pack-patterns= pkgsel/install-language-support=false vga=788 initrd=ubuntu-installer/amd64/initrd.gz --- quiet 
     
#### 编辑文件default     

vim /var/lib/tftpboot/pxelinux.cfg/default     
\# D-I config version 2.0    
\# search path for the c32 support libraries (libcom32, libutil etc.)    
path ubuntu-installer/amd64/boot-screens/      
include ubuntu-installer/amd64/boot-screens/menu.cfg     
default ubuntu-installer/amd64/boot-screens/vesamenu.c32     
prompt 0     
timeout 10  #默认是0（手动），改为10（1秒后自动选择install选项）     
接下来就可以安装系统了。    
