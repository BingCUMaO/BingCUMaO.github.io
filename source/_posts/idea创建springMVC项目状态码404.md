---
title: idea创建springMVC项目状态码404
date: 2020-05-30 14:13:40
categories:
- Errors集合
tags:
- springMVC
- 404
- idea


---





# 今天搞了一整天，刚用springMVC写了一个HelloWorld，结果web项目一直报404

#### 搞了一天，心态有点炸。明明Maven添加了所有依赖，但网页访问一直404，最后发现项目结构的生成中没有相关的jar包依赖。最后网上查了一下，找到了解决方案
### 解决方案：
##### 1、找到Project Structure->Artifacts
##### (如果项目中含Module的话，选择对应的module)

##### 2、自己手动创建lib文件夹
![在这里插入图片描述](idea%E5%88%9B%E5%BB%BAspringMVC%E9%A1%B9%E7%9B%AE%E7%8A%B6%E6%80%81%E7%A0%81404/20200520131138821.png)
##### 3、将自己maven中所用到的依赖jar包都添加进lib文件夹
 ![在这里插入图片描述](idea%E5%88%9B%E5%BB%BAspringMVC%E9%A1%B9%E7%9B%AE%E7%8A%B6%E6%80%81%E7%A0%81404/2020052013143180.png)![在这里插入图片描述](idea%E5%88%9B%E5%BB%BAspringMVC%E9%A1%B9%E7%9B%AE%E7%8A%B6%E6%80%81%E7%A0%81404/20200520131703168.png)