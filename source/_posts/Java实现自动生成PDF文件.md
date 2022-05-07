---
title: Java实现自动生成PDF文件
date: 2020-07-25 20:00:00
categories:
- 技术博客

tags:
- Java
- Pdf

---



# 一、第三方库文件

Java本身是没有能够自动生成PDF文件的API的，因此我们需要导入第三方的库文件。

这里要介绍的第三方jar包是`itext`，这里提供两个文件的下载：

- [itext-asian-5.2.0.jar](/download/itext-asian-5.2.0.jar)

- [itextpdf-5.5.10.jar](/download/itextpdf-5.5.10.jar)

# 二、导入到Maven

**由于Maven仓库中没有提供两个文件的依赖导入方式，因此需要我们将其作为自定义的jar包导入到本地的maven仓库中。当然，如果你并没有使用`依赖管理工具`来管理你的依赖包，那可以直接导入到你的项目中。**



1. 我们需要执行指令 `mvn install`。当然，这个命令有几个参数需要介绍一下

2. -Dfile=【文件路径】
3. -DgroupId=【包的分组ID】
4. -DartifactId=【包名】
5. -Dversion=【包的版本号】
6. -Dpackaging=【打包方式】

因此上面的参数加起来，我们需要执行

```sh
mvn install:install-file -Dfile=【文件路径】 -DgroupId=【包的分组ID】 -DartifactId=【包名】 -Dversion=【包的版本号】 -Dpackaging=【打包方式】
```

所以我们可以直接执行以下两条命令分别将两个包导入到本地的maven仓库：

```sh
$ mvn install:install-file -Dfile=./itextpdf-5.5.10.jar -DgroupId=com.itext -DartifactId=itext -Dversion=5.5.10 -Dpackaging=jar
```

```sh
$ mvn install:install-file -Dfile=./itext-asian-5.2.0.jar -DgroupId=com.itext -DartifactId=itext-asian -Dversion=5.2.0 -Dpackaging=jar
```

**两条命令的执行结果：**

![image-20200725192121783](Java%E5%AE%9E%E7%8E%B0%E8%87%AA%E5%8A%A8%E7%94%9F%E6%88%90PDF%E6%96%87%E4%BB%B6/image-20200725192121783.png)

之后我们就可以通过添加maven依赖的方式来引入这两个文件了：

```xml
    <dependencies>
<!--        导入导出pdf-->
        <dependency>
            <groupId>com.itext</groupId>
            <artifactId>itext</artifactId>
            <version>5.5.10</version>
        </dependency>
        <dependency>
            <groupId>com.itext</groupId>
            <artifactId>itext-asian</artifactId>
            <version>5.2.0</version>
        </dependency>
    </dependencies>
```





# 三、准备工具

1. 我们得先准备一个具有制作`表单`功能的PDF阅读工具（作者这里使用的工具是`Adobe Acrobat Pro DC`）
2. 制作模板

**PDF模板的制作：**

![image-20200725193342040](Java%E5%AE%9E%E7%8E%B0%E8%87%AA%E5%8A%A8%E7%94%9F%E6%88%90PDF%E6%96%87%E4%BB%B6/image-20200725193342040.png)





先点击`准备表单`,然后选择需要扫描的文件后点击开始

![image-20200725193600137](Java%E5%AE%9E%E7%8E%B0%E8%87%AA%E5%8A%A8%E7%94%9F%E6%88%90PDF%E6%96%87%E4%BB%B6/image-20200725193600137.png)



![image-20200725194038613](Java%E5%AE%9E%E7%8E%B0%E8%87%AA%E5%8A%A8%E7%94%9F%E6%88%90PDF%E6%96%87%E4%BB%B6/image-20200725194038613.png)

**表单的每个位置都可以自定义其控件，比如命名控件名为`fill_76`之类的，鼠标右键可打开`属性`进行进一步的编辑，这些文本框控件就是为了之后能让程序自动导入数据，然后生成pdf用的。**

这里就什么也不更改了，直接另存为一个新的文件。



**另存完这个文件之后，我们的PDF模板文件就搞定了**。



# 四、Java自动根据模板生成pdf文件



不多说了，直接入主题：

