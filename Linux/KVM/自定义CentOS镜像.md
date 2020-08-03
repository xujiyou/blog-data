---
title: 自定义CentOS镜像
date: 2020-07-09 13:31:01
tags:
---

制作无人值守的 CentOS 安装镜像，可以提前预装软件，设置一些参数。



## 准备工作

挂载 ISO 镜像：

```bash
$ sudo mount /opt/CentOS-7-x86_64-Minimal-2003.iso /media/
```

创建工作目录，并拷贝数据：

```bash
$ mkdir /opt/iso
$ cp -r /media/* /opt/iso
```

安装工具：

```bash
$ yum install createrepo mkisofs isomd5sum
```



## 添加 RPM 包

准备在 ISO 中添加一个 JDK 的 RPM 包，实现系统安装完成后，自动安装 JDK。。

把 JDK 的 RPM 包放入 /opt/iso/packages 中。然后在 /opt/iso 中更新 repodata：

```bash
$ createrepo -g repodata/83b61f9495b5f728989499479e928e09851199a8846ea37ce008a3eb79ad84a0-c7-minimal-x86_64-comps.xml .
```

验证，添加 repo 文件：

```bash
$ vim /etc/yum.repos.d/local.repo
[iso]
name=iso_package
baseurl=file:///opt/iso
gpgcheck=0
enabled=1
```

安装 JDK 进行测试：

```bash
$ yum makecache
$ yum install jdk 
```

可以进行安装。



## 创建 ks.cfg 文件

ks.cfg文件即kickstart，这个文件定义了所有在安装过程中需要写入的内容。

```bash
$ vim /opt/iso/isolinux/ks.cfg
```

内容如下：

