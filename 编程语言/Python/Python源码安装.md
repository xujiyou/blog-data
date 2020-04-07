# Python 源码安装

下载地址：https://www.python.org/downloads/release/python-382/

只有源码包，没有 RPM 包。下载源码包，编译安装。

解压：

```bash
$ tar zxvf Python-2.7.17.tgz
```

配置：

````bash
$ ./configure --prefix=/usr/local/
````

编译：

```bash
$ make 
```

安装：

```bash
$ make install
```

