# Anaconda 安装

官方教程：https://docs.anaconda.com/anaconda/install/linux/

```bash
$ sudo yum install libXcomposite libXcursor libXi libXtst libXrandr alsa-lib mesa-libEGL libXdamage mesa-libGL libXScrnSaver
$ wget https://repo.anaconda.com/archive/Anaconda3-2020.11-Linux-x86_64.sh
$ bash Anaconda3-2020.11-Linux-x86_64.sh #按照提示安装，安装路径选 /opt/anaconda3
```

搞定。

几个命令：

```bash
$ /opt/anaconda3/bin/conda list
$ /opt/anaconda3/bin/conda install 
$ sudo /opt/anaconda3/bin/conda install redis
```

