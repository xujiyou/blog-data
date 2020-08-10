# Rancher 踩坑笔记

使用 Rancher 时，如果宿主机是 **CentOS 8**，可能会出现 cattle-cluster-agent-64465767c9-ppwqd 发送 ping 超时的错误，即使为 Pod 设置了 hosts 了也不行。

解决方案：https://github.com/rancher/rancher/issues/16454#issuecomment-658795185

在各个宿主机都执行一遍：

```bash
sudo iptables -P FORWARD ACCEPT

echo 'net.ipv4.ip_forward = 1' | sudo tee -a /etc/sysctl.d/50-docker-forward.conf

for mod in ip_tables ip_vs_sh ip_vs ip_vs_rr ip_vs_wrr; do sudo modprobe $mod; echo $mod | sudo tee -a /etc/modules-load.d/iptables.conf; done

sudo dnf -y install network-scripts

sudo systemctl enable network

sudo systemctl disable NetworkManager
```

