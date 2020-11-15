# 交换内存 swap

```
echo "vm.swappiness = 0">> /etc/sysctl.conf 
nohup swapoff -a && swapon -a &
sudo sysctl -p

```

