# OpenWrt安装Tailscale

用ssh登录OpenWrt进行安装。

```bash
opkg update
opkg install tailscale
```

第一次运行时设置
```bash
/etc/init.d/tailscale enable
/etc/init.d/tailscale start
tailscale up --advertise-routes=192.168.2.1/24
```
其中192.168.2.1/24为OpenWrt局域网地址及掩码。
此时会返回一个URL链接，点击链接或在浏览器输入链接，导航到Tailscale网站，用Tailscale帐号登录(如当前浏览器已登录此步骤自动忽略)，自动完成此台OpenWrt设备与Tailscale帐号绑定。


## 测试Tailscale
登录Tailscale帐号，正常状态下，可看到OpenWrt设备已在线。
(略)