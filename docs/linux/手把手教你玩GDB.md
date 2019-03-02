# 手把手教你玩GDB

**第一部分牛刀小试：启动GDB开始调试**

\1.       编译带调试信息的可执行程序：用gcc(g++)编译的时候带上-g选项即可

\2.       启动GDB开始调试

```
（1）gdb program       ///最常用的用gdb启动程序，开始调试的方式
（2）gdb program core   ///用gdb查看core dump文件，跟踪程序core的原因
（3）gdb program pid    ///用gdb调试已经开始运行的程序，指定pid即可
```

\3.       应用程序带命令行参数的情况，可以通过下面三种方法启动：

（1）启动GDB的时候，加上--args选项，然后把应用程序和其命令行参数带在后面，具体格式为：**gdb** --args *program args*

（2）先按1中讲的方法启动GDB,然后在执行run命令的时候，后面加上参数

（3）先按1中讲的方法启动GDB，然后用**set args** *arglist*设置命令行参数，接下来再执行**run**(**r**)命令运行程序

\4.       退出GDB：

（1）**End-of-File(Ctrl+d)**

（2）**quit**或者**q**

\5.       在GDB调试程序的时候执行shell命令：

（1）**shell***command args*（也可以先执行**shell**命令，GDB会退出到当前shell,执行完*command*后，然后在shell中执行**exit**命令，便可回到GDB）

（2）**make** *make-args*（等同于**shell make** *make-args*）

\6.       在GDB中获取帮助：

（1）在GDB中执行**help**命令，可以得到如图1所示的帮助信息：

 

图1 GDB帮助菜单

由图1可以看出，GDB中的命令可以分为八类：别名(aliases)、断点(breakpoints)、数据(data)、文件(files)、内部(internals)、隐含(obscure)、运行(running)、栈(stack)、状态(status)、支持(support)、跟踪点(tracepoints)和用户自定义(user-defined)。

（2）**help** *class-name*：查看该类型的命令的详细帮助说明

（3）**help all**：列出所有命令的详细说明

（4）**help** *command*：列出命令*command*的详细说明

（5）**apropos** *word*：列出与*word*这个词相关的命令的详细说明

（6）**complete** *args*：列出所有以*args*为前辍的命令

\7.       **info**和**show**：

（1）**info**：用来获取和被调试的应用程序相关的信息

（2）**show**：用来获取GDB本身设置相关的一些信息

**第二部分 Breakpoint, Watchpoint和Catchpoint**

\1.       Breakpoints相关命令：（Breakpoint的作用是让程序执行到某个特定的地方停止运行）

（1）设置breakpoint:

a. **break** *function*: 在函数funtion入口处设置breakpoint

b. **break +** *offset*:在程序当前停止的行向前offset行处设置breakpoint

c. **break –** *offset*:在程序当前停止的行向衙offset行处设置breakpoint

d. **break** *linenum*: 在当前源文件的第*linenum*行处设置breakpoint

e. **break** *filename***:***linenum*: 在名为*filename*的源文件的第*linenum*行处设置breakpoint

f. **break** *filename***:***function*: 在名为*filename*的源文件中的*function*函数入口处设置breakpoint

g. **break**  *address*:在程序的地址*address*处设置breakpoint

h. **break**:

i. **break … if** *cond*: **…**代表上面讲到的任意一个可能的参数，在某处设置一个breakpoint,但且仅但*cond*为**true**时，程序停下来

j. **tbreak** *args*: 设置一个只停止一次的breakpoints,*args*与**break**命令的一样。这样的breakpoint当第一次停下来后，就会被自己删除

k. **rbreak** *regex*: 在所有符合正则表达式*regex*的函数处设置breakpoint

（2）**info breakpoints** [*n*]:查看第*n*个breakpoints的相关信息，如果省略了*n*，则显示所有breakpoints的相关信息

（3）pending breakpoints:是指设置在程序开始调试后加载的动态库中的位置处的breakpoints

a. **set breakpoint pending auto**: GDB缺省设置，询问用户是否要设置pending breakpoint

b. **set breakpoint pending on**: GDB当前不能识别的breakpoint自动成为pending breakpoint