```java
package top.bingcu.common.utils.file;

import java.io.ByteArrayOutputStream;
import java.io.File;
import java.io.FileOutputStream;
import java.io.IOException;
import java.util.HashMap;
import java.util.Map;


import com.itextpdf.text.Document;
import com.itextpdf.text.DocumentException;
import com.itextpdf.text.pdf.*;

public class PdfUtil {
    //重载方法
    public void fillTemplate(Map<String,Object> datasMap,File outFilePath,File templatePath) throws IOException, DocumentException {
        this.fillTemplate(datasMap, outFilePath.getAbsolutePath(), templatePath.getAbsolutePath());
    }

    /**
     * 填充pdf模板内容，生成一个新的pdf文件
     *
     * @param datasMap 写入的数据
	 * @param templatePath pdf模板路径
     * @param outFilePath 被生成的pdf文件的路径
     */
    public  void fillTemplate(Map<String,Object> datasMap,String templatePath,String outFilePath) throws IOException, DocumentException {
        PdfReader reader = new PdfReader(templatePath);// 读取pdf模板
        ByteArrayOutputStream bos = new ByteArrayOutputStream();
        PdfStamper stamper = new PdfStamper(reader, bos);
        FileOutputStream out = new FileOutputStream(outFilePath);
        
        //获取所有可编辑内容的控件
        AcroFields form = stamper.getAcroFields();
        //获得pdf容器，准备添加图片，图片可通过datasMap传进来
        //由于图片域处于pdf模板的第二页，因此传入参数2
        PdfContentByte underContent = stamper.getUnderContent(2);
        
        //为解决字体库中有些中文字体不存在而不显示的问题，这里直接从某个路径中读取本地的msyh.ttf（微软雅黑字体）进行字体设置
        File fontPath = new File(new File("").getAbsoluteFile() + "/" + "pdfs/font/msyh.ttf");
        BaseFont baseFont = BaseFont.createFont(fontPath.getAbsolutePath(), BaseFont.IDENTITY_H, BaseFont.EMBEDDED);
        form.addSubstitutionFont(baseFont);

        java.util.Iterator<String> it = form.getFields().keySet().iterator();
        while (it.hasNext()) {
            String name = it.next().toString();
            String value = datasMap.get(name)!=null?datasMap.get(name).toString():null;
            
            //输出日志
            System.out.println("[PDF] 已填充内容："+"\t\t\t类型："+form.getFieldType(name)+"\t\t\t"+name+"\t\t\t"+"文本值："+value);

            //这里的getFieldType其实还有其他类型，由于上面的pdf模板只用到了复选框、文本框和签名域，因此就只列出这三个。
            //如果想知道其他的内容，可以追踪到它里面的源码里看详情。
            switch (form.getFieldType(name)){
                case 2: 
                    //复选框，作者看过它的源码，发现如果第三个参数如果不设置它，而是像文本宽那样调用它，那复选框勾选的默认样式将不是打钩，而是打叉
                    form.setField(name, value, true);
                    break;
                case 4: 
                    //文本框
                    form.setField(name,value);
                    break;
                case 7:         
                    //signature图片域
                    
                    //从datasMap中获取图片
                    Image signature = (Image) datasMap.get(name);
                    //设置图片大小
                    signature.scaleToFit(100,120);
                    //由于pdf模板中设置了两个图片域，因此会有两个图片准备填装进去，所以分别设置两张图片的位置
                    if (name.equals("signature1")){
                        signature.setAbsolutePosition(200, 170);
                    } else if (name.equals("signature2")) {
                        signature.setAbsolutePosition(460, 170);
                    } else {
                         break;
                    }

                    underContent.addImage(signature);
                    break;
            }
        }
        // 如果为false那么生成的PDF文件还能编辑，所以必须要设为true
        stamper.setFormFlattening(true);
        stamper.close();

        /*
         *复制到新文件中进行保存：
         */
        Document doc = new Document();
        PdfCopy copy = new PdfCopy(doc, out);
        doc.open();
        //由于这里的PDF模板有两页，所以将两页都复制到新的文件中。
        PdfImportedPage importPage1 = copy.getImportedPage(new PdfReader(bos.toByteArray()), 1);
        PdfImportedPage importPage2 = copy.getImportedPage(new PdfReader(bos.toByteArray()), 2);
        copy.addPage(importPage1);
        copy.addPage(importPage2);
        doc.close();
    }
}
```





```java
public static void main(String[] args) throws Exception{
    PdfUtil pdfUtil = new PdfUtil();

    Map<String, Object> datasMap = new HashMap<>();
    //根据文本框的属性名设置文本框内的内容
    datasMap.put("fill_1", "TestData01");
    datasMap.put("fill_2", "测试数据02");
    datasMap.put("fill_3", "测试数据03");
    datasMap.put("fill_4", "测试数据04");
    //将复选框设置为“On”状态，即打钩
    //form.setField(name, value, true)中，已经将第三个参数设置为true，所以打钩，否则默认打叉
    //这个“On”字符串是可以在制作模板的时候，在控件的属性中自行设置的
    datasMap.put("toggle_1", "On");
    //datasMap.put("Text1", "□");		//骚操作

    //获取当前项目文件夹的绝对路径
    File prePath = new File("");
    File templatePath = new File(prePath.getAbsolutePath() + "/pdfs/template/pdfTemplate.pdf");
    File outFile = new File(prePath.getAbsolutePath()+"/pdfs/generate/new.pdf");

    if (outFile.exists()){
        //日志输出
        System.out.println("[PDF] 文件已存在！"+"("+ outFile.getAbsoluteFile()+")");

        //允不允许程序覆盖已存在的文件，根据自己的需求即可
        outFile.delete();
        //return;
    }

    //调用刚才手写的工具类
    pdfUtil.fillTemplate(datasMap, outFile.getAbsolutePath(), templatePath.getAbsolutePath());
}
```



**全部OK！看一下运行结果吧**

![image-20200725201709832](Java%E5%AE%9E%E7%8E%B0%E8%87%AA%E5%8A%A8%E7%94%9F%E6%88%90PDF%E6%96%87%E4%BB%B6/image-20200725201709832.png)





![image-20200725202158180](Java%E5%AE%9E%E7%8E%B0%E8%87%AA%E5%8A%A8%E7%94%9F%E6%88%90PDF%E6%96%87%E4%BB%B6/image-20200725202158180.png)