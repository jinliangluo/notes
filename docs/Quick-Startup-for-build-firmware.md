# Quick Startup

# 【编译】

## 1. 下载代码仓库

我们的代码采用多仓库，submodule的方式组织，因此先下载一个总的release工程：

````
git clone git@github.com:hujia-team/fpga_release.git
````

随后使用submodule命令进行更新：

````
git submodule init
git submodule update
````

若发现某些仓库没有权限，可以指定目录进行更新，譬如更新Linux kernel：

````
git submodule update linux-xlnx-mirror
````

## 2. 安装依赖工具&库

````shell
# 安装Makefile配置工具
sudo apt-get install autoconf
# 安装交叉编译 toolchain
sudo apt-get install gcc-4.9-arm-linux-gnueabihf
ln -s /usr/bin/gcc-4.9-arm-linux-gnueabihf-gcc-4.9 /usr/bin/gcc-4.9-arm-linux-gnueabihf-gcc
# 安装uboot工具
sudo apt-get install u-boot-tools
# 安装ssl库
sudo apt-get install libssl-dev
# 安装设备书编译器
sudo apt-get install device-tree-compiler
# 修改dash为bash
sudo dpkg-reconfigure dash
# 安装打包工具
sudo apt-get install mtd-utils
# 安装cmake
sudo apt-get install cmake
````

## 3. 编译 & 打包

````shell
#将embeddedsw checkout到20172
cd embeddedsw/
git checkout -b 20172 remotes/origin/20172
# 执行编译
./build.sh
# 执行打包
./package.sh
````

## 4. 编译Opencv

从其他同事处获取opencv的release包，随后执行：

```shell
sudo apt-get update
sudo apt-get build-dep opencv
cmake . && make && sudo make install
```

# 【连接开发板调试】

## 1. 准备

下载 pcview_client, cam_client, dbcp, id_rsa.db

安装dropbear

````shell
sudo apt-get install dropbear
````

连接开发板：

````shell
dbclient -i id_rsa.db root@192.168.0.233
````

调试文件复制：

````shell
dbcp -i id_rsa.db   src root@192.168.0.233:/directory/target
````

看算法输出图：

````shell
./pcview_client 192.168.0.233
````

看灰度图：

````shell
# 下位机运行：
cam_server
# PC上运行：
cam_client 192.168.0.233
````

