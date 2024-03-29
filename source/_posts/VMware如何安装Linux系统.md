---
title: VMware如何安装Linux系统
date: 2020-09-16 19:01:40
categories:
- 技术博客
tags:
- linux

---



# 1、镜像下载

- 作者推荐几个目前个人正在使用的linux系统镜像的相关站点。
  - 清华大学开源软件镜像站	mirrors.tuna.tsinghua
  - Linux镜像下载站点 mirrors.ustc.edu.cn
  - CentOS中文站  https://www.centoschina.cn/ 



1. 例如我们从`Linux镜像下载站点`这个站点下载我们需要的系统镜像

![image-20200916144646117](VMware%E5%A6%82%E4%BD%95%E5%AE%89%E8%A3%85Linux%E7%B3%BB%E7%BB%9F/image-20200916144646117.png)



2. 下载CentOS系统

![image-20200916144923810](VMware%E5%A6%82%E4%BD%95%E5%AE%89%E8%A3%85Linux%E7%B3%BB%E7%BB%9F/image-20200916144923810.png)

3. 点击获取ISO镜像

![image-20200916145705477](VMware%E5%A6%82%E4%BD%95%E5%AE%89%E8%A3%85Linux%E7%B3%BB%E7%BB%9F/image-20200916145705477.png)

4. 随后我们得到了一个下载后的`.iso`文件，并将其放在一个统一的文件夹下进行管理

![image-20200916151615295](VMware%E5%A6%82%E4%BD%95%E5%AE%89%E8%A3%85Linux%E7%B3%BB%E7%BB%9F/image-20200916151615295.png)

# 2、新建CentOS 8虚拟机

1. 新建虚拟机

![image-20200916151256035](VMware%E5%A6%82%E4%BD%95%E5%AE%89%E8%A3%85Linux%E7%B3%BB%E7%BB%9F/image-20200916151256035.png)

2. 为了简单面向新手，过复杂的定制化便不宜过多介绍，有兴趣的小伙伴可以自行了解。因此，选择`典型安装`，点击`下一步`。

![image-20200916151851512](VMware%E5%A6%82%E4%BD%95%E5%AE%89%E8%A3%85Linux%E7%B3%BB%E7%BB%9F/image-20200916151851512.png)

3. 点击`稍后安装操作系统`后，点击`下一步`。

![image-20200916152244639](VMware%E5%A6%82%E4%BD%95%E5%AE%89%E8%A3%85Linux%E7%B3%BB%E7%BB%9F/image-20200916152244639.png)

4. 因为我们安装的是Linux发行版中的CentOS，因此选择操作系统为Linux，随后将版本选择为CentOS 8 64位（如果你下载的.iso文件是CentOS 7版本， 那么这里的版本选择应当选择CentOS 7）。随后点击`下一步`。

![image-20200916152606750](VMware%E5%A6%82%E4%BD%95%E5%AE%89%E8%A3%85Linux%E7%B3%BB%E7%BB%9F/image-20200916152606750.png)

5. 为虚拟机命名。同时将此次新建的虚拟机保存在你预备好的安装路径，当然你也可以选择默认的路径，只是通常我们不建议安装在系统盘中。选择完毕后，点击`下一步`。

![image-20200916152942639](VMware%E5%A6%82%E4%BD%95%E5%AE%89%E8%A3%85Linux%E7%B3%BB%E7%BB%9F/image-20200916152942639.png)



6. 决定我们所需要的虚拟磁盘的大小，这里为了方便演示，选择了默认的20GB，用户可根据自己的实际情况选择自己需要的磁盘大小，正常来说推荐使用40GB即可。同时作者已经有规划地将所有的虚拟机都放在了K盘，并决定不会轻易转移虚拟机到其他磁盘下，因此，选择了`将虚拟磁盘存储为单个文件`。选择完成后点击`下一步`。

![image-20200916153505878](VMware%E5%A6%82%E4%BD%95%E5%AE%89%E8%A3%85Linux%E7%B3%BB%E7%BB%9F/image-20200916153505878.png)

7. 点击完成。

![image-20200916153652858](VMware%E5%A6%82%E4%BD%95%E5%AE%89%E8%A3%85Linux%E7%B3%BB%E7%BB%9F/image-20200916153652858.png)



8. 一台最基本的虚拟机就完成了。

![image-20200916153752459](VMware%E5%A6%82%E4%BD%95%E5%AE%89%E8%A3%85Linux%E7%B3%BB%E7%BB%9F/image-20200916153752459.png)

# 3、细致化配置CentOS 8



