# 编译运行 TiKV 与 PD

TiKV 是用 Rust 写的，PD 是用 Go 写的。

TiKV 源码地址：https://github.com/tikv/tikv

PD 源码地址：https://github.com/pingcap/pd



## 编译 TiKV

首先要在本地搭建一套 Rust 的环境，我这里早就搭好了。

下载代码后，弄到 Goland 后，会自动下载一堆依赖，耐心等待。

编译很简单，直接在源码根目录执行 `make` 即可，一段时间后，会在 `target` 目录下的 `release` 目录下生成二进制文件。二进制文件名为  `tikv-server` 与 `tike-ctl` 。



## 编译 PD

首先要在本地搭建一套 Go 的环境。

下载代码后，弄到 Goland 后，会自动下载一堆依赖，耐心等待。

PD 的编译也是执行 `make` 即可