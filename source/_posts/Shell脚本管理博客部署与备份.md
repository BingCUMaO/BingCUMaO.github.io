---
title: Shell脚本自动化管理博客部署与备份
date: 2020-07-19 18:50:40
categories:
- 技术博客
tags:
- Github
- Gitee
- Shell




---



# 1、背景介绍

- 一开始，由于本博客站点是通过Hexo部署在github上面的，但由于github的服务器属境外服务器，或者因为种种原因，导致访问速度极其的缓慢
- 之前写过一篇关于解决github访问速度慢的问题，有兴趣的可以去看看。
  - 附带链接：[解决github访问慢问题](https://bingcu.gitee.io/2020/05/30/解决github访问慢问题/)
- 由于速度极其的缓慢，因此我此次将github上的博客站点服务，在码云（Gitee）也部署了一份。这里就忽略Coding了，访问速度和github差不多的，所以就不考虑了，gitee在国内还是蛮快的，而且博客也成功部署了上去。部署完成之后的网址是`bingcu.gitee.io`
- 但gitee缺点就是....
- 就是它没法绑定自己的域名！真心的觉得网址有点太长了...如果直接进入`bingcu.gitee.io`这个网址，那速度肯定是相当快的，毕竟服务器在国内，但这个网址不是作者自己的域名诶，终究也还是gitee下的名字，那个`gitee.io`看着就很不顺眼。
  - 图为域名绑定该博客站点时的监控状态（压根就绑定不了自己的域名，诶~）：
  - ![image-20200719165732797](Shell%E8%84%9A%E6%9C%AC%E7%AE%A1%E7%90%86%E5%8D%9A%E5%AE%A2%E9%83%A8%E7%BD%B2%E4%B8%8E%E5%A4%87%E4%BB%BD/image-20200719165732797.png)
- 网上搜索了一下，发现几乎很多地方都说到Gitee Page Pro支持自定义域名，只不过这是要付费的，虽然也有免费一个月，但无奈的是~想想这也不是一种长久的办法：![image-20200719171034692](Shell%E8%84%9A%E6%9C%AC%E7%AE%A1%E7%90%86%E5%8D%9A%E5%AE%A2%E9%83%A8%E7%BD%B2%E4%B8%8E%E5%A4%87%E4%BB%BD/image-20200719171034692.png)
- 自己买个服务器？感觉大材小用，不划算。
- 所以，本来是打算照着上次`解决github访问慢问题`那篇文章，以同样的方式（就是改本地hosts文件）来间接提升访问速度的，但无奈的是，作者发现gitee的Page服务其实坑的狠：![image-20200719175758127](Shell%E8%84%9A%E6%9C%AC%E7%AE%A1%E7%90%86%E5%8D%9A%E5%AE%A2%E9%83%A8%E7%BD%B2%E4%B8%8E%E5%A4%87%E4%BB%BD/image-20200719175758127.png)
- 发现了没有，其实两个网址被域名解析到了同一台服务器上了！因此你无法对其进行更改或绑定操作...即便更改了hosts文件，也无法正常路由到自己博客站点上（作者亲自测试过的！）
- 无奈~个人使用的时候要是追求访问速度快，那只能从`bingcu.gitee.io`进行访问了...



# 2、两个博客站点的管理问题

- 一个是github下的站点，一个是gitee下的站点。
- 一个国外，一个国内
- 一个访问速度慢，一个访问速度快但域名解析有所限制

### 管理这两个站点的同时，还要对文章随时做好备份工作

全部加起来，每次发布一篇新文章，都要进行很多无意义且繁琐的操作，而且还那么浪费时间。



# 3、博客站点管理自动化

### 1）使用Shell脚本对本地进行自动化管理概要

- 使用Shell脚本，只要一条指令或一次双击运行，就可以完美解决之前复杂且繁琐的管理方案，作者将从中解放出自己的双手，来享受人生。
- 不说了，直接上脚本

### 2） 本地自动化管理文件

- 作者本地博客文件的管理，分成了一下几个文件夹：![image-20200719181908832](Shell%E8%84%9A%E6%9C%AC%E7%AE%A1%E7%90%86%E5%8D%9A%E5%AE%A2%E9%83%A8%E7%BD%B2%E4%B8%8E%E5%A4%87%E4%BB%BD/image-20200719181908832.png)
- 即`Hexo-gitee`存储的是gitee站点上的相关内容，`Hexo-github`存储的是github站点上的相关内容。
- 而`Hexo-sh`则是负责管理`Hexo-gitee`和`Hexo-github`的Shell脚本集合。（是的，`Hexo-gitee`和`Hexo-github`是负责搬砖的，而`Hexo-sh`属于管理层）
- 每次新写一篇博客的时候，就得需要对`Hexo-gitee`和`Hexo-github`文件夹里面的内容进行同步（每次都把写好的博客都复制来粘贴去的，每次都要打开文件进入正确的目录，然后后退到另一个正确的目录。这不仅麻烦，还容易操作失误），因此，我们需要一个能够自动复制文件的脚本：

```sh
#!/bin/bash
#auto copy source directory which contains blog '.md' file
#by author bingcu.top


source_post="../Hexo-github/source/_posts"
dest_gitee="../Hexo-gitee/source/"
dest_backup="../Hexo-github/hexo-backup/test_branch/source/"

echo -e "\033[36m [copy] Start...\033[0m"

echo -e "\033[36m [copy] to gitee...\033[0m"
cp -r $source_post $dest_gitee
echo -e "\033[36m [copy] to backup...\033[0m"
cp -r $source_post $dest_backup

echo -e "\033[36m [copy] All Executed Successfully!\033[0m"
```

- ok，本地文件自动化算是解决了，附上运行结果：![image-20200719182855137](Shell%E8%84%9A%E6%9C%AC%E7%AE%A1%E7%90%86%E5%8D%9A%E5%AE%A2%E9%83%A8%E7%BD%B2%E4%B8%8E%E5%A4%87%E4%BB%BD/image-20200719182855137.png)

### 3）自动化部署到github上的博客站点

- 不多说了，直接贴代码（其实就是直接执行了`hexo clean && hexo genearate && hexo deploy`这条命令）

```sh
#!/usr/bin/bash
#auto deploy hexo blog
#by author bingcu.top

#change directory
cd ../Hexo-github/

echo -e "\033[36m [hexo] Start deploy github...\033[0m"

echo -e "\033[36m [hexo] cleanning...\033[0m"
hexo clean
echo -e "\033[36m [hexo] generating...\033[0m"
hexo generate
echo -e "\033[36m [hexo] deploying...\033[0m"
hexo deploy

echo -e "\033[36m [hexo] Github OK.\033[0m"

#restore directory
cd ../Hexo-sh/
```





### 4） 自动化部署到gitee上的博客站点

- 其实和第3点一样的，因为`Hexo-gitee`和`Hexo-github`这两个文件夹里面的东西，除了几个配置文件不一样外，其他文件基本上都一样的，所以部署的操作也一样

```sh
#!/usr/bin/bash
#auto deploy hexo blog
#by author bingcu.top
#change directory

cd ../Hexo-gitee/

echo -e "\033[36m [hexo] Start deploy gitee...\033[0m"

echo -e "\033[36m [hexo] cleanning...\033[0m"
hexo clean
echo -e "\033[36m [hexo] generating...\033[0m"
hexo generate
echo -e "\033[36m [hexo] deploying...\033[0m"
hexo deploy

echo -e "\033[36m [hexo] Gitee OK.\033[0m"

echo -e "\033[36m [hexo] All Executed Successfully \033[0m"

#restore directory
cd ../Hexo-sh/
```

- 由于功能都一样，我们可以将两个站点的自动化部署的脚本，集合为一个脚本
- 即：

```sh
#!/usr/bin/bash
#auto deploy hexo blog
#by author bingcu.top

#change directory
cd ../Hexo-github/

echo -e "\033[36m [hexo] Start deploy github...\033[0m"

echo -e "\033[36m [hexo] cleanning...\033[0m"
hexo clean
echo -e "\033[36m [hexo] generating...\033[0m"
hexo generate
echo -e "\033[36m [hexo] deploying...\033[0m"
hexo deploy

echo -e "\033[36m [hexo] Github OK.\033[0m"



#change directory
cd ../Hexo-gitee/

echo -e "\033[36m [hexo] Start deploy gitee...\033[0m"

echo -e "\033[36m [hexo] cleanning...\033[0m"
hexo clean
echo -e "\033[36m [hexo] generating...\033[0m"
hexo generate
echo -e "\033[36m [hexo] deploying...\033[0m"
hexo deploy

echo -e "\033[36m [hexo] Gitee OK.\033[0m"

echo -e "\033[36m [hexo] All Executed Successfully \033[0m"

#restore directory
cd ../Hexo-sh/
```



### 5）备份文章到远程库中

- 作者的博客备份远程库，放在了github远程站点的同一个repository仓库下的另外一个分支`test_branch`。
- 由于第2点中的本地自动化脚本，自动复制了需要备份的所有文件，因此，直接使用git进行push操作，将需要备份的文件push到远程库就可以了。

```sh
#!/usr/bin/bash
#auto backup blog file which save in github's 'test_branch' branch.
#by author bingcu.top

#change directory
cd ../Hexo-github/hexo-backup/test_branch/

echo -e "\033[36m [git] Start to backup...\033[0m"

echo -e "\033[36m [git] adding...\033[0m"
git status
git add source
echo -e "\033[36m [git] commiting\033[0m"
git status
git commit -m "shell auto backup"
echo -e "\033[36m [git] pushing...\033[0m"
git status
git push
echo -e "\033[36m [git] Push OK...\033[0m"

echo -e "\033[36m [git] All Executed Successfully\033[0m"

#restore directory
cd ../../../Hexo-sh
```



### 6）集成上面的几个功能

- 以上的内容，本着模块化的思维分成了几个内容，因此，本着模块化的思维，将以上内容模块化成3个脚本文件
- 即`copy_posts.sh`、`hexo_deploy.sh`、`github_push.sh`
- 可是作者很懒，懒得每次自动化管理还要去启动3个脚本，于是，本着模块化的思维和作者懒惰的天性，增加了一个脚本（老板角色）来负责调用这3个脚本。

```sh
#!/usr/bin/bash
#auto update BinGCU's blog
#by author bingcu.top

echo -e "\033[35m [BinGCU] ~BinGCU\033[0m"
echo -e "\033[35m [BinGCU] ~Ready~\033[0m"
echo -e "\033[35m [BinGCU] ~Go!\033[0m"
echo -e "\033[35m ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~\033[0m"
echo -e "\033[35m ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~\033[0m"

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

- 贴上目录结构：![image-20200719184750804](Shell%E8%84%9A%E6%9C%AC%E7%AE%A1%E7%90%86%E5%8D%9A%E5%AE%A2%E9%83%A8%E7%BD%B2%E4%B8%8E%E5%A4%87%E4%BB%BD/image-20200719184750804.png)
- 因此每次只要双击一下bingcu.sh这个脚本，就能自动化更新博客和备份博客了。一切皆自动~



