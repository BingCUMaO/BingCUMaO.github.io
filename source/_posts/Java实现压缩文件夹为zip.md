---
title: Java实现自动生成PDF文件
date: 2020-08-04 20:00:00
categories:
- Java

tags:
- Java
- zip



---





# 一、

```java
/**
 * 压缩sourceDirectory文件夹下的所有文件
 *
 * @param sourceDirectory 准备压缩的文件夹
 * @param zipDirectoryPath 压缩包所在的目录
 * @param zipName 文件名，包括.zip的后缀名
 * @throws IOException
 */
public void folderToZip(String sourceDirectory, String zipDirectoryPath, String zipName) {
    File src = new File(sourceDirectory);
    ZipOutputStream zos = null;

    if (src == null  || !src.exists() || !src.isDirectory()) {
        // 源目录不存在 或不是目录 , 则异常
        throw new RuntimeException("压缩源目录不存在或非目录!" + sourceDirectory);
    }

    File destdir = new File(zipDirectoryPath);
    if (!destdir.exists()) {
        // 创建目录
        destdir.mkdirs();
    }
    File zipfile = new File(new File(zipDirectoryPath).getAbsolutePath() + "/" + zipName);
    File[] srclist = src.listFiles();

    if (srclist == null || srclist.length == 0) {
        // 源目录内容为空,无需压缩
        throw new RuntimeException("源目录内容为空,无需压缩下载!" + sourceDirectory);
    }

    try {
        zos = new ZipOutputStream(new BufferedOutputStream(new FileOutputStream(zipfile)));
        // 递归压缩目录下所有的文件  ;
        compress(zos, src, src.getName());
        zos.close();

    } catch (FileNotFoundException e) {
        throw new RuntimeException("压缩目标文件不存在!" + e.getMessage());
    } catch (IOException e) {
        throw new RuntimeException("压缩文件IO异常!" + e.getMessage());
    }
    finally {
        if (zos != null) {
            try {
                zos.close();
            } catch (Exception e) {
                // TODO: handle exception
            }
        }
    }

}
```





```java
/**
 * 用于给folderToZip()方法进行递归压缩文件用的
 * @param zos zip的输出流
 * @param src 源文件
 * @param name 文件名
 * @throws IOException
 */
private void compress(ZipOutputStream zos, File src, String name) throws IOException {
    if (src == null || !src.exists()) {
        return ;
    }
    if (src.isFile()) {
        byte[] bufs = new byte[10240];

        ZipEntry zentry = new ZipEntry(name);
        zos.putNextEntry(zentry);

        FileInputStream in = new FileInputStream(src);

        BufferedInputStream bin = new BufferedInputStream(in, 10240);

        int readcount = 0 ;

        while( (readcount = bin.read(bufs, 0 , 10240)) != -1) {
            zos.write(bufs, 0 , readcount);
        }

        zos.closeEntry();
        bin.close();
    } else {
        // 文件夹
        File[] fs = src.listFiles();

        if (fs == null || fs.length == 0 ) {
            zos.putNextEntry(new ZipEntry(name + File.separator ));
            zos.closeEntry();
            return ;
        }

        for (File f : fs) {
            compress(zos, f, name + File.separator + f.getName());
        }
    }
}
```



# 二、



为了防止忘记用到的类都是哪个包下的，附带包目录，可供参考（包含一些无关的，懒得删）：

```java
import org.apache.commons.io.FileUtils;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.HttpHeaders;
import org.springframework.http.HttpStatus;
import org.springframework.http.MediaType;
import org.springframework.http.ResponseEntity;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import java.io.*;
import java.lang.reflect.Type;
import java.time.LocalDate;
import java.util.*;
import java.util.zip.ZipEntry;
import java.util.zip.ZipOutputStream;
```



# 三、



- 如果还想用来作为网络传输用的，需要对字节流进行一点包装：



```java
public ResponseEntity<byte[]> obtainZipDirEntity(String directoryAbsolutePath, String zipName) throws IOException {
    File dir = new File(directoryAbsolutePath);
    System.out.println("Check dir real path： "+dir);

    //设置HttpHeaders，使浏览器解析响应头后响应下载
    HttpHeaders headers = new HttpHeaders();
    //attachment 口令告诉了浏览器你需要进行下载动作
    headers.setContentDispositionFormData("attachment", zipName);
    //设置响应方式为二进制
    headers.setContentType(MediaType.APPLICATION_OCTET_STREAM);

    //设置默认的压缩目录
    File tempDirPath = new File(new File("").getAbsolutePath() + "/temp");
    if (!tempDirPath.exists()) {
        tempDirPath.mkdirs();
    }

    String fileName = zipName + ".zip";
    //进行压缩
    this.folderToZip(directoryAbsolutePath, tempDirPath.getAbsolutePath(), fileName);

    //找到压缩的压缩文件路径，并读取它为字节流
    File tempFilePath = new File(tempDirPath.getAbsolutePath() + "/" + fileName);
    return new ResponseEntity<byte[]>(FileUtils.readFileToByteArray(tempFilePath), headers, HttpStatus.CREATED);
}
```

