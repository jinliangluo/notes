## conan 打包一个已经编译过的库



### 1. 初始化： 

+ new一个: `conan new LibName/0.0.1@User/testing`  

  ```bash
  conan new LibName/Version@User/testing # eg: conan new demo/1.0.0@luo/testing
  ```

+ 替换`conanfile.py`
  将附件中的`conanfile.py`替换当前目录下的`conanfile.py`

+ 根据第一步LibName修改conanfile.py里的名称(`name`)和版本号(`version`)

+ 注：更新一个工程的新版本时，不用再次执行此步骤，直接从"2.准备待打包的库"开始即可


### 2. 准备待打包的库

创建目录 `src/lib`和`src/include`，并将需要打包的`.so`/`.a`文件和`.h`分别复制到`src/lib`和`src/include`目录下



### 3. 打包

```bash
conan create . User/channel # 会生成一个caninput/1.0.0@luo/testing
```



### 4. 上传

打开以下网页https://conan.minieye.cc/artifactory/webapp/#/home，点击 `Set Me Up`下相应的路径可查看上传package的方法；

```eg: conan upload vehicleInfo_h20 -r MinieyePersonal --all```



### 5. 使用

使用`import`可以从平台上package仓库里直接下载.so到工程；

方法参考网页：https://docs.conan.io/en/latest/reference/conanfile_txt.html#imports