---
title: SpringMVC文件上传（前后端分离）
date: 2020-06-20 9:25:00
categories:
- 技术博客
tags:
- springMVC
- 文件上传




---





## 0、分析

**SpringMVC要配置它自带的文件上传功能，我们要配置的便是`Multipart Resolver`。**

**刚开始可能有些疑惑，但是贴上一段来自官方文档的原话，就能大致明白流程了**

```txt
MultipartResolver from the org.springframework.web.multipart package is a strategy for parsing multipart requests including file uploads. There is one implementation based on Commons FileUpload and another based on Servlet 3.0 multipart request parsing.

To enable multipart handling, you need to declare a MultipartResolver bean in your DispatcherServlet Spring configuration with a name of multipartResolver. The DispatcherServlet detects it and applies it to the incoming request. When a POST with content-type of multipart/form-data is received, the resolver parses the content and wraps the current HttpServletRequest as MultipartHttpServletRequest to provide access to resolved parts in addition to exposing them as request parameters.

```

很好，这段话为我们指明了我们所需要配置的方向。

##### ① need to declare a MultipartResolver bean in your DispatcherServlet Spring configuration with a name of multipartResolver. 即我们需要在dispatcher-servlet.xml（文件名为自定义，表示你所配置的dispatcherServlet配置文件）中配置一个bean，且以`multipartResolver`的名字命名这个bean。我们基本都是采用id这个属性为每个bean配置对象名，因此，将id配置为`multipartResolver`。但是我们要配置哪个类的bean对象呢？即class属性指向谁呢？

##### ② 看官方文档，官方为我们提供了两种实现

```txt
1) Apache Commons FileUpload

To use Apache Commons FileUpload, you can configure a bean of type CommonsMultipartResolver with a name of multipartResolver. You also need to have commons-fileupload as a dependency on your classpath.
```

```txt
2) Servlet 3.0

Servlet 3.0 multipart parsing needs to be enabled through Servlet container configuration. To do so:
    In Java, set a MultipartConfigElement on the Servlet registration.
    In web.xml, add a "<multipart-config>" section to the servlet declaration.
```

简单点，我们只介绍` Apache Commons FileUpload`的配置过程。

## 1、依赖配置

- 打开maven项目中的pom.xml。并添加相关依赖

  ```xml
  <dependency>
      <groupId>commons-fileupload</groupId>
      <artifactId>commons-fileupload</artifactId>
      <version>1.4</version>
  </dependency>
  ```

## 2、配置id为multipartResolver的bean对象

- 找到对应的applicationContext.xml（当然你也可以将文件上传下载相关的配置单独提取为一个.xml配置文件）

  ```xml
  <!--    文件上传配置-->
  <bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
      <property name="defaultEncoding" value="UTF-8"/>
      <!--        设置上传文件的大小上限，单位为字节（10485760 Byte = 10 MB）-->
      <!--        可按自身具体业务情况进行配置  -->
      <!--        <property name="maxUploadSize" value="10485760"/>-->
      <!--        <property name="maxInMemorySize" value="40960"/>-->
  </bean>
  ```

- 看到官方文档的 Apache Commons FileUpload配置流程，只有上面那短短的几句话，以为就这样配置完成了，此时测试一下上传（下文将会把上传相关的代码贴出来），发现HTTP错误码500

  ```txt
  HTTP状态 500 - 内部服务器错误
  类型 异常报告
  
  消息 Request processing failed; nested exception is org.springframework.web.multipart.MultipartException: Failed to parse multipart servlet request; nested exception is java.lang.IllegalStateException: 由于没有提供multi-part配置，无法处理parts
  
  描述 服务器遇到一个意外的情况，阻止它完成请求。
  ```

  它这里说`由于没有提供multi-part配置，无法处理parts`，我们回到官方文档，和`Apache Commons FileUpload`一样实现multipart servlet的`Servlet 3.0`中，官方写着这么一段话（上文已经将这段话贴出来了）：

  ```txt
  In web.xml, add a "<multipart-config>" section to the servlet declaration.
  ```

  是不是感觉和刚才的报错很像。于是尝试着在web.xml中添加这个配置，但这要配置到哪呢？



## 3、在web.xml中进行配置 

- 在web.xml中配置springmvc的dispatcherServlet

  ```xml
  <!--    配置springmvc分发器的相关参数以及启动等级-->
  <servlet>
      <servlet-name>springmvc</servlet-name>
      <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
      <init-param>
          <param-name>contextConfigLocation</param-name>
          <param-value>classpath:dispatcher-servlet.xml</param-value>
      </init-param>
      <load-on-startup>1</load-on-startup>
  </servlet>
  <servlet-mapping>
      <servlet-name>springmvc</servlet-name>
      <url-pattern>/</url-pattern>
  </servlet-mapping>
  ```

  

