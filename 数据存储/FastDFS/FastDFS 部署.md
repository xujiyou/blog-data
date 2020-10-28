# FastDFS 部署

参考文档：https://segmentfault.com/a/1190000022045552

安装方式：编译安装

版本：最新版

tracker 和 storage 是同一个 fastdfs 程序的两个不同的概念，



## 安装

安装基础软件：

```bash
$ sudo yum install gcc gcc-c++ libevent -y
```

解压并安装 libfastcommon：

```bash
$ tar zxvf libfastcommon-1.0.43.tar.gz
$ ./make.sh
$ sudo ./make.sh install
```

安装结果：

```bash
mkdir -p /usr/lib64
mkdir -p /usr/lib
mkdir -p /usr/include/fastcommon
install -m 755 libfastcommon.so /usr/lib64
install -m 644 common_define.h hash.h chain.h logger.h base64.h shared_func.h pthread_func.h ini_file_reader.h _os_define.h sockopt.h sched_thread.h http_func.h md5.h local_ip_func.h avl_tree.h ioevent.h ioevent_loop.h fast_task_queue.h fast_timer.h process_ctrl.h fast_mblock.h connection_pool.h fast_mpool.h fast_allocator.h fast_buffer.h skiplist.h multi_skiplist.h flat_skiplist.h skiplist_common.h system_info.h fast_blocked_queue.h php7_ext_wrapper.h id_generator.h char_converter.h char_convert_loader.h common_blocked_queue.h multi_socket_client.h skiplist_set.h fc_list.h json_parser.h buffered_file_writer.h /usr/include/fastcommon
if [ ! -e /usr/lib/libfastcommon.so ]; then ln -s /usr/lib64/libfastcommon.so /usr/lib/libfastcommon.so; fi
```

解压并安装 fastdfs：

```bash
$ tar zxvf fastdfs-6.06.tar.gz 
$ ./make.sh
$ sudo ./make.sh install
```

安装结果：

```bash
mkdir -p /usr/bin
mkdir -p /etc/fdfs
cp -f fdfs_trackerd /usr/bin
if [ ! -f /etc/fdfs/tracker.conf.sample ]; then cp -f ../conf/tracker.conf /etc/fdfs/tracker.conf.sample; fi
if [ ! -f /etc/fdfs/storage_ids.conf.sample ]; then cp -f ../conf/storage_ids.conf /etc/fdfs/storage_ids.conf.sample; fi
mkdir -p /usr/bin
mkdir -p /etc/fdfs
cp -f fdfs_storaged  /usr/bin
if [ ! -f /etc/fdfs/storage.conf.sample ]; then cp -f ../conf/storage.conf /etc/fdfs/storage.conf.sample; fi
mkdir -p /usr/bin
mkdir -p /etc/fdfs
mkdir -p /usr/lib64
mkdir -p /usr/lib
cp -f fdfs_monitor fdfs_test fdfs_test1 fdfs_crc32 fdfs_upload_file fdfs_download_file fdfs_delete_file fdfs_file_info fdfs_appender_test fdfs_appender_test1 fdfs_append_file fdfs_upload_appender fdfs_regenerate_filename /usr/bin
if [ 0 -eq 1 ]; then cp -f libfdfsclient.a /usr/lib64; cp -f libfdfsclient.a /usr/lib/;fi
if [ 1 -eq 1 ]; then cp -f libfdfsclient.so /usr/lib64; cp -f libfdfsclient.so /usr/lib/;fi
mkdir -p /usr/include/fastdfs
cp -f ../common/fdfs_define.h ../common/fdfs_global.h ../common/mime_file_parser.h ../common/fdfs_http_shared.h ../tracker/tracker_types.h ../tracker/tracker_proto.h ../tracker/fdfs_shared_func.h ../tracker/fdfs_server_id_func.h ../storage/trunk_mgr/trunk_shared.h tracker_client.h storage_client.h storage_client1.h client_func.h client_global.h fdfs_client.h /usr/include/fastdfs
if [ ! -f /etc/fdfs/client.conf.sample ]; then cp -f ../conf/client.conf /etc/fdfs/client.conf.sample; fi
```

- /usr/bin 包含了可执行文件
- /etc/fdfs 包含了配置文件

拷贝配置文件：

```bash
$ sudo cp conf/* /etc/fdfs/
```



## 配置 Tracker 服务

创建系统用户：

```bash
$ sudo useradd -r fastdfs
```

修改配置文件`/etc/fdfs/tracker.conf`

```
base_path = /hadoop/fastdfs-tracker
run_by_group = fastdfs
run_by_user = fastdfs
```

创建数据目录：

```bash
$ sudo mkdir /hadoop/fastdfs-tracker
$ sudo chown fastdfs:fastdfs /hadoop/fastdfs-tracker
```

启动 tracker：

```bash
$ sudo /usr/bin/fdfs_trackerd /etc/fdfs/tracker.conf start
```

查看状态：

```bash
$ sudo netstat -nltp | grep 22122
$ sudo ps -ef | grep tracker
$ sudo lsof -i:22122
```



## 配置 stroage 服务

修改配置文件 `/etc/fdfs/storage.conf`

```
base_path = /hadoop/fastdfs-storage
store_path0 = /hadoop/fastdfs-storage
tracker_server = 172.19.40.16:22122
```

创建目录：

```bash
$ sudo mkdir /hadoop/fastdfs-storage
```

启动 storage：

```bash
$ sudo /usr/bin/fdfs_storaged /etc/fdfs/storage.conf start
```

