---
title: 微信小程序 之 RSA长文本分段加解密
date: 2020-07-23 20:00:00
categories:
- 微信小程序

tags:
- 微信小程序
- 加密与解密
- 网络安全



---







# 1、问题

顺着上一篇博客[微信小程序 之 RSA非对称加密](https://bingcu.top/2020/07/23/%E5%BE%AE%E4%BF%A1%E5%B0%8F%E7%A8%8B%E5%BA%8F%E4%B9%8BRSA%E9%9D%9E%E5%AF%B9%E7%A7%B0%E5%8A%A0%E5%AF%86/)的讲解，我们知道了如何在微信小程序中使用RSA加解密网路传输过程中的数据。然而，作者在今天这是将上面的内容投入开发过程的时候，发现之前提供的`jsencrypt.js`文件，以及自己写的后端RSA工具，都有一个令人头疼的问题：

**没法进行长文本的加解密，确切的说是每次只能加解密 `117 bytes` 内的数据，最多处理`117 bytes `的数据。**

这确实很头疼啊，一般从前端发送给后端的次数是越少越好。因为尽量减少前端的请求次数，能够有效减少后端服务器的压力，而且分为多次发送，那数据的顺序本身也会变得凌乱，因此这是不可取的！

那得想个办法啊！

于是作者发现，既然一次加密不能超过`117 bytes`，那就分为多次进行加密。然后只要被加密的文本不超过`117 bytes`，那解密的时候也自然不会出问题了，于是我们需要手动写一个前端的`分段加密函数`，和一个后端的`分段解密函数`。





# 2、分段加密

- 前端 JS：



```js
//首先记得先引进上一篇博客所提到的包含RSA算法的相关文件
var jsencrypt = require('../../utils/jsencrypt.min.js');
```



```js
//分段加密，返回json格式的对象
blockEncryption: function(publickey, obj){
    //创建rsa对象
    var rsa = new jsencrypt.JSEncrypt();
    //设置公钥
    rsa.setPublicKey(publickey)

    //将obj对象转为字符串
    var str = JSON.stringify(obj)
    //限定每次切割的明文片段的最大长度
    const max =117;

    //存储每次切割加密后的密文片段
    var arr=[]
    while(str.length>max){
        //从明文文本中抽取前max个字节
        var part = str.substr(0, max)
        str = str.slice(max)
        
        //encrypt()函数加密后返回的密文文本片段长度固定为172
        //后端对密文切割时需注意，要按172 bytes 来进行切割
        const ciphertext = rsa.encrypt(part)
        arr.push(ciphertext)
    }
    
    //单独处理最后一个明文文本片段
    //因为最后的一个明文文本肯定是小于或等于117bytes的
    const ciphertext = rsa.encrypt(str)
    arr.push(ciphertext)

    //将所有的密文拼接成一个字符串，然后返回
    var result = "";
    for(var i = 0;i<arr.length;i++){
        result += arr[i]
    }
    return result;
}
```

# 3、分段解密

- 后端 Java



```java
/**
 * 分段解密
 *
 * @param privateKey 密钥
 * @param ciphertext 被加密的文本
 * @return 返回明文
 * @throws Exception 解密异常
 */
public static String blockDecryption( String privateKey, String ciphertext) throws Exception {
    //用来存储被切割后的密文片段
    ArrayList<String> cipherList = new ArrayList<>();
    int keyLength = 172;    //文本被加密后的密文长度都是172，所以每172个字符切割一次
    int textLength = 117;   //117是加密时的最大长度，这是加密时需要的，在这里没用，只是为了提醒一下
    
    //切割密文，然后将每个密文片段都添加到cipherList中
    int pos;
    for (pos = keyLength;pos<ciphertext.length();pos+=keyLength){
        String part = ciphertext.substring(pos-keyLength, pos);
        cipherList.add(part);
    }
    cipherList.add(ciphertext.substring(pos - keyLength, ciphertext.length()) );

    //解密cipherList中所有的密文片段，导出完整的明文
    StringBuilder cleartext = new StringBuilder();
    for(String text: cipherList){
        String part = RSAEncrypt.decrypt(text, privateKey);
        cleartext.append(part);
    }

    //返回明文
    return cleartext.toString();
}
```

