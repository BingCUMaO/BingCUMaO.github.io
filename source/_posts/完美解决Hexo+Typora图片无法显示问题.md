---
title: 完美解决Hexo+Typora图片无法显示问题
date: 2020-07-19 21:50:40
categories:
- Hexo
tags:
- github
- Hexo
- Typora





---



# 1、原因

由于Hexo在对markdown进行渲染的时候，会对图片的位置进行移动，导致我们在Typora中的图片路径与部署到hexo中的图片路径产生差异。

# 2、解决

- 1）设置Typora的偏好设置，将路径设置如下状态：![image-20200719210147162](%E5%AE%8C%E7%BE%8E%E8%A7%A3%E5%86%B3Hexo+Typora%E5%9B%BE%E7%89%87%E6%97%A0%E6%B3%95%E6%98%BE%E7%A4%BA%E9%97%AE%E9%A2%98/image-20200719210147162.png)

这样的话，每次写markdown文件的时候，它就会自动的将文件保存到当前目录下的同名文件夹中，方便我们对图片进行管理。

- 2）按照这种设置，每次Typora中的图片与Hexo上面的图片总是相差一个级别的目录：
  - Typora中的图片路径：
  - ![image-20200719210659170](%E5%AE%8C%E7%BE%8E%E8%A7%A3%E5%86%B3Hexo+Typora%E5%9B%BE%E7%89%87%E6%97%A0%E6%B3%95%E6%98%BE%E7%A4%BA%E9%97%AE%E9%A2%98/image-20200719210659170.png)
  - 需要注意的是，Typora自动隐藏了同名文件夹的位置，所以完整的位置应该是3部分，即`和markdown文件同名的文件夹名字/Inter%20x86%EF%BC%88CISC%EF%BC%89%E6%9E%B6%E6%9E%84%E5%B8%B8%E7%94%A8%E6%8C%87%E4%BB%A4%E9%9B%86%EF%BC%88%E8%8B%B1%E6%96%87%E7%89%88%EF%BC%89/image-20200711221126160.png`![image-20200719210635699](%E5%AE%8C%E7%BE%8E%E8%A7%A3%E5%86%B3Hexo+Typora%E5%9B%BE%E7%89%87%E6%97%A0%E6%B3%95%E6%98%BE%E7%A4%BA%E9%97%AE%E9%A2%98/image-20200719210635699.png)
  - Hexo渲染完成之后页面中的图片路径：
    - ![image-20200719211024135](%E5%AE%8C%E7%BE%8E%E8%A7%A3%E5%86%B3Hexo+Typora%E5%9B%BE%E7%89%87%E6%97%A0%E6%B3%95%E6%98%BE%E7%A4%BA%E9%97%AE%E9%A2%98/image-20200719211024135.png)
  - 看起来貌似路径没有变化，但由于Typora本身隐藏了一个父级的目录位置（Typora能正常显示是因为Typora会自动增加一个级别的目录名然后再去查找图片，而Hexo则是直接根据相对位置去查找）
- 3）网上的很多办法其实都感觉没法解决，作者也试着直接去找了第三方的插件试过，但感觉效果不佳，因此作者决定，直接将放图片的文件夹，放两份图片，一份适配Typora，一份适配Hexo，这样虽然浪费了一点空间，但是至少不用再烦恼了。
- 4）因为每次复制图片时，在本地的Typora写着写着发现图片显示的没问题，然后一部署到Hexo，才发现自己忘了复制两份图片了，所以Hexo无法正常显示，才不得已又要跑一边Hexo的部署流程。为了防止自己写完markdown后再犯了上面的失误，作者决定直接写一个自动化脚本来帮助自己完成图片的复制。

```sh
#!/bin/bash
#auto copy typora's images to it's sub directory
#by author bingcu.top

#copy imgs

IFS=$'\n' 			# 修改默认分隔符
OLDIFS="$IFS"

for dir in $(ls ./)		#遍历所有文件名/文件夹名（可以包含空格）
do
    if [ -d $dir ]; then	#如果是文件夹，那就进去
        cd $dir
        if [ ! -d ./$dir ];then	#如果该文件夹下没有和自己同名的子文件夹，那就创建它
            mkdir $dir
        fi
  
        for file in $(ls ./)	#遍历所有文件名/文件夹名
        do
             if [ -f $file ]; then	#如果是文件，那就复制到刚才创建的文件夹下
                 cp $file $dir
             fi
        done
        cd ../		#恢复当前目录位置到上一级
        
    fi   

done  
```



# 3、拓展

- 作者在之前的一篇文章中介绍了自己编写的一个自动化脚本来帮助自己对整个博客系统进行自动化的管理。有兴趣的小伙伴可以去看看，这里可以将这个图片复制的自动化Shell脚本集成到上一篇文章中介绍的自动化脚本中。这样作者的博客管理，就已经非常自动化了~
  - 附带文章链接：[Shell脚本自动化管理博客部署与备份]([https://bingcu.gitee.io/2020/07/19/Shell%E8%84%9A%E6%9C%AC%E7%AE%A1%E7%90%86%E5%8D%9A%E5%AE%A2%E9%83%A8%E7%BD%B2%E4%B8%8E%E5%A4%87%E4%BB%BD/](https://bingcu.gitee.io/2020/07/19/Shell脚本管理博客部署与备份/))
- 如果需要集成到一起，需要对上面的脚本进行一点更改：

```sh
#!/bin/bash
#auto copy typora's images to it's sub directory
#by author bingcu.top

#change directory
cd ../Hexo-github/source/_posts

#copy imgs
echo -e "\033[36m [copy] Start...\033[0m"

IFS=$'\n' 			# 修改默认分隔符
OLDIFS="$IFS"

for dir in $(ls ./)		#遍历所有文件名/文件夹名（可以包含空格）
do
    if [ -d $dir ]; then	#如果是文件夹，那就进去
        cd $dir
        if [ ! -d ./$dir ];then	#如果该文件夹下没有和自己同名的子文件夹，那就创建它
            mkdir $dir
        fi
  
        for file in $(ls ./)	#遍历所有文件名/文件夹名
        do
             if [ -f $file ]; then	#如果是文件，那就复制到刚才创建的文件夹下
                 cp $file $dir
	 echo -e "\033[36m [copy]--   $file\033[0m"
             fi
        done
        cd ../		#恢复当前目录位置到上一级
        
    fi   

done  

echo -e "\033[36m [copy] All Executed Successfully!\033[0m"

#restore directory
cd ../../../Hexo-sh
```

- 同时在`bingcu.sh`增加上面脚本的调用：

```sh
#!/usr/bin/bash
#auto update BinGCU's blog
#by author bingcu.top

echo -e "\033[35m [BinGCU] ~BinGCU\033[0m"
echo -e "\033[35m [BinGCU] ~Ready~\033[0m"
echo -e "\033[35m [BinGCU] ~Go!\033[0m"
echo -e "\033[35m ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~\033[0m"
echo -e "\033[35m ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~\033[0m"

source ./copy_typora_imgs.sh	#增加图片复制的脚本调用
source ./copy_posts.sh
source ./hexo_deploy.sh
source ./github_push.sh

echo -e "\033[35m ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~\033[0m"
echo -e "\033[35m ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~\033[0m"
echo -e "\033[35m [BinGCU] ~All Executed Successfully!\033[0m"
echo -e "\033[35m [BinGCU] ~Bye~\033[0m"



echo "Press any key to exit"
read
```