```bash
# 密码选项 使用屏蔽口令 加密方式为sha512
auth --enableshadow --passalgo=sha512
# 安装源为 U 盘，因为我的 PC 中有两块盘了，所以这里的 U 盘为 sdc4，还可以是 cdrom NFS HTTP FTP等
harddrive --partition=sdc4 --dir=sdc4
# 文本界面 设置graphical则为图形化界面
text

# 安装系统而不是更新系统
install
# 第一次启动 启用向导 X系统起作用
firstboot --enable
# 忽略其他磁盘 仅使用sda
ignoredisk --only-use=sda
# 键盘部局设置
keyboard --vckeymap=us --xlayouts='us'
# 系统语言支持
lang en_US.UTF-8

# 网络配置 这里是动态获取 启动时启用网卡
# 设置主机名
network  --bootproto=dhcp --device=enp2s0 --onboot=on --ipv6=auto --activate
network  --hostname=loaclhost.localdomain

# 设置root用户帐号 和/etc/shadow中密码字段内容一样,直接从别的系统中取下来即可 这里密码是123456
rootpw --iscrypted $6$SZ69HCyt$llZhDzWNhTGhvVwpRFlLLn3.2w22bZK6iF.A8QuC.4wlq3RDK3zHqhN24GAwtHzWAgXfaajVI.FwLHxKK7UKX1
# 添加一个bbders 用户，密码生成：openssl passwd -1 "bbders@bbdops.com"
user --name=bbders --password='$1$ngb.0U/n$J0BbpI7yDZzlUz3wp.OpN1' --iscrypted --gecos="bbders"

# 禁用chronyd服务
services --disabled="chronyd"
# 禁用防火墙
firewall --disabled
#禁用selinux
selinux --disabled
# 系统时区
timezone Asia/Shanghai --isUtc --nontp

# 系统引导配置 写入sda的mbr
bootloader --append=" crashkernel=auto" --location=mbr --boot-drive=sda
#autopart --type=lvm 自动分区 若磁盘够大 /boot分区和/home分区将会分配较大空间 不推荐

# 清空原有分区 初始化磁盘 仅针对sda
clearpart --all --initlabel --drives=sda

# 分区设置
# not lvm: default / and /var partion's size 307200 == 300GB , you need to adjust the size again;
# for GPT disk label condition, you have to create a 1Mib biosboot type partion;
part biosboot --fstype=biosboot --size=1
part /boot --asprimary --fstype="xfs" --mkfsoptions='-n ftype=1' --size=512
part swap  --fstype="swap" --label=swap --size=2048
part / --asprimary --fstype="xfs" --mkfsoptions='-n ftype=1' --grow --size=1

# 安装完成后重启
reboot --eject

# 指定要安装的软件包
# 默认
%packages
@^minimal
@core
jdk
%end

%addon com_redhat_kdump --enable --reserve-mb='auto'
%end

%anaconda
# 密码策略
pwpolicy root --minlen=6 --minquality=1 --notstrict --nochanges --notempty
pwpolicy user --minlen=6 --minquality=1 --notstrict --nochanges --emptyok
pwpolicy luks --minlen=6 --minquality=1 --notstrict --nochanges --notempty
%end

#%pre
# 安装前执行的脚本 注意 此时磁盘尚未格式化 所以能执行的工作有限
# 常用于读取磁盘信息 生成分区脚本 然后供之后格式化使用
#%end
 
# 安装后脚本
# %post --nochroot
# 安装后脚本有两种模式 一种是在安装环境运行 此时系统挂载于/mnt/sysimage 光盘挂载于/mnt/install/repo
# 可以从光盘复制文件到系统
 
# %post --log=/root/ks-post.log
# 另一种方式是直接在新系统中运行 可以运行新系统中几乎所有命令
# 但是如果安装源不被识别为/dev/sr0 则不能通过挂载的方式把光盘中的文件复制到系统

%post
echo 'do what you want'
#rm -f /root/anaconda-ks.cfg
chmod +x /etc/rc.d/rc.local
systemctl disable firewalld
systemctl enable acpid
# disable Ctrl+Alt+Del
systemctl mask ctrl-alt-del.target
systemctl daemon-reload

# bbders 用户加入 sudo 权限
echo "bbders ALL=(ALL:ALL) NOPASSWD:ALL" > /etc/sudoers.d/bbders

# ssh config
# 修改 ssh 端口
# sed -i 's/#Port 22/Port 51668/g' /etc/ssh/sshd_config
# 禁止root登录
sed -i 's/#PermitRootLogin yes/PermitRootLogin no/' /etc/ssh/sshd_config
sed -i -e "/GSSAPIAuthentication/s/yes/no/g" -e "/GSSAPICleanupCredentials/s/yes/no/g" -e"s/^#UseDNS\ no/UseDNS\ no/" -e"s/^#UseDNS\ yes/UseDNS\ no/" /etc/ssh/sshd_config
# 60秒后自动关闭 ssh
# echo -e "ClientAliveInterval 60\nClientAliveCountMax 10\n" >> /etc/ssh/sshd_config
# echo -e ‘export TMOUT=600\nreadonly TMOUT’ |tee -a /etc/profile
# 只允许 bbders 用户远程登录。
echo "AllowUsers bbders" >> /etc/ssh/sshd_config

# 历史命令存放地址和存放格式
mkdir /usr/share/.usermonitor/ && touch /usr/share/.usermonitor/usermonitor.log && chmod 002 /usr/share/.usermonitor/usermonitor.log
echo 'export HISTORY_FILE=/usr/share/.usermonitor/usermonitor.log' >> /etc/profile
echo export PROMPT_COMMAND="'"'{ echo "time="$(date "+%Y-%m-%dT%H:%M:%S")"'#user='"$(who am i |awk "{print \$1}")"'#ip='"$(who am i | awk "{print \$NF}" | grep -oP "[\d.]+")"'#command='"$(history 1 | { read x cmd; echo "$cmd"; });}' '>>' '${HISTORY_FILE}'"'" >> /etc/profile
echo 'shopt -s histappend' >> /etc/profile

# 设置最大进程数
cat > /etc/security/limits.d/20-nproc.conf << EOF
# Default limit for number of user's processes to prevent
# accidental fork bombs.
# See rhbz #432903 for reasoning.

#nproc
*          soft    nproc     65535
root       soft    nproc     unlimited
#nofile
*          soft    nofile     655350
*          hard    nofile     655350
EOF

# 设置 systemd 的配置，若在系统内修改，还需执行 systemctl daemon-reexec
cat >> /etc/systemd/system.conf << EOF
DefaultLimitCORE=infinity
DefaultLimitNOFILE=100000
DefaultLimitNPROC=100000
EOF
cat >>  /etc/systemd/user.conf << EOF
DefaultLimitCORE=infinity
DefaultLimitNOFILE=100000
DefaultLimitNPROC=100000
EOF

# 设置默认编辑器为 vim
cat >> /etc/bashrc << EOF
export EDITOR=/usr/bin/vim
EOF

# 修改内核参数
cat >> /etc/sysctl.conf << EOF
net.ipv4.ip_forward = 1
net.core.netdev_max_backlog = 262144
net.core.wmem_default = 8388608
net.core.rmem_default = 8388608
net.core.rmem_max = 16777216
net.core.wmem_max = 16777216
net.ipv4.route.gc_timeout = 20
net.ipv4.ip_local_port_range = 1025 65535
net.ipv4.tcp_retries2 = 5
net.ipv4.tcp_fin_timeout = 15
net.ipv4.tcp_syn_retries = 1
net.ipv4.tcp_synack_retries = 1
net.ipv4.tcp_timestamps = 0
net.ipv4.tcp_tw_recycle = 0
net.ipv4.tcp_tw_reuse = 1
net.ipv4.tcp_keepalive_time = 120
net.ipv4.tcp_keepalive_probes = 3
net.ipv4.tcp_keepalive_intvl = 15
net.ipv4.tcp_max_tw_buckets = 200000
net.ipv4.tcp_max_orphans = 3276800
net.ipv4.tcp_max_syn_backlog = 262144
net.ipv4.tcp_wmem = 8192 131072 16777216
net.ipv4.tcp_rmem = 32768 131072 16777216
net.ipv4.tcp_mem = 94500000 915000000 927000000
net.ipv4.netfilter.ip_conntrack_tcp_timeout_time_wait = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
vm.swappiness = 0
#vm.min_free_kbytes = 1024000
#fs.nr_open = 1048576
net.core.somaxconn = 16384
net.netfilter.nf_conntrack_max = 655350
net.netfilter.nf_conntrack_tcp_timeout_established = 1200
EOF


# netfilter 配置
cat >> /etc/modules-load.d/netfilter.conf << EOF
nf_conntrack
nf_conntrack_ipv4
EOF

modprobe nf_conntrack
modprobe nf_conntrack_ipv4

%end
```

