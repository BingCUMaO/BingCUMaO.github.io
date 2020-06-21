---
title: spring实现文件下载
date: 2020-06-21 9:50:00
categories:
- springMVC
tags:
- springMVC
- spring
- 文件下载


---





## 1、假如我们将架构分为dao层、service层和controller层。

- 此时我们将文件保存到服务器的磁盘中存储，我们并不需要对数据库进行操作，只需要在磁盘上进行io操作。因此就没有了dao层的事情了。
- 实现service层，我们需要封装service层的方法，同时spring考虑的很周到，为我们提供了一个专门用来文件下载的类：`org.springframework.http.ResponseEntity`。
- 我们目前需要做的就是，通过这个现成的类，来对其再封装，封装成我们需要的样子。
  - 附带springframework的api文档：`https://docs.spring.io/spring/docs/5.3.0-SNAPSHOT/javadoc-api/`

## 2、上代码（service层）

- service层的接口

```java
package top.BinGCU.service;

import org.springframework.http.ResponseEntity;
import java.io.IOException;

public interface ResourcesParsingService {

     /**
     * 该方法用来获取头像的响应实体
     *
     * @param downloadFileName 客户端下载文件时的文件名
     * @param realPath 文件在服务器磁盘中的全路径
     * @return 返回可供客户端解析的响应实体
     * @throws IOException
     */
    ResponseEntity<byte[]> obtainAvatarReponseEntity(String downloadFileName, String realPath ) throws IOException;
    
}
```

- service层的实现

```java
package top.BinGCU.service.Impl;

import org.apache.commons.io.FileUtils;
import org.springframework.context.ApplicationContext;
import org.springframework.http.HttpHeaders;
import org.springframework.http.HttpStatus;
import org.springframework.http.MediaType;
import org.springframework.http.ResponseEntity;
import top.BinGCU.Util.ApplicationContextShortcut;
import top.BinGCU.dao.UserMapper;
import top.BinGCU.service.ResourcesParsingService;

import java.io.File;
import java.io.IOException;

public class ResourcesParsingServiceImpl implements ResourcesParsingService {
    
    @Override
    public ResponseEntity<byte[]> obtainAvatarReponseEntity(String downloadFileName, String realPath ) throws IOException {
        File file = new File(realPath);
        System.out.println("Check file real path： "+file);

        //设置HttpHeaders，使浏览器解析响应头后响应下载
        HttpHeaders headers = new HttpHeaders();
        //attachment 口令告诉了浏览器你需要进行下载动作
        headers.setContentDispositionFormData("attachment", downloadFileName);
        //设置响应方式为二进制
        headers.setContentType(MediaType.APPLICATION_OCTET_STREAM);

        return new ResponseEntity<byte[]>(FileUtils.readFileToByteArray(file), headers, HttpStatus.CREATED);
    }

}
```



## 3、上代码（controller层）

比如我这里的FileLoadController中的其中一个url就是用来下载用户的头像图片：

```java
package top.BinGCU.controller;

import org.springframework.http.ResponseEntity;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;
import top.BinGCU.service.Impl.ResourcesParsingServiceImpl;
import top.BinGCU.service.ResourcesParsingService;

import javax.servlet.http.HttpServletRequest;
import java.io.IOException;

@Controller
public class FileLoadController {
    private static String sessionName = "userAccountStr";
    private ResourcesParsingService resourcesParsingService = new ResourcesParsingServiceImpl();

    @RequestMapping("/fileLoader/avatar/download")
    @ResponseBody
    public ResponseEntity<byte[]> avatarDownload(HttpServletRequest request) throws IOException {
        //1、根据session.getAttribute，获取userAccount
        String userAccountStr = (String) request.getSession().getAttribute(sessionName);
        //2、根据userAccount 从数据库中查找avatarPath
        //(由于这里的realPath表示全路径，因此要获得downName，需要对其进行分割)
        //(可以采用Regex的方式进行分割)
        //(我这里直接保留和request的请求路径相比多出来的那部分，这部分就直接是文件名了)
        String realPath = resourcesParsingService.queryAvatarPathByUserAccountStr(userAccountStr);
        String downName = realPath.substring(
            request.getServletContext().getRealPath("/WEB-INF/static/img/avatar").length(), 
            realPath.length()
        );

        System.out.println("downloadFileName："+downName);

        //3、根据avatarPath 从服务器的磁盘中读取图片文件到缓存中
        //4、想办法将图片文件发送到前端
        //（本来预想第3、4步是需要在controller中实现的，但误打误撞地发现其实在service层就能够简单实现）
        return resourcesParsingService.obtainAvatarReponseEntity(downName, realPath);
    }
}
```

为了简洁，去掉注释和多于没用的信息后的效果：

```java
@Controller
public class FileLoadController {
    private static String sessionName = "userAccountStr";
    private ResourcesParsingService resourcesParsingService = new ResourcesParsingServiceImpl();

    @RequestMapping("/fileLoader/avatar/download")
    @ResponseBody
    public ResponseEntity<byte[]> avatarDownload(HttpServletRequest request) throws IOException {

        String userAccountStr = (String) request.getSession().getAttribute(sessionName);

        String realPath = resourcesParsingService.queryAvatarPathByUserAccountStr(userAccountStr);
        String downName = realPath.substring(
            request.getServletContext().getRealPath("/WEB-INF/static/img/avatar").length(), 
            realPath.length()
        );

        return resourcesParsingService.obtainAvatarReponseEntity(downName, realPath);
    }
}
```

