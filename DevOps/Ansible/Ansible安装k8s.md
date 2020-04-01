# Annsible 安装 k8s

Playbook 文件：

```yaml
---
- name: k8s 1.18.0 install
  hosts: fueltank-3
  remote_user: admin
  become: yes
  tasks:
    - name: 复制配置文件
      synchronize:
        src: ./config/kubernetes
        dest: /etc/
        checksum: yes
    - name: 修改配置文件所属用户及组
      shell: chown -R root:root /etc/kubernetes

    - name: 复制各组件二进制文件
      with_fileglob:
        - ./binary/*
      synchronize:
        src: "{{item}}"
        dest: /usr/bin/
        checksum: yes

    - name: 复制 service 文件
      with_fileglob:
        - ./service/*
      synchronize:
        src: "{{item}}"
        dest: /usr/lib/systemd/system/
        checksum: yes
        dirs: no

    - name: 修改 kube-apiserver 配置 --advertise-address
      lineinfile:
         dest: /etc/kubernetes/kube-apiserver
         regexp: "^--advertise-address="
         line: "--advertise-address={{ip}} \\"
    - name: 修改 kube-apiserver 配置 --bind-address
      lineinfile:
         dest: /etc/kubernetes/kube-apiserver
         regexp: "^--bind-address="
         line: "--bind-address={{ip}} \\"
    - name: 修改 kube-apiserver 配置 --external-hostname
      lineinfile:
         dest: /etc/kubernetes/kube-apiserver
         regexp: "^--external-hostname="
         line: "--external-hostname={{hostname}} \\"
    - name: 修改 kube-apiserver 配置 --external-hostname
      lineinfile:
         dest: /etc/kubernetes/kube-apiserver
         regexp: "^--external-hostname="
         line: "--external-hostname={{hostname}} \\"
    - name: 修改 kubelet 配置 --hostname-override
      lineinfile:
         dest: /etc/kubernetes/kubelet
         regexp: "^--hostname-override="
         line: "--hostname-override={{hostname}} \\"

    - name: 启动 kube-apiserver 服务
      systemd:
        name: kube-apiserver
        state: started
        enabled: yes
    - name: 启动 kube-controller-manager 服务
      systemd:
        name: kube-controller-manager
        state: started
        enabled: yes
    - name: 启动 kube-scheduler 服务
      systemd:
        name: kube-scheduler
        state: started
        enabled: yes
    - name: 启动 kube-proxy 服务
      systemd:
        name: kube-proxy
        state: started
        enabled: yes
    - name: 启动 kubelet 服务
      systemd:
        name: kubelet
        state: started
        enabled: yes

    - name: 复制 admin 的 kubectl 配置文件
      synchronize:
        src: ./config/admin-config.yaml
        dest: /home/admin/.kube/config
        checksum: yes
```



执行：

```bash
$ ansible-playbook k8s-install.yaml -f 10 
```

