# 获取线程id及其系统占用的方法(htop)



## 1、简单的查看进程占用的方法

```
top	# top命令获取系统进程占用情况
```



## 2、通过`htop`命令实现

1. 获取线程id

   ```
   #include <sys/types.h>
   #include <sys/syscall.h>
   
   void * test_thread(void *arg)
   {
       ...
       printf("pid = %ld, tid = %ld\n", pthread_self(), syscall(SYS_gettid));	// pthread_self()在进程里唯一，但想要获取线程在系统中的唯一id，需要通过syscall(SYS_gettid)获取tid
       ...
       
   }
   ```

2. 查看线程占用情况

   ```
   # 1. 安装htop（如果系统中没有安装htop）
   sudo apt install htop
   
   # 2. 启动htop
   htop #通过htop可以查看实时的线程占用情况;
   
   # 3. 配置下htop的显示
   F2(setup) --> “Display options” --> 勾选 "Tree view"和"Show program path" --> F10(Done)
   
   # 4. 可以通过F3搜索进程名快速定位到进程，查看相应进程下各线程(tid)的系统占用情况
   ```
