# sockcan 使用指南



1. 建立套接字

   ```
   int fd = socket(PF_CAN, SOCK_RAW, CAN_RAW);
   if (0 > fd)
   {
       perror("socket");
       return -1;
   }
   ```

2. 绑定端口

   ```
   struct ifreq ifr;
   memset(&ifr, 0x00, sizeof(struct ifreq));
   strcpy(ifr.ifr_name, "can0");
   if (0 > ioctl(fd, SIOCGIFINDEX, &ifr))
   {
       perror("ioctl");
       close(fd);
       return -1;
   }
   
   struct sockaddr_can addr;
   addr.can_ifindex = ifr.ifr_ifindex;	// 如果要绑定所有端口，可以设置addr.can_ifindex = 0;
   if (0 > bind(fd, (struct sockaddr *)&addr, sizeof(addr)))
   {
       perror("bind");
       close(fd);
       return -1;
   }
   ```

3. 滤波设置

   ```
   //方式1、设置一个或多个滤波(示例2个，id_1=0x7df, id_2=0x726)
   struct can_filter filter[N]; #根据实际需要设置几个滤波值
   filter[0].can_id = 0x726;
   filter[0].can_mask = 0x7ff;
   filter[0].can_id = 0x7df;
   filter[0].can_mask = 0x7ff;
   if (0 > setsockopt(fd, SOL_CAN_RAW, CAN_RAW_FILTER, &filter, sizeof(filter)))
   {
       perror("setsockopt");
       close(fd);
       return -1;
   }
   
   // 方式2、设置接收所有报文
   struct can_filter filter; #根据实际需要设置几个滤波值
   filter.can_mask = 0x0;	// mask设置为0,不屏蔽任何报文
   if (0 > setsockopt(fd, SOL_CAN_RAW, CAN_RAW_FILTER, &filter, sizeof(filter)))
   {
       perror("setsockopt");
       close(fd);
       return -1;
   }
   
   // 方式3、设置不接收报文
   if (0 > setsockopt(fd, SOL_CAN_RAW, CAN_RAW_FILTER, NULL, 0))
   {
       perror("setsockopt");
       close(fd);
       return -1;
   }
   ```

4. 环回功能

   ```
   int loopback = 0;	// 0=disable; 1=enable(default); 本地环回功能默认开启
   if (0 > setsockopt(can_fd, SOL_CAN_RAW, CAN_RAW_LOOPBACK, &loopback, sizeof(loopback)))
   {
       perror("setsockopt");
       close(can_fd);
       return -1;
   }
   ```

5. CAN发送

   ```
   // 1、发送标准帧
   struct can_frame frame;
   frame.can_id = 0x555;
   frame.can_dlc = 8;
   char buf[]="01234567";
   memcpy(frame.data, buf, 8);
   write(can_fd, &frame, sizeof(struct can_frame));
   
   // 2、发送扩展帧
   struct can_frame frame;
   frame.can_id = 0x18fe5be8 | CAN_EFF_FLAG;
   frame.can_dlc = 8;
   char buf[]="01234567";
   memcpy(frame.data, buf, 8);
   write(can_fd, &frame, sizeof(struct can_frame));
   
   // 3、使用sendto
   struct ifreq ifr;
   struct sockaddr_can addr;
   strcpy(ifr.ifr_name, "can0");
   ioctl(fd, SIOCGIFINDEX, &ifr);
   addr.can_ifindex = ifr.ifr_ifindex;
   addr.can_family  = AF_CAN;
   struct can_frame frame;
   int len;
   int nbytes = sendto(fd, &frame, sizeof(struct can_frame),
                       0, (struct sockaddr*)&addr, sizeof(addr)); // 当fd绑定所有端口时需要使用sendto指定发送端口
   ```

6. CAN接收

   ```
   // 1、使用read
   struct can_frame frame;
   if (0 < read(fd, &frame, sizeof(struct can_frame)))
   {
       // 接收数据的处理   
   }
   
   // 2、使用recvfrom
   if (0 < recvfrom(can_fd, &frame, sizeof(struct can_frame), 0, (struct sockaddr*)&addr, &len)) // 当绑定所有端口时需要使用recvfrom指定接收端口
   {
   	// 接收数据的处理   
   }
   ```

7. CAN linux配置

   * 虚拟CAN配置

     ```
     # 创建并启动一个虚拟CAN： vcan0
     sudo modprobe vcan
     sudo ip link add dev vcan0 type vcan
     sudo ip link set up vcan0
     # 移除vcan0
     sudo ip link del vcan0
     ```

   * CAN设置

     ```
     # 设置CAN设备属性
     ip link set can0 type can help  #获取帮助信息，可以获取到CAN的常规配置参数及方法
     ip link set can0 up #启动CAN0
     ip link set can0 down #关闭CAN0，CAN的参数需要在关闭时配置
     ip link set can0 type can bitrate 500000 #设置波特了500KHz
     
     # 获取CAN设备详情
     ip -details -statistics link show can0 # 或者 ip -details link show can0
     
     ```

8.  头文件

   ```
   #include <sys/ioctl.h>
   #include <sys/socket.h>
   #include <linux/can.h>
   #include <linux/can/raw.h>
   #include <linux/can/error.h>
   #include <net/if.h>
   #include <errno.h>
   #include <string.h>
   ```

9. can_id的一些标志位

   ```
   #define CAN_EFF_FLAG	0x80000000U	// 扩展帧标识
   #define CAN_RTR_FLAG	0x40000000U	// 远程帧标识
   #define CAN_ERR_FLAG	0x20000000U	// 错误帧标识
   ```
