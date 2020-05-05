# Welcome to the mykernel 2.0

Develop your own OS kernel by reusing Linux infrastructure, based on x86-64/Linux Kernel 5.4.34.

[mykernel 1.0](https://github.com/mengning/mykernel/tree/cc6f687daaa831a350f3022853825ebe8d78aa2f) based on IA32/Linux Kernel 3.9.4.

## Set up mykernel 2.0 in Ubuntu 18.04

```
wget https://raw.github.com/mengning/mykernel/master/mykernel-2.0_for_linux-5.4.34.patch
sudo apt install axel
axel -n 20 https://mirrors.edge.kernel.org/pub/linux/kernel/v5.x/linux-5.4.34.tar.xz
xz -d linux-5.4.34.tar.xz
tar -xvf linux-5.4.34.tar
cd linux-5.4.34
patch -p1 < ../mykernel-2.0_for_linux-5.4.34.patch
sudo apt install build-essential libncurses-dev bison flex libssl-dev libelf-dev
make defconfig # Default configuration is based on 'x86_64_defconfig'
# 使用allnoconfig编译出来qemu无法加载启动，不知道为什么？有明白的告诉我，完整编译太慢了，消耗的资源也多。
make -j$(nproc) # 编译的时间比较久哦
sudo apt install qemu # install QEMU
qemu-system-x86_64 -kernel arch/x86/boot/bzImage
```
从qemu窗口中您可以看到my_start_kernel在执行，同时my_timer_handler时钟中断处理程序周期性执行。

进入mykernel目录您可以看到qemu窗口输出的内容的代码mymain.c和myinterrupt.c。当前有一个CPU执行C代码的上下文环境，同时具有中断处理程序的上下文环境，我们通过Linux内核代码模拟了一个具有时钟中断和C代码执行环境的硬件平台。

您只要在mymain.c基础上继续写进程描述PCB和进程链表管理等代码，在myinterrupt.c的基础上完成进程切换代码，一个可运行的小OS kernel就完成了。start to write your own OS kernel, enjoy it!

+ mykernel-2.0 patch generated by this command: 
    + diff -Naur linux-5.4.34 linux-5.4.34-mykernel > mykernel-2.0_for_linux-5.3.34.patch

## your own OS kernel example code

* mypcb.h、mymain.c和myinterrupt.c实现了一个简单的时间片轮转调度进程的精简内核，如下为进程上下文切换的关键代码:
```
    	printk(KERN_NOTICE ">>>switch %d to %d<<<\n",prev->pid,next->pid);  
    	/* switch to next process */
    	asm volatile(	
        	"pushq %%rbp\n\t" 	    /* save rbp of prev */
        	"movq %%rsp,%0\n\t" 	/* save rsp of prev */
        	"movq %2,%%rsp\n\t"     /* restore  rsp of next */
        	"movq $1f,%1\n\t"       /* save rip of prev */	
        	"pushq %3\n\t" 
        	"ret\n\t" 	            /* restore  rip of next */
        	"1:\t"                  /* next process start here */
        	"popq %%rbp\n\t"
        	: "=m" (prev->thread.sp),"=m" (prev->thread.ip)
        	: "m" (next->thread.sp),"m" (next->thread.ip)
    	); 
```

## Comments

* mykernel这样一个短小精悍的模拟内核，时常给我提供了看问题的角度和思路。当被庞杂的Linux内核代码弄得一头雾水时，我就去看看mykernel，很多复杂的问题就可以用简单的机制解释。——[pianogirl](http://blog.csdn.net/pianogirl123/article/details/51287024)

# 采用mykernel的课程和项目

* [庖丁解牛Linux内核MOOC课程](https://mooc.study.163.com/course/1000072000?_trace_c_p_k2_=12d5497350df49e2a6e3878d1a5aa5ae&share=2&shareId=1000001002#/info) - 国家精品在线开放课程
* [kernel-in-kernel](https://github.com/jserv/kernel-in-kernel)
* [操作系统导论](https://github.com/jserv/linuxkernel)
* [《庖丁解牛Linux内核分析》](https://j.youzan.com/pfzVI9)
* 微信公众号互动二维码
<img src="https://user-images.githubusercontent.com/609053/81026703-8c25df00-8ead-11ea-8254-29830c3e1146.png" alt="微信公众号互动二维码" width="100" align="bottom" />