- 配置`<multipart-config>`

  ```xml
  <!--    配置springmvc分发器的相关参数以及启动等级-->
  <servlet>
      <servlet-name>springmvc</servlet-name>
      <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
      <init-param>
          <param-name>contextConfigLocation</param-name>
          <param-value>classpath:dispatcher-servlet.xml</param-value>
      </init-param>
      <load-on-startup>1</load-on-startup>
  
  <!--  配置multipart-config  -->
      <multipart-config>
          <!-- 这里有几个属性可以配置，按照需求进行配置就行了，我这里就记得的配置了一下，单位为byte -->
          <max-file-size>20848820</max-file-size>
          <max-request-size>418018841</max-request-size>
          <file-size-threshold>1048576</file-size-threshold>
      </multipart-config>
  </servlet>
  
  <servlet-mapping>
      <servlet-name>springmvc</servlet-name>
      <url-pattern>/</url-pattern>
  </servlet-mapping>
  ```

- 重新尝试着上传文件，结果是成功的![image-20200620085324970](SpringMVC%E6%96%87%E4%BB%B6%E4%B8%8A%E4%BC%A0%EF%BC%88%E5%89%8D%E5%90%8E%E7%AB%AF%E5%88%86%E7%A6%BB%EF%BC%89/image-20200620085324970.png)

  ![image-20200620085438051](SpringMVC%E6%96%87%E4%BB%B6%E4%B8%8A%E4%BC%A0%EF%BC%88%E5%89%8D%E5%90%8E%E7%AB%AF%E5%88%86%E7%A6%BB%EF%BC%89/image-20200620085438051.png)

---

配置是配置完了，但是代码怎么写呢？



## 4、前端代码

为了简洁，降低复杂度，这里就直接省略css样式好了（才不是作者懒得写呢，哼）

```html
<form action="/uploadAvatar" method="post" id="head-avatar"  enctype="multipart/form-data">

    上传：<input type="file" name="file"><br/>
    <input type="submit" value="上传文件">
    
</form>
```

当然地，如果你需要上传图片到服务器，比如上传头像到服务器进行存储，也可以在`<form>`表单中指定所上传的文件类型为图片

```html
<form action="/uploadAvatar" method="post" id="head-avatar"  enctype="multipart/form-data">
    
    <!--上传：<input type="file" name="file"><br/>-->
    头像：<input type="file" name="avatar" accept="image/*"><br/>
    <input type="submit" value="上传头像">
    
</form>
```



## 5、后端代码

```java
@Controller
public class UploadFileController {
    
    @PostMapping("/uploadAvatar")
    @ResponseBody
    public String saveAvatar(@RequestParam("avatar") MultipartFile file, HttpServletRequest request) {
        //简单点的话，这里可以直接返回前端，文件本身的文件名
        String uploadFileName = file.getOriginalFilename();
        return uploadFileName;
    }
}
```

**注意：** `@RequestParam("avatar")`中的avatar需要与前端代码中`<input type="file" name="avatar" accept="image/*">`标签中的name属性一致。否则SpringMVC无法找到对应相关的参数。



- 完整一点，如何将  图片/文件  保存到服务器呢？

```java
@Controller
public class UploadFileController {
    
    @PostMapping("/uploadAvatar")
    @ResponseBody
    public String saveAvatar(@RequestParam("avatar") MultipartFile file, HttpServletRequest request) throws IOException {
        //这些上传的头像图片都是保存在同一个文件夹下
        //因此为了极大减少前后上传的两个文件重命名，所采用的方案是，在文件名前增加一段UUID来极大降低命名冲突的可能性
        String uuid = UUID.randomUUID().toString().replace("-", "");
        String uploadFileName = uuid+file.getOriginalFilename();

        if ("".equals(uploadFileName)) return "fail to upload avatar!";

        System.out.println("所上传的文件名："+uploadFileName);

//        上传路径
        String path = request.getSession().getServletContext().getRealPath("/WEB-INF/static/img/avatar");
//        如果路径不存在，就创建一个
        File readPath = new File(path);
        if (!readPath.exists()) readPath.mkdirs();

        System.out.println("上传文件的保存地址："+readPath);
        
//		  第一种方式：使用io流将所上传的文件保存到服务器的磁盘中
//        InputStream is = file.getInputStream();
//        OutputStream os = new FileOutputStream(new File(readPath, uploadFileName));
//
////        读取写出
//        int len = 0;
//        byte[] buffer = new byte[1024];
//        while(  (len=is.read(buffer))!=-1  ){
//            os.write(buffer, 0, len);
//            os.flush();
//        }
//
//        os.close();
//        is.close();

//        第二种方式，通过MultipartFile 中提供的transferTo方法，直接传入完整的路径名即可
        file.transferTo(new File(readPath + "/" + uploadFileName));

        return uploadFileName;
    }
}
```





