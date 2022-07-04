# MacOS刷OpenWrt启动U盘

## 安装brew
https://brew.sh


## 下载OpenWrt固件
https://downloads.openwrt.org/releases/
可看到所有已发行版本。

https://downloads.openwrt.org/releases/21.02.3/targets/x86/64/
可看到当前稳定版本21.02.3的x86-64分支。

下载文件
```bash
cd ~/Downloads
brew install wget

#下载文件
wget https://downloads.openwrt.org/releases/21.02.3/targets/x86/64/openwrt-21.02.3-x86-64-generic-squashfs-combined-efi.img.gz
wget https://downloads.openwrt.org/releases/21.02.3/targets/x86/64/sha256sums
wget https://downloads.openwrt.org/releases/21.02.3/targets/x86/64/sha256sums.asc
```

验证文件签名
```bash
brew install gpg
#接收公钥
gpg --keyserver keyserver.ubuntu.com --receive-keys 88CA59E88F681580
#验证签名
gpg  --verify sha256sums.asc sha256sums 2>&1| grep -e "Good signature" -e "完好的签名"
```
如显示字符包含"Good signature"或"完好的签名"，表示签名正确。

验证文件Hash
```bash
sha256sum openwrt-21.02.3-x86-64-generic-squashfs-combined-efi.img.gz|cut -d' ' -f1|grep -f- sha256sums
```
第1命令计算hash，第2命令按空格分割取第1字段，第3命令在sha256sums文件中查找hash值。
如显示找到hash值表示正常，否则错误。

解压缩
```bash
gunzip -k openwrt-21.02.3-x86-64-generic-squashfs-combined-efi.img.gz
```
得到文件openwrt-21.02.3-x86-64-generic-squashfs-combined-efi.img


## 用dd把OpenWrt固件刷到启动U盘

```bash
#列出所有磁盘
diskutil list
#一般可通过SIZE看出哪个是U盘，记下磁盘序号

#把下面diskX中的X替换成上面找到的磁盘序号
sudo dd if=openwrt-21.02.0-x86-64-generic-ext4-combined.img bs=1M of=/dev/diskX

#弹出U盘
#把下面diskX中的X替换成上面找到的磁盘序号
diskutil eject /dev/diskX
```

此时U盘可以启动OpenWrt，可正常使用，但分区大小只有100多兆。
如想扩容请用sgdisk扩展分区。


## 用sgdisk扩展分区(可选)


### 安装sgdisk
```bash
#安装sgdisk对应的gptfdisk
brew install gptfdisk
#显示版本号，测试是否能正常运行
sgdisk -V
```


显示U盘分区
```bash
#把下面diskX中的X替换成上面找到的磁盘序号
sudo sgdisk -p /dev/diskX
```
显示第U盘第2分区详细信息
```bash
sudo sgdisk -i 2 /dev/diskX
```
找到并记下Partition unique GUID(此处为529E51F2-6479-DFA3-F1D0-7FB0BEC60D02)、First sector(此处为33792)
```text
Partition unique GUID: 529E51F2-6479-DFA3-F1D0-7FB0BEC60D02
First sector: 33792 (at 16.5 MiB)
```

删除U盘第2分区，
```bash
sudo sgdisk -d 2 /dev/diskX
```
此时系统可能弹出对话框“此电脑不能读取您连接的磁盘”，点“忽略”。


新建第U盘第2分区，用上面记下的First sector(此处为33792)，
```bash
sudo sgdisk -n 2:33792:0 /dev/diskX
```
0表示把分区扩充到最大。


把第2分区改回上面记下的Partition unique GUID
```bash
sudo sgdisk -u 2:529E51F2-6479-DFA3-F1D0-7FB0BEC60D02 /dev/diskX
```
此时系统可能弹出对话框“此电脑不能读取您连接的磁盘”，点“推出”；
或点“忽略”后用命令行推出U盘。
```bash
diskutil eject /dev/diskX
```

## 利用启动U盘首次启动OpenWrt
OpenWrt首次启动会自动扩展文件系统占满当前分区。
如已连接显示器和键盘，可按Enter键直接进入终端。

查看网卡信息
```bash
ip a
```
如看不到网卡信息，可能是网卡驱动未安装，需要用ImageBuilder做一个包含网卡驱动的映像，重新刷U盘启动。

## 修改OpenWrt默认ip

OpenWrt默认使用第一张网卡eth0作为局域网Lan口，使用第二张网卡eth1作为广域网Wan口。
在Lan口eth0插入网线。
默认局域网Lan口ip为192.168.1.1/24。

如想修改ip，用ssh远程登录继续设置。
如当前局域网可用ipv6，可在OpenWrt终端用ip a查看ipv6地址。
还可以修改远程登录电脑ip为192.168.1.x/24进行连接。

ipv6登录
```bash
ssh root@fe80::2e2:xxxx:xxxx:xxxx
```

ipv4登录
```bash
ssh root@192.168.1.1
```

修改网络配置ip地址
```bash
vi /etc/config/network
```
内容如下
```text
config interface 'lan'
        option ipaddr '192.168.2.1'
```
然后重启
```bash
reboot
```

## 从Web登录设置
用浏览器访问
http://192.168.2.1/
用户名为root，密码不填，点“登录”进入，设置root密码。

## 连接互联网

修改Network|Interface|WAN|Edit，Protocol选PPPoE然后点切换，输入拨号username、passwd保存。

把Wan口网线插在第2网卡eth1。

## uhttpd设置
默认防火墙已禁止从wan访问，为更安全，取消监听互联网ip:port。
安装软件包luci-app-uhttpd。
取消“忽略公共接口上的私有 IP”，允许从固定的私有ipv6地址访问。
修改监听ip:port为局域网内。




