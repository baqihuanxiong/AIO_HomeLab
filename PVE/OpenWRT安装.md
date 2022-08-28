# OpenWRT安装
## 下载镜像
[openwrt](https://downloads.openwrt.org/releases/21.02.3/targets/x86/64/)
下载`x86-64-generic-ext4-combined-efi`镜像

## 安装
注意不需要磁盘，且添加直通网卡时最好按照顺序添加，启动前修改`/etc/pve/qemu-server/{VM_ID}.conf`，将镜像转化为磁盘：
```
ide2: local:iso/openwrt-21.02.3-x86-64-generic-ext4-combined-efi.img,cache=unsafe
```

## 查看日志
```
logread
```

## VLAN配置
在`Network->Interfaces->br-lan->Configuration->Bridge VLAN filtering`下勾选`Enable VLAN filtering`并配置VLAN标记，点`Save`后先**不要**点`Save & Apply`生效，修改`Network->Interfaces->lan->Global Settings->Device`为管理VLAN，然后点`Save & Apply`，**否则将无法连接Openwrt管理页面**。
![](./img/VLAN_tag.png)
![](./img/VLAN_interface.png)

## firewall配置
![](./img/firewall.png)

## mwan3配置
`/etc/config/mwan3`：
```
config rule 'wan2_rule'
        option src_ip '192.168.55.0/24'
        option proto 'all'
        option sticky '0'
        option use_policy 'wan2_only'

config rule 'default_wan'
        option proto 'all'
        option sticky '0'
        option use_policy 'wan_only'

config policy 'wan_only'
        list use_member 'wan_m10_w5'
        option last_resort 'unreachable'

config policy 'wan2_only'
        list use_member 'wan2_m20_w2'
        option last_resort 'unreachable'

config globals 'globals'
        option mmx_mask '0x3F00'

config interface 'wan'
        option initial_state 'online'
        option family 'ipv4'
        option track_method 'ping'
        option reliability '1'
        option count '1'
        option size '56'
        option max_ttl '60'
        option check_quality '0'
        option timeout '4'
        option interval '10'
        option failure_interval '5'
        option recovery_interval '5'
        option down '5'
        option up '5'
        list track_ip '114.114.114.114'
        list track_ip '223.5.5.5'
        option enabled '1'

config member 'wan_m10_w5'
        option interface 'wan'
        option metric '10'
        option weight '5'

config interface 'wan2'
        option initial_state 'online'
        option family 'ipv4'
        option track_method 'ping'
        option reliability '1'
        option count '1'
        option size '56'
        option max_ttl '60'
        option check_quality '0'
        option timeout '4'
        option interval '10'
        option failure_interval '5'
        option recovery_interval '5'
        option down '5'
        option up '5'
        list track_ip '114.114.114.114'
        list track_ip '223.5.5.5'
        option enabled '1'

config member 'wan2_m20_w2'
        option interface 'wan2'
        option metric '20'
        option weight '2'
```

## miniupnp配置
`/etc/config/upnpd`：
```
config upnpd 'config'
        option download '1024'
        option upload '512'
        option internal_iface 'lan untrusted'
        option external_iface 'wan'
        option port '5000'
        option upnp_lease_file '/var/run/miniupnpd.leases'
        option igdv1 '1'
        option enabled '1'
        option uuid 'bd170636-93e4-42e9-a838-420ff3cd07f8'

config perm_rule
        option action 'allow'
        option ext_ports '1024-65535'
        option int_addr '0.0.0.0/0'
        option int_ports '1024-65535'
        option comment 'Allow high ports'

config perm_rule
        option action 'deny'
        option ext_ports '0-65535'
        option int_addr '0.0.0.0/0'
        option int_ports '0-65535'
        option comment 'Default deny'
```
实测不支持填写多个wan到external_iface
```
service miniupnpd restart
service miniupnpd service
```