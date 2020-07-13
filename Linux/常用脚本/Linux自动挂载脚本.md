# Linux 自动挂载脚本

```bash
#!/bin/bash

cd `dirname $0`

function data_disks_list(){
	#查找系统盘
	sys_disk=`df -h|grep -w '/boot'|awk '{print $1}'|sed 's/.$//g'|uniq`
	#全部磁盘
	all_disks=`lsblk -dnp -e 9,11,180,253 --output KNAME,SIZE,RM,RO,ROTA|egrep -v 'mapper|\/dev\/md'|awk '$3=='0' && $4=='0''|sort`
	data_disks=`echo "$all_disks"|grep -v $sys_disk`
	echo "$data_disks" > data_disks.list
	echo -ne "-------------------------grab data_disks list success--------------\n"
}


function lv_ssd_sata(){
	cp /etc/fstab /tmp/fstab.`date +%F-%H_%M_%S`
	ssd_sata=`cat data_disks.list | awk '{if ($5 == '0') print $1 }'|grep -v 'nvme'|xargs`
	lv='/dev/vg_ssd/lv_ssd'
	if [[ -n $ssd_sata ]];then
		if [[ ! -a $lv ]];then
			mountdir="/data1"
			wipefs -af $ssd_sata
			pvcreate $ssd_sata
			vgcreate vg_ssd $ssd_sata
			lvcreate -l +100%FREE -n lv_ssd vg_ssd
			mkfs.xfs -fq $lv
			[[ ! -d $mountdir ]] && mkdir -p $mountdir
			umount -l $mountdir 2> /dev/null
			echo "$lv	$mountdir	xfs	defaults,noatime	0 0" >> /etc/fstab
			mount -a
			echo -ne "-------------------------SATA_SSD mount success-------------------------\n"
		else
			echo -ne "-------------------------$lv exists,quit-------------------------\n"
			exit
		fi
	else
		echo -ne "-------------------------NO SATA_SSD FOUND-------------------------\n"
		#exit
	fi
}


function lv_ssd_nvme(){
	cp /etc/fstab /tmp/fstab.`date +%F-%H_%M_%S`
	ssd_nvme=`cat data_disks.list | awk '{if ($5 == '0') print $1 }'|grep 'nvme'|xargs`
	lv='/dev/vg_nvme/lv_nvme'
	mountdir="/data2"

	if [[ -n $ssd_nvme ]];then
		if [[ ! -a $lv ]];then
			wipefs -af $ssd_nvme
			pvcreate $ssd_nvme
			vgcreate vg_nvme $ssd_nvme
			lvcreate -l +100%FREE -n lv_nvme vg_nvme
			mkfs.xfs -fq $lv
			[[ ! -d $mountdir ]] && mkdir -p $mountdir
			umount -l $mountdir 2> /dev/null
			echo "$lv	$mountdir	xfs	defaults,noatime	0 0" >> /etc/fstab
			mount -a
			echo -ne "-------------------------NVME_SSD mount success-------------------------\n"
		else
			echo -ne "-------------------------$lv exists,quit-------------------------\n"
			exit
		fi
	else
		echo -ne "-------------------------NO NVME_SSD FOUND-------------------------\n"
		#exit
	fi
}


function mount_hdd(){
	cp /etc/fstab /tmp/fstab.`date +%F-%H_%M_%S`
	hdd=`cat data_disks.list | awk '{if ($5 == '1') print $1 }'|sort|xargs`
	#判断硬盘列表是否为空
	if [[ -n $hdd ]];then
		wipefs -af $hdd
		hdd_num=`cat data_disks.list | awk '{if ($5 == '1') print $1 }' | wc -l`
		var=1
		#循环挂载单个HDD
		while [[ $var -le $hdd_num ]];do
			for i in $hdd;do
				mkfs.xfs -fq $i
				uuid=`blkid -s UUID $i | awk '{print $2}'`
				[[ ! -d /data$var ]] && mkdir -p /data$var
				umount -l /data$var 2> /dev/null
				echo "$uuid	/data$var	xfs	defaults	0 0" >> /etc/fstab
				var=$((var + 1))
			done
		done
		mount -a
		echo -ne "-------------------------HDD mount success-------------------------\n"
	else
		echo -ne "-------------------------NO HDD FOUND-------------------------\n"
		#exit
	fi
}


function help_me(){
	echo
	echo "使用说明:该脚本用于自动判断磁盘类型(SATA-SSD/NVME-SSD/HDD)并格式化,挂载."
	echo
	echo "命令参数:"
	echo "-l: 查找出系统盘以外的磁盘,并输出到data_disks.list"
	echo "-s: 格式化全部SATA类型的ssd并挂载."
	echo "-n: 格式化全部nvme类型(PCI-E)的ssd并挂载."
	echo "-h: 格式化全部机械磁盘并挂载."
	echo "-a: 处理全部数据盘,包括SATA/NVME类型的SSD和机械盘."
}


#调用函数
if [[ $# -gt 0 ]];then
	while getopts "lsnha" opt; do
		case $opt in
		
			l)
				(data_disks_list);;

			s)
				(data_disks_list)
				(lv_ssd_sata);;

			n)
				(data_disks_list)
				(lv_ssd_nvme);;

			h)
				(data_disks_list)
				(mount_hdd);;
			
			a)
				(data_disks_list)
				(lv_ssd_sata)
				(lv_ssd_nvme)
				(mount_hdd);;
		esac
	done
else
	help_me
fi
```