1. 在点击`开启此虚拟机`之前，选择`编辑虚拟机设置`进行虚拟机的细致化配置。例如配置网络、CPU、内存大小、实体机与虚拟机之前的文件共享等。
   - 下面标志有`【必须】`的内容，请务必配置它。反之标志有`【非必须】`的内容仅提供参考，若不想配置它们则按默认即可。

![image-20200916154008985](VMware%E5%A6%82%E4%BD%95%E5%AE%89%E8%A3%85Linux%E7%B3%BB%E7%BB%9F/image-20200916154008985.png)

2. 【必须】镜像配置。前面的过程中，没有配置我们所下载的系统镜像，因此，这一步我们需要配置系统镜像。选择`CD/DVD`后选择`使用ISO映像文件`，点击`浏览`选择我们前面所下载的`.iso`文件，选择完毕后点击`确定`即可。

![image-20200916170820673](VMware%E5%A6%82%E4%BD%95%E5%AE%89%E8%A3%85Linux%E7%B3%BB%E7%BB%9F/image-20200916170820673.png)



3. 【非必须】内存配置。正常来说，若你的Linux系统是用于配置服务器的，那么就推荐使用2GB以上的内存。若用于学习使用或简单使用，那么1GB的内存即可。

![image-20200916154800348](VMware%E5%A6%82%E4%BD%95%E5%AE%89%E8%A3%85Linux%E7%B3%BB%E7%BB%9F/image-20200916154800348.png)

4. 【非必须】CPU配置。若用于个人学习使用或简单使用，那么1核1GB基本够用。但若需要安装各种奇怪的东西或用于部署其他内容，那么推荐使用1核2GB或以上的配置。

![image-20200916160754375](VMware%E5%A6%82%E4%BD%95%E5%AE%89%E8%A3%85Linux%E7%B3%BB%E7%BB%9F/image-20200916160754375.png)

5. 【非必须】多磁盘配置（模拟现实物理机中的多块机械硬盘/SSD）。选择硬盘，然后点击添加，随后检查硬件类型是否为`硬盘`，随后若不懂的内容直接点击下一步按默认的就可以了。直至新硬盘分配的`完成`。【这一步可不配置它，但需要模拟多磁盘环境的情况下 可专门过来配置它】

![image-20200916161048302](VMware%E5%A6%82%E4%BD%95%E5%AE%89%E8%A3%85Linux%E7%B3%BB%E7%BB%9F/image-20200916161048302.png)



6. 【非必须】配置虚拟机的模拟网络的架构模式。
   1. 桥接模式。相当于你的虚拟机变成了一台物理机，并且和你实际的物理机同处在一个子网内。比如你的物理机ip为192.168.0.1，那么虚拟机的ip就有可能变成192.168.0.2。
   2. NAT模式。相当于你的物理机变成了一个路由器，用来转发虚拟机的所有网络包。例如说，如果物理机的ip为10.23.31.40，处在子网A中；虚拟机的ip为192.168.0.1，处在子网B中。那么按照NAT模式，子网B就归属于A网络。
   3. 仅主机模式。是一种比NAT模式更加封闭的网络连接模式，它创建一个专用网络，使得该网络仅主机（物理机）可见。相当于该模式下，物理机和1个或N个虚拟机构成了自己的内部网络世界，即使物理机的网络断开了，这个内部网络世界中的物理机和N台虚拟机之间还是能互相联系的。相对地，外部的因特网除了物理机1台外，谁也看不见那些虚拟机所在的内部网络世界。

![image-20200916165254959](VMware%E5%A6%82%E4%BD%95%E5%AE%89%E8%A3%85Linux%E7%B3%BB%E7%BB%9F/image-20200916165254959.png)

7. 物理机与虚拟机之间的我`文件共享`。点击`虚拟机设置`后选择`选项`，找到`共享文件夹`并启用它，添加上即将共享的文件目录。

![image-20200916165509966](VMware%E5%A6%82%E4%BD%95%E5%AE%89%E8%A3%85Linux%E7%B3%BB%E7%BB%9F/image-20200916165509966.png)



![image-20200916170034550](VMware%E5%A6%82%E4%BD%95%E5%AE%89%E8%A3%85Linux%E7%B3%BB%E7%BB%9F/image-20200916170034550.png)

- 共享文件夹的前置条件搞定了，那么剩下的就是我们需要在虚拟机安装完成之后，手动安装`VMware Tools`即可实现文件夹的共享。



# 4、开始安装CentOS 8

1. 点击`开启此虚拟机`进入首次的系统安装。

![image-20200916170320824](VMware%E5%A6%82%E4%BD%95%E5%AE%89%E8%A3%85Linux%E7%B3%BB%E7%BB%9F/image-20200916170320824.png)

