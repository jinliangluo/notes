# conan使用
## 1、 创建包
### 1.1、 创建

```bash
# -s :源码本地 -i: 不用编译
$ conan new test/1.0.0@luo/testing -t
```



### 1.2、 生成配置文件

```bash
$ conan install . demo/testing

# install 可以指定配置参数
$ conan create . demo/testing -s build_type=Debug
$ conan create . demo/testing -o Hello:shared=True -s arch=x86
$ conan create . demo/testing -pr my_gcc49_debug_profile
...
$ conan create ...

# 可以用命令配置某个包
$ conan install . -s MyPkg:compiler=gcc -s compiler=clang ..

# 可以选择编译成动态库
$ conan install . -o *:shared=True
```



### 1.3 修改仓库

```python
# 选择仓库可以使用self.run
from conans import ConanFile, CMake, tools

class HelloConan(ConanFile):
    ...
    def source(self):
        self.run("git clone https://github.com/memsharded/hello.git")
        self.run("cd hello && git checkout static_shared")

# 也可以使用git命令
from conans import ConanFile, CMake, tools

class HelloConan(ConanFile):
    ...
    def source(self):
        git = tools.Git(folder="hello")
        git.clone("https://github.com/memsharded/hello.git", "static_shared")
        ...
        
# 还可以使用scm  url
from conans import ConanFile, CMake, tools

class HelloConan(ConanFile):
     scm = {
        "type": "git",
        "subfolder": "hello",
        "url": "https://github.com/memsharded/hello.git",
        "revision": "static_shared"
     }
    ...
```



## 2 使用bin文件创建包

​	如果已有编译过的bin文件，不想再按照流程创建包，可以使用下面的办法

``` bash
$ # 先new一个包
$ conan new caninput/1.0.0@luo/testing --bare

$ # 再将bin文件和头文件copy到当前目录
$ cp ~/code/test/test.so ~/code/test/test.h .

$ # 再打包即生成了一个可被他人调用的conan包
$ conan export-pkg . caninput/1.0.0@luo/testing -s os=Linux -s compiler=gcc
```



## 3 上传

```bash
# 1. 添加一个仓库到conan CLI, <REMOTE>为自定义的一个名字，唯一标识仓库
conan remote add <REMOTE> https://conan.minieye.cc/artifactory/api/conan/MinieyeTesting
# 2. 登录
conan user -p AP5f*************ZE4dQ -r <REMOTE> luojinliang
# 3. 上传package， <RECIPE>为包名
conan upload <RECIPE> -r <REMOTE> --all
```