查看状态：

```bash
$ sudo ps -ef | grep storage
```



## 测试上传

修改配置文件 `/etc/fdfs/client.conf`

```
base_path = /hadoop/fastdfs-client
tracker_server = 172.19.40.16:22122
```

创建目录：

```bash
$ sudo mkdir /hadoop/fastdfs-client
```

测试上传：

```bash
$ echo "hello world" > test.txt
$ /usr/bin/fdfs_test /etc/fdfs/client.conf upload test.txt 
```

返回如下：

```
This is FastDFS client test program v6.06

Copyright (C) 2008, Happy Fish / YuQing

FastDFS may be copied only under the terms of the GNU General
Public License V3, which may be found in the FastDFS source kit.
Please visit the FastDFS Home Page http://www.fastken.com/ 
for more detail.

[2020-10-28 10:11:42] DEBUG - base_path=/hadoop/fastdfs-client, connect_timeout=5, network_timeout=60, tracker_server_count=1, anti_steal_token=0, anti_steal_secret_key length=0, use_connection_pool=0, g_connection_pool_max_idle_time=3600s, use_storage_id=0, storage server id count: 0

tracker_query_storage_store_list_without_group: 
        server 1. group_name=, ip_addr=172.19.40.16, port=23000

group_name=group1, ip_addr=172.19.40.16, port=23000
storage_upload_by_filename
group_name=group1, remote_filename=M00/00/00/rBMoEF-Y016AcZcWAAAADK8IOy0472.txt
source ip address: 172.19.40.16
file timestamp=2020-10-28 10:11:42
file size=12
file crc32=2936552237
example file url: http://172.19.40.16/group1/M00/00/00/rBMoEF-Y016AcZcWAAAADK8IOy0472.txt
storage_upload_slave_by_filename
group_name=group1, remote_filename=M00/00/00/rBMoEF-Y016AcZcWAAAADK8IOy0472_big.txt
source ip address: 172.19.40.16
file timestamp=2020-10-28 10:11:42
file size=12
file crc32=2936552237
example file url: http://172.19.40.16/group1/M00/00/00/rBMoEF-Y016AcZcWAAAADK8IOy0472_big.txt
```

在本次测试中，数据存放在了 `/hadoop/fastdfs-storage/data/00/00/` 中。



## 配置 Nginx

fastdfs 安装好之后是无法通过 http访问的，这时候需要配置 Nginx。

解压 fastdfs 的 nginx 插件包：

```bash
$ tar zxvf fastdfs-nginx-module-1.22.tar.gz 
```

解压后进入目录，然后拷贝配置文件：

```bash
$ sudo cp src/mod_fastdfs.conf /etc/fdfs/
```

再修改其中的 `src/config` 文件，将 local 字符删除！

配置编译 Nginx 选项：

```
$ tar zxvf nginx-1.18.0.tar.gz 
$ ./configure \
--prefix=/usr/local/nginx \
--pid-path=/var/run/nginx/nginx.pid \
--lock-path=/var/lock/nginx.lock \
--error-log-path=/var/log/nginx/error.log \
--http-log-path=/var/log/nginx/access.log \
--with-http_gzip_static_module \
--http-client-body-temp-path=/var/temp/nginx/client \
--http-proxy-temp-path=/var/temp/nginx/proxy \
--http-fastcgi-temp-path=/var/temp/nginx/fastcgi \
--http-uwsgi-temp-path=/var/temp/nginx/uwsgi \
--http-scgi-temp-path=/var/temp/nginx/scgi \
--add-module=/home/bbders/fastdfs/fastdfs-nginx-module-1.22/src
```

返回结果：

```
Configuration summary
  + using system PCRE library
  + OpenSSL library is not used
  + using system zlib library

  nginx path prefix: "/usr/local/nginx"
  nginx binary file: "/usr/local/nginx/sbin/nginx"
  nginx modules path: "/usr/local/nginx/modules"
  nginx configuration prefix: "/usr/local/nginx/conf"
  nginx configuration file: "/usr/local/nginx/conf/nginx.conf"
  nginx pid file: "/var/run/nginx/nginx.pid"
  nginx error log file: "/var/log/nginx/error.log"
  nginx http access log file: "/var/log/nginx/access.log"
  nginx http client request body temporary files: "/var/temp/nginx/client"
  nginx http proxy temporary files: "/var/temp/nginx/proxy"
  nginx http fastcgi temporary files: "/var/temp/nginx/fastcgi"
  nginx http uwsgi temporary files: "/var/temp/nginx/uwsgi"
  nginx http scgi temporary files: "/var/temp/nginx/scgi"
```

编译并安装 Nginx：

```bash
$ make
$ sudo make install
```

修改 `/etc/fdfs/mod_fastdfs.conf`

```
tracker_server=172.19.40.16:22122
url_have_group_name = true
store_path0=/hadoop/fastdfs-storage
```

修改 `/usr/local/nginx/conf/nginx.conf `

```
    server {
        listen       80;
        server_name  localhost;

        location /group1/M00 {
            ngx_fastdfs_module;
        }

    }
```

启动 Nginx：

```bash
$ sudo mkdir -p /var/temp/nginx/client
$ cd /usr/local/nginx/sbin
$ sudo ./nginx
```

停止 Nginx：

```bash
$ sudo ./nginx -s stop
```

浏览器访问上面的测试文件：http://172.19.40.16/group1/M00/00/00/rBMoEF-Y016AcZcWAAAADK8IOy0472_big.txt















