2. 随后我们通过键盘“↑”和“↓”，选择1或2都可以直接安装CentOS系统。区别是，2进行了硬件的检查，因此比较耗时了点。

![image-20200916171134912](VMware%E5%A6%82%E4%BD%95%E5%AE%89%E8%A3%85Linux%E7%B3%BB%E7%BB%9F/image-20200916171134912.png)

3. 进行安装之后，稍等片刻，便会进入一个安装界面。选择系统语言，根据自己的语言偏好选择即可。（这里作者选择默认的系统语言）

![image-20200916171510171](VMware%E5%A6%82%E4%BD%95%E5%AE%89%E8%A3%85Linux%E7%B3%BB%E7%BB%9F/image-20200916171510171.png)

4. 系统安装向导介绍

![image-20200916172126986](VMware%E5%A6%82%E4%BD%95%E5%AE%89%E8%A3%85Linux%E7%B3%BB%E7%BB%9F/image-20200916172126986.png)

- 分区配置。若不懂分区的相关知识，建议直接默认，并不影响正常使用。点击Done。

![image-20200916172229256](VMware%E5%A6%82%E4%BD%95%E5%AE%89%E8%A3%85Linux%E7%B3%BB%E7%BB%9F/image-20200916172229256.png)

- 网络配置。前面已选择了网络连接的模式（桥接、NAT和仅主机模式），这里直接开启网络即可。然后点击Done。

![image-20200916172418290](VMware%E5%A6%82%E4%BD%95%E5%AE%89%E8%A3%85Linux%E7%B3%BB%E7%BB%9F/image-20200916172418290.png)

- 时区和时间设置。选择你所在的时区和时间即可。完成后点击Done。

![image-20200916172634942](VMware%E5%A6%82%E4%BD%95%E5%AE%89%E8%A3%85Linux%E7%B3%BB%E7%BB%9F/image-20200916172634942.png)

- 初始系统软件环境的安装。勾选自己需要的初始化系统环境即可。

![image-20200916173005791](VMware%E5%A6%82%E4%BD%95%E5%AE%89%E8%A3%85Linux%E7%B3%BB%E7%BB%9F/image-20200916173005791.png)



![image-20200916173415912](VMware%E5%A6%82%E4%BD%95%E5%AE%89%E8%A3%85Linux%E7%B3%BB%E7%BB%9F/image-20200916173415912.png)

- 若感觉上面的选择很艰难，也可在左侧选择`Minimal Install`后，在右侧选择`Standard`即可，那么将按照默认进行安装。选择OK之后点击Done。



5. 通过上文中的系统安装向导，一切配置完毕之后，点击`Begin Installation`进行安装。

![image-20200916174034210](VMware%E5%A6%82%E4%BD%95%E5%AE%89%E8%A3%85Linux%E7%B3%BB%E7%BB%9F/image-20200916174034210.png)

6. 在安装的过程中，可配置用户组以及用户。这里作者简单点，配置一个root用户（超级管理员用户）。

![image-20200916174158452](VMware%E5%A6%82%E4%BD%95%E5%AE%89%E8%A3%85Linux%E7%B3%BB%E7%BB%9F/image-20200916174158452.png)

![image-20200916174404960](VMware%E5%A6%82%E4%BD%95%E5%AE%89%E8%A3%85Linux%E7%B3%BB%E7%BB%9F/image-20200916174404960.png)

- 完成之后点击`两次`Done（为什么是两次呢？因为这是系统要求的），等待剩下的安装进度。

7. 安装完成之后，点击Reboot（重启）。

![image-20200916181954382](VMware%E5%A6%82%E4%BD%95%E5%AE%89%E8%A3%85Linux%E7%B3%BB%E7%BB%9F/image-20200916181954382.png)

8. 等各种配置启动之后，成功启动CentOS，键入刚才设置的root用户和密码即可登录。（PS：Linux系统键入密码的时候是默认不显示在窗口上的，因此正常键入密码即可）

![image-20200916182229413](VMware%E5%A6%82%E4%BD%95%E5%AE%89%E8%A3%85Linux%E7%B3%BB%E7%BB%9F/image-20200916182229413.png)

![image-20200916182252737](VMware%E5%A6%82%E4%BD%95%E5%AE%89%E8%A3%85Linux%E7%B3%BB%E7%BB%9F/image-20200916182252737.png)

![image-20200916182543976](VMware%E5%A6%82%E4%BD%95%E5%AE%89%E8%A3%85Linux%E7%B3%BB%E7%BB%9F/image-20200916182543976.png)