c. **set breakpoint pending off**: GDB当前不能识别某个breakpoint时，直接报错

d. **show breakpoint pending**:查看GDB关于pending breakpoint的设置的行为(auto, on, off)

（4）breakpoints的删除：

a. **clear**:清除当前stack frame中下一条指令之后的所有breakpoints

b. **clear** *function* & **clear** *filename:function*: 清除函数*function*入口处的breakpoints

c. **clear** *linenum* & **clear** *filename:linenum*: 清除第*linenum*行处的breakpoints

d. **delete** [**breakpoints**][*range*…]:删除由*range*指定的范围内的breakpoints，*range*范围是指breakpoint的序列号的范围

（5）breakpoints的禁用、启用:

a. **disable** [**breakpoints**][*range*…]:禁用由*range*指定的范围内的breakpoints

b. **enable** [**breakpoints**][*range*…]:启用由*range*指定的范围内的breakpoints

c. **enable** [**breakpoints**]**once** [*range*…]: 只启用一次由*range*指定的范围内的breakpoints，等程序停下来后，自动设为禁用

d. **enable** [**breakpoints**]**delete** [*range*…]: 启用*range*指定的范围内的breakpoints，等程序停下来后，这些breakpoints自动被删除

（6）条件breakpoints相关命令：

a. 设置条件breakpoints可以通过**break** …**if** *cond*来设置，也可以通过**condition** *bnum expression*来设置，在这里首先要通过（1）中介绍的命令设置好breakpoints，然后用**condition**命令来指定某breakpoint的条件，该breakpoint由*bnum*指定，条件由*expression*指定

b. **condition** *bnum*: 取消第*bnum*个breakpoint的条件

c. **ignore** *bnum count*: 第*bnum*个breakpoint跳过*count*次后开始生效

（7）指定程序在某个breakpoint处停下来后执行一串命令：

a. 格式：**commands** [*bnum*]

​         *… command-list …*

​         **end**

b. 用途：指定程序在第*bnum*个breakpoint处停下来后，执行由*command-list*指定的命令串，如果没有指定*bnum*，则对最后一个breakpoint生效

c. 取消命令列表：**commands** [*bnum*]

​                  **end**

d. 例子：

**break** foo if x>0

**commands**

**silent**

**printf** "x is %d\n",x

**continue**

​       **end**

上面的例子含义：当x>0时，在foo函数处停下来，然后打印出x的值，然后继续运行程序

\2.       Watchpoints相关命令：（Watchpoint的作用是让程序在某个表达式的值发生变化的时候停止运行，达到‘监视’该表达式的目的）

（1）设置watchpoints:

a. **watch** *expr*: 设置写watchpoint，当应用程序写*expr*,修改其值时，程序停止运行

b. **rwatch** *expr*: 设置读watchpoint，当应用程序读表达式*expr*时，程序停止运行

c. **awatch** *expr*: 设置读写watchpoint, 当应用程序读或者写表达式*expr*时，程序都会停止运行

（2）**info watchpoints**:查看当前调试的程序中设置的watchpoints相关信息

（3）watchpoints和breakpoints很相像，都有enable/disabe/delete等操作，使用方法也与breakpoints的类似

\3.       Catchpoints相关命令：（catchpoints的作用是让程序在发生某种事件的时候停止运行，比如C++中发生异常事件，加载动态库事件）

（1）设置catchpoints:

a. **catch** *event*: 当事件*event*发生的时候，程序停止运行，这里event的取值有：

1）**throw**: C++抛出异常

2）**catch**: C++捕捉到异常

3）**exec**: exec被调用

4）**fork**: fork被调用

5）**vfork**: vfork被调用

6）**load**:加载动态库

7）**load** *libname*: 加载名为*libname*的动态库

8）**unload**:卸载动态库

9）**unload** *libname*: 卸载名为*libname*的动态库

10）**syscall** [*args*]:调用系统调用，*args*可以指定系统调用号，或者系统名称

b. **tcatch** *event*: 设置只停一次的catchpoint，第一次生效后，该catchpoint被自动删除

（2）catchpoints和breakpoints很相像，都有enable/disabe/delete等操作，使用方法也与breakpoints的类似

