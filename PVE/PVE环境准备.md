# PVE环境准备
- `rufus.exe`通过DD镜像模式写入

## 安装到EMMC
安装选项选择`Install Proxmox VE (Debug mode)`：
- 第一次出现shell直接exit
- 第二次出现shell编辑`/usr/bin/proxinstall`如下：
```
} elsif ($dev =~ m|^/dev/[^/]+/hd[a-z]$|) {
   return "${dev}$partnum";
} elsif ($dev =~ m|^/dev/nvme\d+n\d+$|) {
   return "${dev}p$partnum";
} elsif ($dev =~ m|^/dev/mmcblk\d+$|) {
   return "${dev}p$partnum";
} else {
   die "unable to get device for partition $partnum on device $dev\n";
}
```
- 正常安装到emmc下，注意交换区大小置0，第三次出现shell直接exit

## 开启硬件直通
`nano /etc/default/grub`编辑下行：
```
GRUB_CMDLINE_LINUX_DEFAULT="quiet intel_iommu=on"
```
`update-grub`生效。
`nano /etc/modules`添加：
```
vfio
vfio_iommu_type1
vfio_pci
vfio_virqfd
```
重启生效。

## 添加硬盘
- 在`{node}->磁盘`下擦除磁盘
- 在`{node}->磁盘->目录`下创建

### 磁盘性能测试
连续写入：
```
dd if=/dev/zero of=./testfile bs=1G count=1 oflag=direct
```
读取性能（第二次读有缓存）：
```
dd if=./testfile of=/dev/null bs=4k
```
写入延迟：
```
dd if=/dev/zero of=./testfile bs=512 count=1000 oflag=direct
```
