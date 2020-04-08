# Python 离线安装包

由于项目需要离线安装包，这里测试记录一下。安装的包包括：Flask、Pandas、Flask-Script。

python 版本是 2.7.

## Flask 离线安装

Flask 官方安装教程：https://flask.palletsprojects.com/en/1.1.x/installation/#installation

需要下载各种源码包。

- werkzeug：https://github.com/pallets/werkzeug/releases
- jinja：https://github.com/pallets/jinja/releases
- MarkupSafe ：https://github.com/pallets/markupsafe/releases
- itsdangerous：https://github.com/pallets/itsdangerous/releases
- click：https://github.com/pallets/click/releases
- flask：https://github.com/pallets/flask/releases

都下载最新版本的即可，然后按照顺序依次执行：

```bash
$ pip install -U /Volumes/新加卷/tianjin_project/Python/flask/werkzeug-1.0.1.tar.gz
$ pip install -U /Volumes/新加卷/tianjin_project/Python/flask/jinja-2.11.1.tar.gz
$ pip install -U /Volumes/新加卷/tianjin_project/Python/flask/markupsafe-1.1.1.tar.gz
$ pip install -U /Volumes/新加卷/tianjin_project/Python/flask/itsdangerous-1.1.0.tar.gz
$ pip install -U /Volumes/新加卷/tianjin_project/Python/flask/click-7.1.1.tar.gz
$ pip install -U /Volumes/新加卷/tianjin_project/Python/flask/flask-1.1.2.tar.gz
```

亲测有效。

查看已经安装的包：

```bash
$ pip list
```



## Pandas 离线安装

Pandas 官方安装教程：https://pandas.pydata.org/docs/getting_started/install.html

需要下载的源码包：

- setuptools：https://github.com/pypa/setuptools/releases
- NumPy：https://github.com/numpy/numpy/releases
- python-dateutil：https://github.com/dateutil/dateutil/releases
- pytz：https://github.com/stub42/pytz/releases
- numexpr：https://github.com/pydata/numexpr/releases
- bottleneck：https://github.com/pydata/bottleneck
- pandas：https://github.com/pandas-dev/pandas/releases

安装过程各种出错，不兼容，遂放弃。

使用 `Anaconda3-5.2.0-Linux-x86_64.sh` 真香。里面有各种科学计算包

我司离线源地址：http://mirrors.bbdops.com/list/anaconda/archive/



## Flask-Script 离线安装

这个比较简单，下载地址：https://github.com/smurfix/flask-script/releases

下载完成后直接 `pip install -U ` 即可。