**第三部分常用命令介绍**

\1.         **attach** *process-id*/**detach** 

（1）**attach** *process-id*: 在GDB状态下，开始调试一个正在运行的进程，其进程ID为*process-id*

（2）**detach**:停止调试当前正在调试有进程，与**attach**配对试用

**2.        kill** 

（1）基本功能：杀掉当前GDB正在调试的应用程序所对应的子进程

（2）如果想不退出GDB而对当前正在调试的应用程序重新编译、链接，可以在GDB中执行**kill**杀掉子进程，等编译、链接完后，再重新执行**run**，GDB便可加载新的可执行程序启动调试

\3.         多线程程序调试相关：

（1）**thread** *threadno*：切换当前线程到由*threadno*指定的线程

（2）**info threads**：查看GDB当前调试的程序的各个线程的相关信息

（3）**thread apply** [*threadno*] [**all**]*args*：对指定（或所有）的线程执行由args指定的命令

\4.         多进程程序调试相关(fork/vfork)：

（1）缺省方式：fork/vfork之后，GDB仍然调试父进程，与子进程不相关

（2）**set follow-fork-mode** *mode*：设置GDB行为，*mode*为**parent**时，与缺省情况一样；*mode*为**child**时，fork/vfork之后，GDB进入子进程调试，与父进程不再相关

（3）**show follow-fork-mode**：查看当前GDB多进程跟踪模式的设置

\5.         **step** & **stepi** 

（1）**step** [*count*]:如果没有指定*count*, 则继续执行程序，直到到达与当前源文件不同的源文件中时停止；如果指定了*count*,则重复行上面的过程*count*次

（2）**stepi** [*count*]:如果没有指定*count*, 继续执行下一条机器指令，然后停止；如果指定了*count*，则重复上面的过程*count*次

（3）**step**比较常见的应用场景：在函数*func*被调用的某行代码处设置断点，等程序在断点处停下来后，可以用**step**命令进入该函数的实现中，但前提是该函数编译的时候把调试信息也编译进去了，负责**step**会跳过该函数。

\6.         **next** & **nexti**

（1）**next** [*count*]:如果没有指定*count*, 单步执行下一行程序；如果指定了*count*，单步执行接下来的*count*行程序

（2）**nexti** [*count*]:如果没有指定count, 单步执行下一条指令；如果指定了count,单步执行接下来的count条执行

（3）**stepi**和**nexti**的区别：**nexti**在执行某机器指令时，如果该指令是函数调用，那么程序执行直到该函数调用结束时才停止。

\7.         **continue** [*ignore-count*]: 唤醒程序，继续运行，至到遇到下一个断点，或者程序结束。如果指定*ignore-count*，那么程序在接下来的运行中，忽略*ignore-count*次断点。

\8.         **finish** & **return**

（1）**finish**:继续执行程序，直到当前被调用的函数结束，如果该函数有返回值，把返回值也打印到控制台

（2）**return** [*expression*]:中止当前函数的调用，如果指定了*expression*，把*expresson*值当做当前函数的返回值；如果没有，直接结束当前函数调用

\9.         信号的处理

（1）**info signals** &**info handle**：打印所有的信号相关的信息，以及**GDB**缺省的处理方式

 

（2）**handle** *signal keywords*: 设置**GDB**对具体某个信号的处理方式。*signal*可以为信号整数值，也可以为SIGSEGV这样的符号。keywords的取值有：

a. **stop**和**nostop**:**nostop**表示当**GDB**收到指定的信号，不会应用停止程序的执行，只会打印出一条收到信号的消息，因此，**nostop**也暗含了下面的**print**;而**stop**则表示，当**GDB**收到指定的信号，停止应用程序的执行。

b. **print**和**noprint**:**print**表示如果收到指定的信号，打印出一条信息;**noprint**与**print**表示相反的意思

c. **pass**和**nopass**：**pass**表示如果收到指定的信号，把该信号通知给应用程序;**nopass**表示与pass相反的意思

d. **ignore**和**noignore**:**ignore**与**nopass**同义，同理，**noignore**与**pass**同义

**第四部分调用栈（Call Stack）**

\1.       查看调用栈信息: 

a. **backtrace**:显示程序的调用栈信息，可以用**bt**缩写

b. **backtrace** *n*: 显示程序的调用栈信息，只显示栈顶*n*桢(frame)

c. **backtrace -** *n*:显示程序的调用栈信息，只显示栈底部n桢(frame)

d. **set backtrace limit** *n*: 设置**bt**显示的最大桢层数

e. **where**,**info stack**：都是**bt**的别名，功能一样

\2.       选择某一桢进行查看：

a. **frame** *n*: 查看第*n*桢的信息

b. **frame** *addr*: 查看pc地址为*addr*的桢的相关信息

c. **up***n*: 查看当前桢上面第*n*桢的信息

d. **down***n*: 查看当前桢下面第*n*桢的信息

\3.       frame信息内容：

a. 用**backtrace**、**frame***n*或者**frame***addr*得到的简要信息内容：

（1）桢序号(Frame number)

（2）函数名

（3）Program counter（除非set print address off）（在程序当前执行到的那一桢，PC不会被显示）

（4）源代码文件名和行号

（5）函数的参数名和传入的值

b. 用**info frame**、**info frame***n*或者**info frame***addr*得到的详细的信息内容：

（1）当前桢的地址

（2）下一桢的地址

（3）上一桢的地址

（4）源代码所用的程序的语言(c/c++)

（5）当前桢的参数的地址

（6）当前相中局部变量的地址

（7）PC（program counter）

（8）当前桢中存储的寄存器

\4.       info args：查看当前桢中的参数

\5.       info locals：查看当前桢中的局部变量

\6.       info catch：查看当前桢中的异常处理器（exception handlers）

**第五部分数据和源代码（Data and Source Code）**

\1.       list命令（缩写为l）:

（1）**list***linenum*: 打印当前文件中第*linenum*行周围的源代码

（2）**list***function*: 打印*function*函数周围的源代码

（3）**list**:在上一次使用**list**命令的基础上，再多打印一些源代码

（4）**list -**:打印和上一次使用**list**命令一样的源代码

（5）**set listsize***count*: 设置**list**命令显示源代码的行数

（6）**show listsize**:查看**list**命令显示

\2.       用**edit**命令在GDB模式下编辑源代码：

（1）选择合适的编辑器，gdb会选择/bin/ex做为源代码编辑器，有些linux发行版上可能会没有安装/bin/ex，可以把编辑器修改为比较常见的vim，具体做法为：有启动gdb之前，在命令行执行**export** EDITOR=/usr/bin/vim（或者可以在.profile中设置EDITOR这个变量的值为/usr/bin/vim，这样就不用每次启动gdb的时候都去设置一下了）

（2）**edit**:编辑当前文件

（3）**edit***number*: 编辑当前文件的第*number*行

（4）**edit***function*: 编辑当前文件的*function*函数

（5）**edit***filename*: *number*: 编辑名为*filename*的文件的第*number*行

（6）**edit***filename*: *function*: 编辑名为*filename*的文件的*function*函数

\3.       设置源代码目录

（1）**directory***dirname*: 设置当前调试的程序的源代码目录为*dirname*

（2）**directory**:将当前调试的程序的源代码目录清空

（3）**show directories**:打印当前调试的程序的源代码目录

\4.       **print**命令打印数据：

（1）**print***expr*: 打印表达式*expr*的值

（2）**print /***f expr*:以*f*指定的格式打印表达式*expr*的值

（3）**print**:打印上一次打印的表达式的值

（4）**print /***f*:以*f*指定的形式打印上一次打印的表达式的值

（print支持的格式有：x-16进制整数，d-10进制整数，u-10进制无符号整数，o-8进制整数，t-2进制整数，a-地址形式整数，c-字符常量整数，f-浮点数）

\5.       GDB支持的表达式：

（1）Any kind of constant, variable or operator defined by the programming language you are using is valid in an expression inGDB.

（2）**@***address*:把*address*指定的内存当作数组，语法为**p \*array@len**

（3）*file***::***variable*:指定变量*varialbe*被定义的文件*file*

（4）*function***::***varable*:指定变量*variable*被定义的函数*function*

（5）{*type*}*address*: 把*address*指定的内存解释为*type*类型（类似于强制转型，更加强）

\6.       打印内存：**x** /*nfu addr*:

（1）*n*：重复次数，缺省是1

（2）*f*：打印格式，与print的相同的，还有s-C风格字符串，i-机器指令，缺省是x

（3）*u*：打印单位大小，b-byte，h-halfwords（2字节），w-words（4字节），g-Giant words（8字节）

\7.       自动打印：

（1）**display** /*f expr*|*addr*：以*f*为格式，打印*expr*或者*addr*

（2）**undisplay***dnums*，**delete display** dnums：删除第*dnums*个display点

（3）**disable display***dnums*：禁用第*dnums*个display点

（4）**enable display***dnums*：启用第*dnums*个display点

（5）**info display**：查看所有的display点

\8.       打印选项：

a. **set print***field*: 打开*field*选项

b. **set print***field* **on**: 打开*field*选项

c. **set print***field* **off**: 关闭*field*选项

d. **show print***field*: 查看*field*选项的打开、关闭情况

（1）**set print array**：以一种比较好看的方式打印数组，缺省是关闭的

（2）**set print elements***num-of-elements*：设置GDB打印数据时显示元素的个数，缺省为200，设为0表示不限制(unlimited)

（3）**set print null-stop**：设置GDB打印字符数组的时候，遇到NULL时停止，缺省是关闭的

（4）**set print pretty**：设置GDB打印结构的时候，每行一个成员，并且有相应的缩进，缺省是关闭的

（5）**set print object**：设置GDB打印多态类型的时候，打印实际的类型，缺省为关闭

（6）**set print static-members**：设置GDB打印结构的时候，是否打印static成员，缺省是打开的

（7）**set print vtbl**：以漂亮的方式打印C++的虚函数表，缺省是关闭的

\9.       寄存器：

（1）**info registers**:查看当前桢中的各个寄存器的情况

（2）**info registers***regname*: 查看指定的寄存器

（3）各个寄存器：

 

\10.   内存copy:

（1）**dump** [*format*]**memory** *filename start_addr end_addr*

（2）**append** [**binary**]**memory** *filename start_addr end_addr*

（3）**restore***filename* [**binary**] *bias start end*

\11.   在GDB中定义宏：

（1）**info macro***macro*：查看宏*macro*的定义

（2）**macro define***macro replacement-list*：（还没实现）

（3）**macro define***macro(arglist) replacement-list*：（还没实现）

（4）**macro undef***macro*：（还没实现）

\12.   修改程序的运行(Altering Execution)

（1）修改变量值：

a. **print***v=value*:　修改变量值的同时，把修改后的值显示出来

b. **set** [**var**]*v=value*:　修改变量值，需要注意如果变量名与GDB中某个**set**命令中的关键字一样的话，前面加上**var**关键字

c. **whatis***v*: 查看变量类型

（2）**signal***signal*: 向程序发送信号*signal*,*signal*可以是信号的符号或数字形式，如果*signal*=0，那么程序将会继续运行，程序不会收到任何信号。

（3）**return** [*expression*]:中断函数执行，从当前位置直接返回。（注意：**finish**是把函数运行完，再返回，**return**是直接返回。）

（4）**call***expr*: 在GDB中调用应用程序中的函数

\13.   用户自定义命令：

（1）定义一个命令

**define***commandname*

​        …

​        **end**

（2）条件语句：

**if** *cond-expr*

​     …

​     **else**

​     …

​     **end** 

（4）循环语句：

**while***cond-expr*

​       …

​       **end**

（5）定义一个命令的文档信息，在**help***commandname*的时候可以显示：

**document***commandname*

​        …

​        **end**

（6）$arg0…$arg9：表示命令行参数，最多10个

（7）**help user-defined**：查看所有的用户自定义命令

（8）**show user**commandname：查看自定义命令commandname的定义

（9）**help**commandname：查看自定义命令commandname的帮助信息

（10）**show max-user-call-depth**：查看用户自定义命令的最大递归调用深度

（11）**set max-user-call-depth**：设置用户自定义命令的最大递归调用深度
