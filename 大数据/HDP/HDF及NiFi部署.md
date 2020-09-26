# HDF及NiFi 部署

需要把 HDF 集成进 HDP 中，然后安装 NiFI。



## 集成 HDF

HDF 版本：3.4.1.1-4

HDF 安装教程：https://docs.cloudera.com/HDPDocuments/HDF3/HDF-3.4.1.1/installing-hdf-on-hdp/content/hdf-upgrade-ambari-and-hdp.html

HDF 下载界面：https://docs.cloudera.com/HDPDocuments/HDF3/HDF-3.4.1.1/release-notes/content/hdf_repository_locations.html

CentOS7 版 HDF mpack 包下载链接：http://public-repo-1.hortonworks.com/HDF/centos7/3.x/updates/3.4.1.1/tars/hdf_ambari_mp/hdf-ambari-mpack-3.4.1.1-4.tar.gz



hdf-ambari-mpack-3.4.1.1-4.tar.gz 下载好之后，运行：

```bash
$ ambari-server install-mpack --mpack=hdf-ambari-mpack-3.4.1.1-4.tar.gz
```

报错说：

```
INFO: Management pack hdf-ambari-mpack-3.4.1.1-4 successfully installed! Please restart ambari-server.
INFO: Executing after-install hook script : /var/lib/ambari-server/resources/mpacks/hdf-ambari-mpack-3.4.1.1-4/hooks/after_install.py
INFO: about to run command: ['/usr/bin/ambari-python-wrap', u'/var/lib/ambari-server/resources/mpacks/hdf-ambari-mpack-3.4.1.1-4/hooks/after_install.py']
INFO: 
process_pid=13493
ERROR: Failed to execute after-install hook. Failed with error code after-install
ERROR: Traceback (most recent call last):
  File "/var/lib/ambari-server/resources/mpacks/hdf-ambari-mpack-3.4.1.1-4/hooks/after_install.py", line 36, in <module>
    smartsense_versioner.fix_smartsense_versions()
  File "/var/lib/ambari-server/resources/mpacks/hdf-ambari-mpack-3.4.1.1-4/hooks/smartsense_versioner.py", line 58, in fix_smartsense_versions
    shell.checked_call(["cp", "-f", source_view_jar_file_path, new_view_jar_file_path], sudo=True)
  File "/usr/lib/ambari-server/lib/resource_management/core/shell.py", line 72, in inner
    result = function(command, **kwargs)
  File "/usr/lib/ambari-server/lib/resource_management/core/shell.py", line 102, in checked_call
    tries=tries, try_sleep=try_sleep, timeout_kill_strategy=timeout_kill_strategy, returns=returns)
  File "/usr/lib/ambari-server/lib/resource_management/core/shell.py", line 150, in _call_wrapper
    result = _call(command, **kwargs_copy)
  File "/usr/lib/ambari-server/lib/resource_management/core/shell.py", line 314, in _call
    raise ExecutionFailed(err_msg, code, out, err)
resource_management.core.exceptions.ExecutionFailed: Execution of 'cp -f /var/lib/ambari-server/resources/stacks/HDF/3.2.b/services/SMARTSENSE/package/files/view/smartsense-ambari-view-2.7.5.jar /var/lib/ambari-server/resources/stacks/HDF/3.2.b/services/SMARTSENSE/package/files/view/smartsense-ambari-view-1.5.0.2.7.5.0-72.jar' returned 1. cp: cannot stat '/var/lib/ambari-server/resources/stacks/HDF/3.2.b/services/SMARTSENSE/package/files/view/smartsense-ambari-view-2.7.5.jar': No such file or directory
```

看来是执行命令：`/usr/bin/ambari-python-wrap /var/lib/ambari-server/resources/mpacks/hdf-ambari-mpack-3.4.1.1-4/hooks/after_install.py` 过程中报错，因为找不到 `smartsense-ambari-view-2.7.5.jar` 导致的，smarsense 版本不对，这个目录中有 `smartsense-ambari-view-2.7.3.jar`

这个组件不重要，但是这个报错耽误了后面程序的执行。

解决方案，手动执行：

```
cd /usr/lib/python2.7/site-packages/ 

ln -s /usr/lib/ambari-server/lib/resource_management resource_management 
ln -s /usr/lib/ambari-server/lib/ambari_simplejson ambari_simplejson 
ln -s /usr/lib/ambari-server/lib/ambari_jinja2 ambari_jinja2 
ln -s /usr/lib/ambari-server/lib/ambari_commons ambari_commons 
ln -s /usr/lib/ambari-server/lib/ambari_server ambari_server

export PATH=$PATH:/var/lib/ambari-agent/
cp /var/lib/ambari-server/resources/stacks/HDF/3.2.b/services/SMARTSENSE/package/files/view/smartsense-ambari-view-2.7.3.jar  /var/lib/ambari-server/resources/stacks/HDF/3.2.b/services/SMARTSENSE/package/files/view/smartsense-ambari-view-2.7.5.jar
ambari-python-wrap /var/lib/ambari-server/resources/mpacks/hdf-ambari-mpack-3.4.1.1-4/hooks/after_install.py
```

完美解决！！！



然后重启 ambari-server：

```bash
$ ambari-server restart
```



重启完成后，浏览器打开 Ambari 的界面，配置HDF 的软件源。

CentOS7 版 HDF RPM 包下载链接：http://public-repo-1.hortonworks.com/HDF/centos7/3.x/updates/3.4.1.1/HDF-3.4.1.1-centos7-rpm.tar.gz

下载完成后，将其解压放入本地软件源中。

然后在 Ambari 界面中依次点击：Admin --> Manager Ambari --> Versions --> HDP-3.1 ，

在 HDF-3.4 中填入 URL：http://server02.bbdops.net/HDF/centos7/3.4.1.1-4

点击 Save 保存。



## 安装 NiFi

安装完 HDF 后，添加 NiFi 的 Service 即可。

`Encrypt Configuration Master Key Password` 、 `Sensitive property values encryption password` 、`NiFi CA Token` 都是设置的 nifi_password。