然后修改 `/opt/iso/isolinux/isolinux.cfg` 中的：

```
menu title BBD Customer System based CentOS 7
label linux
  menu label ^Install CentOS 7
  menu default
  kernel vmlinuz
  append initrd=initrd.img inst.stage2=hd:LABEL=My_CentOS7 inst.ks=hd:sdc4:/isolinux/ks.cfg quiet
```

加入 `menu default` ，把 label check 里面的 `menu default` 注释掉。

这里 `append` 是重点，`inst.stage2` 后面是参数要靠 mkisofs 中的 -V 参数指定，`inst.ks` 指定 U 盘的盘符和 ks.cfg 的位置。

这里 U 盘的盘符需要数硬盘，比如我的 PC 上有两块盘，U 盘就是第三块盘，所以是 sdc，一般 U 盘都是 4，所以是 sdc4。



## 打包 ISO

打包

```bash
$ mkisofs -o /opt/My_CentOS7.iso -b isolinux/isolinux.bin -c isolinux/boot.cat -no-emul-boot -boot-load-size 4 -boot-info-table -joliet-long -R -J -v -V "My_CentOS7" -T /opt/iso/
```

注意 -V 参数 和 上边的 `inst.stage2=hd:LABEL` 要一致。

往 ISO 内加入 md5 校验码

```bash
$ implantisomd5 ../My_CentOS7.iso
$ checkisomd5 ../My_CentOS7.iso
```

这样制作的 ISO 文件就可以使用了，使用软碟通写入 U 盘，设置开机启动 U 盘即可。

在启动界面那里按 tab 可以修改 U 盘的盘符，我这里我算好了是 sdc 4 就可以不用修改了。

设置完盘符后按回车即可一键安装操作系统，自动安装完成后，会自动重启，这时候需要拔下 U 盘。



## IPMI 使用 ISO

在IPMI 安装操作系统时，ISO 对应的盘是 /dev/sr0。















