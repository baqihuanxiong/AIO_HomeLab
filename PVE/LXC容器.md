# LXC容器
## CT模板换源
```
cp /usr/share/perl5/PVE/APLInfo.pm /usr/share/perl5/PVE/APLInfo.pm_back
sed -i 's|http://download.proxmox.com|https://mirrors.tuna.tsinghua.edu.cn/proxmox|g' /usr/share/perl5/PVE/APLInfo.pm
```
重启服务
```
systemctl restart pvedaemon.service
```

**创建时取消勾选无特权容器**

## 硬件映射和参数配置
`nano /etc/pvc/lxc/{CT_ID}.conf`加入硬件参数（通过`ls -l /dev/{DEVICE}`查看）：
```
lxc.cgroup2.devices.allow: c 10:200 rwm
lxc.cgroup2.devices.allow: c 226:0 rwm
lxc.cgroup2.devices.allow: c 226:128 rwm
lxc.cgroup2.devices.allow: c 29:0 rwm
lxc.mount.entry: /dev/net dev/net none bind,optional,create=dir
lxc.mount.entry: /dev/dri dev/dri none bind,optional,create=dir
lxc.mount.entry: /dev/fb0 dev/fb0 none bind,optional,create=dir
lxc.apparmor.profile: unconfined
lxc.cgroup.devices.allow: a
lxc.cap.drop: 
```

`lxc.cgroup.devices.allow: a`和`lxc.cap.drop: `保证docker可以使用`--privileged`参数。

## 安装docker
```
apt update
apt install curl -y
curl -sSL https://get.daocloud.io/docker | sh
```
卸载`apparmor`，否则无法使用docker：
```
systemctl stop apparmor
systemctl disable apparmor
apt remove --assume-yes --purge apparmor
```