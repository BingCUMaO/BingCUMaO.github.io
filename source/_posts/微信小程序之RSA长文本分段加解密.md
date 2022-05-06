---
title: 微信小程序 之 RSA长文本分段加解密
date: 2020-07-24 14:00:00
categories:
- 技术博客

tags:
- 微信小程序
- 加密与解密
- 网络安全



---







# 1、问题

顺着上一篇博客[微信小程序 之 RSA非对称加密](https://bingcu.top/2020/07/23/%E5%BE%AE%E4%BF%A1%E5%B0%8F%E7%A8%8B%E5%BA%8F%E4%B9%8BRSA%E9%9D%9E%E5%AF%B9%E7%A7%B0%E5%8A%A0%E5%AF%86/)的讲解，我们知道了如何在微信小程序中使用RSA加解密网路传输过程中的数据。然而，作者在今天正式将上面的内容投入开发过程的时候，发现之前提供的`jsencrypt.js`文件，以及自己写的后端RSA工具，都有一个令人头疼的问题：

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

    //将obj对象转为JSON字符串
    //同时将JSON字符串转为URI格式编码，以解决中文等多字节字符分割后无法解码的问题
    var str =  encodeURI(JSON.stringify(obj))
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





***注意：***

**传进来的`ciphertext`文本是前端为了解决中文字符等字符占用多字节问题，转化为URI编码格式后，再将加密成的n个密文片段组成在一起的。**

**因此该方法返回后的明文是URI格式的，记得转化成`UFT-8`的编码格式：**



```java
import java.net.URLDecoder;

String urlFormatText = RSAEncrypt.blockDecryption(privateKey, ciphertext);
String cleartext = URLDecoder.decode(urlFormatText, "UTF-8");
```







---

---

## 更新于2020-07-30	↓

纯手写的一个RSA加解密工具类，提供对byte[] 和String类型转化的工具，并提供“部分加密”的字节加密工具：

```java
package top.bingcu.common.utils.sign;

import org.apache.commons.codec.binary.Base64;
import javax.crypto.Cipher;
import java.io.ByteArrayInputStream;
import java.io.ByteArrayOutputStream;
import java.security.KeyFactory;
import java.security.KeyPair;
import java.security.KeyPairGenerator;
import java.security.NoSuchAlgorithmException;
import java.security.SecureRandom;
import java.security.interfaces.RSAPrivateKey;
import java.security.interfaces.RSAPublicKey;
import java.security.spec.PKCS8EncodedKeySpec;
import java.security.spec.X509EncodedKeySpec;
import java.util.HashMap;
import java.util.Map;

/**
 * 该类提供字节与字符串的加解密工具，以及优化分段加解密
 * 此外还提供了采用文件部分加密策略的工具。
 *
 * 其他方法都默认经过Base64处理成8 bit的格式。
 * 加密相关的方法会编码成Base64，解密相关的方法会解码成Base64。
 * 除了encryptBytesOnlyPartB64()方法和decryptBytesOnlyPartB64()方法
 * 会对指定重量的字节流进行加密和Base64编码以外，其他字节流均保持不变。
 */
public class RSAEncrypt {

    /**
     * 封装随机产生的公钥与私钥
     */
    private static  Map<String, String> keyMap = new HashMap<String, String>();

    /**
     * 默认的"部分加密"的权重，越重代表加密的内容越多。
     *
     * 重量默认为10表示：依照字节流的顺序，被加密的内容为字节流的
     * 前117*10个字节的数据，即表示加密重量为10
     */
    private static final int DEFAULT_WEIGHT = 10;

    /**
     * 该类提供文件字节流的局部加密，该属性为局部加密提供范围配置
     */
    private static Integer defaultFileEncryptionWeight = DEFAULT_WEIGHT;


    /**
     * 随机生成密钥对
     *
     * @return 公钥
     * @throws NoSuchAlgorithmException 生成失败异常
     */
    public static String generateKeyPair() throws NoSuchAlgorithmException {
        // KeyPairGenerator类用于生成公钥和私钥对，基于RSA算法生成对象
        KeyPairGenerator keyPairGen = KeyPairGenerator.getInstance("RSA");
        // 初始化密钥对生成器，密钥大小为96-1024位
        keyPairGen.initialize(1024,new SecureRandom());
        // 生成一个密钥对，保存在keyPair中
        KeyPair keyPair = keyPairGen.generateKeyPair();
        RSAPrivateKey privateKey = (RSAPrivateKey) keyPair.getPrivate();   // 得到私钥
        RSAPublicKey publicKey = (RSAPublicKey) keyPair.getPublic();  // 得到公钥
        String publicKeyString = new String(Base64.encodeBase64(publicKey.getEncoded()));
        // 得到私钥字符串
        String privateKeyString = new String(Base64.encodeBase64((privateKey.getEncoded())));

        synchronized (keyMap){
            // 将公钥和私钥保存到Map
            //key:公钥，  value:私钥
            keyMap.put(publicKeyString, privateKeyString);
        }

        return publicKeyString;
    }

    /**
     * 获取公钥对应的密钥
     *
     * @param publicKey 公钥
     * @return 密钥
     */
    public static String obtainPrivateKey(String publicKey){
        return keyMap.get(publicKey);
    }

    /**
     * 删除密钥对
     *
     * @param publicKey 公钥
     * @return 被删除的密钥
     */
    public  static String dropKeyPair(String publicKey){
        synchronized (keyMap){
            return keyMap.remove(publicKey);
        }
    }


    /**
     * 普通的RSA公钥加密，单次执行最多仅支持加密117个字节的数据
     *
     * @param text 需加密的字符串文本，且要求未经过Base64编码
     * @param publicKey 公钥
     * @return 已经过Base64编码的密文
     * @throws Exception 加密过程中的异常信息
     */
    public static String encrypt( String text, String publicKey ) throws Exception{
        return RSAEncrypt.encryptFromBytes(text.getBytes("UTF-8"), publicKey);
    }

    /**
     * 普通的RSA密钥解密，单次执行最多仅支持解密一份RSA所生成的密文，
     * 且密文长度固定为172个字节
     *
     * @param text 需解密的字符串格式的文本，且要求经过Base64编码
     * @param privateKey 密钥
     * @return 经过Base64解码的明文字符串
     * @throws Exception 解密过程中的异常信息
     */
    public static String decrypt(String text, String privateKey) throws Exception{
        byte[] bytes = RSAEncrypt.decryptToBytes(text, privateKey);
        return new String(bytes);
    }


    /**
     * 普通的RSA公钥加密，单次执行最多仅支持加密117个字节的数据
     *
     * @param text 需加密的byte[] 格式文本，且要求未经过Base64编码
     * @param publicKey 公钥
     * @return 已经过Base64编码的密文
     * @throws Exception 加密过程中的异常信息
     */
    public static String encryptFromBytes(byte[] text, String publicKey) throws Exception{
        //base64编码的公钥
        byte[] decoded = Base64.decodeBase64(publicKey);
        RSAPublicKey pubKey = (RSAPublicKey) KeyFactory
                .getInstance("RSA")
                .generatePublic(new X509EncodedKeySpec(decoded));
        //RSA加密
        Cipher cipher = Cipher.getInstance("RSA");
        cipher.init(Cipher.ENCRYPT_MODE, pubKey);
        String outStr = Base64.encodeBase64String(cipher.doFinal(text));
        return outStr;
    }


    /**
     * 普通的RSA密钥解密，单次执行最多仅支持解密一个RSA所生成的密文，
     * 并返回已经过Base64编码的8bit字节数组明文
     *
     * @param text 需解密的文本，且要求经过Base64编码
     * @param privateKey 密钥
     * @return 经过Base64解码的明文字节数据
     * @throws Exception 加密过程中的异常信息
     */
    public static byte[] decryptToBytes(String text, String privateKey) throws Exception{
        //base64编码的私钥
        byte[] decoded = Base64.decodeBase64(privateKey);
        RSAPrivateKey priKey = (RSAPrivateKey) KeyFactory.getInstance("RSA").generatePrivate(new PKCS8EncodedKeySpec(decoded));
        //RSA解密
        Cipher cipher = Cipher.getInstance("RSA");
        cipher.init(Cipher.DECRYPT_MODE, priKey);

        //64位解码加密后的字符串
        byte[] inputByte = Base64.decodeBase64(text.getBytes("UTF-8"));
        return cipher.doFinal(inputByte);
    }

    /**
     * 由于文件本身就很大，而RSA由于公钥长度固定，所以被加密文本的长度
     * 也被受限了，全部都进行加密，那加解密时间将会非常长，用户体验极差。
     *
     * 因此为解决文件加密，采用字节流部分加密，只加密字节流加密前117*10个
     * bytes。分为10次加密，每次加密117个字节。然后得到172*10个字节的字节数组密文，
     * 将这些字节替换掉原来的117*10个字节，然后返回。
     *
     * 若字节流长度为N且小于117*10个字节，将全部被加密。并将N个字节全部替换
     * 成长度为172*10个字节的密文。
     *
     * @param publicKey 公钥
     * @return 返回前172*10个字节为密文的byte[]类型的文件流，且已经过Base64编码
     */
    public  static byte[] encryptFileBytes(byte[] stream, String publicKey) throws Exception {
        return RSAEncrypt.encryptBytesOnlyPartB64(stream, publicKey, getDefaultFileEncryptionWeight());
    }


    /**
     * encryptFileBytes()方法对应的解密策略，循环10次解密前172*10个字节。
     * 若stream的大小本身小于172*10个字节，那将全部解密
     *
     * @param stream 经过Base64编码的密文流
     * @param privateKey 密钥
     * @return 经过Base64解码的明文流
     */
    public static byte[] decryptFileBytes(byte[] stream, String privateKey) throws Exception {
        return RSAEncrypt.decryptBytesOnlyPartB64(stream, privateKey, getDefaultFileEncryptionWeight());
    }


    /**
     * 分段加密，工具weight权重参数进行加密
     *
     * 可自定义加密字节数据的权重大小，重量越大被加密的部分越多。
     *
     * @param stream 未经过Base64编码的字节流
     * @param publicKey 公钥
     * @param weight 加密重量
     * @return 返回部分经过Base64编码的密文字节流和剩余未加密的明文字节流
     * @throws Exception 加密异常
     */
    public  static byte[] encryptBytesOnlyPartB64(byte[] stream, String publicKey, int weight) throws Exception {
        final int tick = weight;
        final int block = 117;  //每次加密的字节块长度
        int bigBlock = block*tick;  //被加密的总字节块长度

        //base64编码的公钥
        byte[] decoded = Base64.decodeBase64(publicKey);
        RSAPublicKey pubKey = (RSAPublicKey) KeyFactory
                .getInstance("RSA")
                .generatePublic(new X509EncodedKeySpec(decoded));
        //RSA加密初始化
        Cipher cipher = Cipher.getInstance("RSA");
        cipher.init(Cipher.ENCRYPT_MODE, pubKey);

        bigBlock = stream.length < bigBlock ? stream.length : bigBlock;
        //若stream的长度直接就小于一次加密的文本长度上限，那就一次性直接加密它。
        if (bigBlock<block){
            byte[] cipherBytes = cipher.doFinal(stream);
            return Base64.encodeBase64String(cipherBytes).getBytes();
        }

        //剩余未被加密的旁观者bytes
        byte[] spectator = new byte[stream.length - bigBlock];
        System.arraycopy( stream, bigBlock, spectator, 0, spectator.length);


        //加密参与者bytes
        ByteArrayInputStream bais = new ByteArrayInputStream(stream);
        ByteArrayOutputStream baos = new ByteArrayOutputStream();
        byte[] buff = new byte[block];
        for (int i = 0;i<tick; i++) {
            int read = bais.read(buff);
            if (-1 == read) break;
            byte[] temp = cipher.doFinal(buff, 0, read);
            baos.write(Base64.encodeBase64String(temp).getBytes());
        }
        baos.write(spectator);

        byte[] retult = baos.toByteArray();
        bais.close();
        baos.close();
        return retult;
    }


    /**
     * 分段解密，工具weight权重参数进行解密
     *
     * encryptBytesOnlyPartB64()方法对应的解密策略，仅支持解密
     * 与加密时具有相同加密重量的文本字节流，否则将编码不正确
     *
     * @param stream 部分经过Base64解码的字节流
     * @param privateKey 密钥
     * @param weight 重量
     * @return 返回经过Base64解码的明文字节流
     * @throws Exception
     */
    public static byte[] decryptBytesOnlyPartB64(byte[] stream, String privateKey, int weight) throws Exception {
        final int tick = weight;
        final int block = 172;
        int bigBlock = block * tick;

        //base64编码的私钥
        byte[] decoded = Base64.decodeBase64(privateKey);
        RSAPrivateKey priKey = (RSAPrivateKey) KeyFactory
                .getInstance("RSA")
                .generatePrivate(new PKCS8EncodedKeySpec(decoded));
        //RSA解密初始化
        Cipher cipher = Cipher.getInstance("RSA");
        cipher.init(Cipher.DECRYPT_MODE, priKey);

        //剩余未被加密的旁观者bytes
        byte[] spectator = null;
        //加密参与者bytes
        byte[] participant = null;
        if (weight*block<stream.length){       //存在一部分不会被解密
            spectator = new byte[stream.length - bigBlock];
            System.arraycopy( stream, bigBlock, spectator, 0, spectator.length);
            participant = new byte[bigBlock];
            System.arraycopy( stream, 0, participant, 0, bigBlock);
        }else {
            spectator = new byte[0];
            participant = stream;
        }

        byte[] buff = new byte[block];
        ByteArrayOutputStream baos = new ByteArrayOutputStream();
        ByteArrayInputStream bais = new ByteArrayInputStream(participant);
        for (int i = 0;i<tick; i++) {
            int read = bais.read(buff);
            if (-1 == read) break;
            else if (read!=buff.length){
                byte[] temp = new byte[read];
                System.arraycopy( buff, 0, temp, 0, read);
                buff = temp;
            }
            byte[] part = Base64.decodeBase64(buff);
            baos.write(cipher.doFinal(part));
        }
        baos.write(spectator);

        byte[] retult = baos.toByteArray();
        bais.close();
        baos.close();
        return retult;
    }


        /**
         * 分段加密，这种分段加密，频繁地调用RSAEncrypt.encrypt方法，
         * 频繁创建和配置相关的对象，效率较低
         *
         * @param publicKey 公钥
         * @param cleartext 明文
         * @return 密文
         * @throws Exception 加密异常
         */
//    public static String blockEncryption(String publicKey, String cleartext) throws Exception {
//        //用来存储被切割后的明文片段
//        ArrayList<String> clearList = new ArrayList<>();
//        int keyLength = 172;    //文本被加密后的密文长度都是172，所以每172个字符切割一次,在这里没用，只是为了提醒一下
//        int textLength = 117;   //117是加密时的最大长度，这是加密时需要的
//        int pos;
//
//        cleartext = new String(cleartext.getBytes(), "UTF-8");
//
//        //切割明文，然后将每个片段都添加到
//        for (pos = textLength;pos<cleartext.length();pos+=textLength){
//            String part = cleartext.substring(pos-textLength, pos);
//            clearList.add(part);
//        }
//        clearList.add(cleartext.substring(pos - textLength, cleartext.length()) );
//
//        //加密cipherList中所有的明文片段，导出完整的密文
//        StringBuilder ciphertext = new StringBuilder();
//        for(String text: clearList){
//            System.out.println("Text的byte长度："+text.getBytes().length);
//            String part = RSAEncrypt.encrypt(text, publicKey);
//            ciphertext.append(part);
//        }
//
//        //返回明文
//        return ciphertext.toString();
//    }



    /**
     * 完全分段加密，优化版本
     *
     * 所有的文本都会被加密
     *
     * @param publicKey 公钥
     * @param cleartext 未经过Base64编码的明文
     * @return 经过Base64编码的密文
     * @throws Exception 加密异常
     */
    public static String blockEncryption(String publicKey, String cleartext) throws Exception {
        byte[] bytes = RSAEncrypt.encryptBytesOnlyPartB64(cleartext.getBytes("UTF-8"), publicKey, cleartext.length());
        return new String(bytes);
    }


//    /**
//     * 分段解密，这种分段解密，频繁地调用RSAEncrypt.decrypt方法，
//     * 频繁创建和配置相关的对象，效率较低效率较低
//     *
//     * @param privateKey 密钥
//     * @param ciphertext 被加密的文本
//     * @return 返回明文
//     * @throws Exception 解密异常
//     */
//    public static String blockDecryption( String privateKey, String ciphertext) throws Exception {
//
//        //用来存储被切割后的密文片段
//        ArrayList<String> cipherList = new ArrayList<>();
//        int keyLength = 172;    //文本被加密后的密文长度都是172，所以每172个字符切割一次
//        int textLength = 117;   //117是加密时的最大长度，这是加密时需要的，在这里没用，只是为了提醒一下
//        int pos;
//
//        //切割密文，然后将每个片段都添加到
//        for (pos = keyLength;pos<ciphertext.length();pos+=keyLength){
//            String part = ciphertext.substring(pos-keyLength, pos);
//            cipherList.add(part);
//        }
//        cipherList.add(ciphertext.substring(pos - keyLength, ciphertext.length()) );
//
//        //解密cipherList中所有的密文片段，导出完整的明文
//        StringBuilder cleartext = new StringBuilder();
//        for(String text: cipherList){
//            String part = RSAEncrypt.decrypt(text, privateKey);
//            cleartext.append(part);
//        }
//
//        //返回明文
//        return cleartext.toString();
//    }


    /**
     * 完全分段解密，优化版本
     *
     * 会解密所有的文本
     *
     * @param privateKey 密钥
     * @param ciphertext 已经过Base64编码的密文字符文本
     * @return 返回经过Base64解码的明文
     * @throws Exception 解密异常
     */
    public static String blockDecryption( String privateKey, String ciphertext) throws Exception {
        byte[] bytes = RSAEncrypt.decryptBytesOnlyPartB64(ciphertext.getBytes(), privateKey, ciphertext.length());
        return new String(bytes);
    }



    /**
     * 获取全局的文件加密重量
     *
     * @return 返回默认的文件加密重量
     */
    public static int getDefaultFileEncryptionWeight() {
        return defaultFileEncryptionWeight;
    }

    /**
     * 设置全局的文件加密重量，仅针对文件加密方法
     *
     * @param defaultFileEncryptionWeight 文件加密重量
     */
    public static void setDefaultFileEncryptionWeight(int defaultFileEncryptionWeight) {
        synchronized (RSAEncrypt.defaultFileEncryptionWeight){
            RSAEncrypt.defaultFileEncryptionWeight = defaultFileEncryptionWeight;
        }

    }
}
```

```js

generateKeyPair: function(){
    var rsa = new jsencrypt.JSEncrypt();
    var keys = rsa.getKey()
    var publicKey  = keys.getPublicKey();
    var privateKey = keys.getPrivateKey();

    publicKey = publicKey.replace("-----BEGIN PUBLIC KEY-----", "");
    publicKey = publicKey.replace("-----END PUBLIC KEY-----", "")
    publicKey = publicKey.replace(" ", "")

    privateKey = privateKey.replace("-----BEGIN RSA PRIVATE KEY-----", "")
    privateKey = privateKey.replace("-----END RSA PRIVATE KEY-----", "")
    privateKey = privateKey.replace(" ", "");


    var pair = {}
    pair.publicKey = publicKey
    pair.privateKey = privateKey

    return pair;
}

//分段加密，返回json格式的对象
blockEncryption: function(publickey, str){
    var rsa = new jsencrypt.JSEncrypt();
    rsa.setPublicKey(publickey)

    const max =117;

    var arr=[]
    while(str.length>max){
        var part = str.substr(0, max)
        str = str.slice(max)

        const ciphertext = rsa.encrypt(part)
        arr.push(ciphertext)
    }
    const ciphertext = rsa.encrypt(str)
    arr.push(ciphertext)

    var result = "";
    for(var i = 0;i<arr.length;i++){
        result += arr[i]
    }
    return result;
}

blockDecryption: function(privateKey, str){
    var rsa = new jsencrypt.JSEncrypt();
    rsa.setPrivateKey(privateKey)

    const max = 172;

    var arr = []
    while(str.length>max){
        var part = str.substr(0, max)
        str = str.slice(max)

        const cleartext = rsa.decrypt(part)
        arr.push(cleartext)
    }
    const cleartext = rsa.decrypt(str)
    arr.push(cleartext)

    var result = "";
    for(var i = 0;i<arr.length;i++){
        result+=arr[i];
    }
    return result;
}

blockDecryptFile: function(fileBytes, privateKey,  weight){
    var rsa = new jsencrypt.JSEncrypt();
    rsa.setPrivateKey(privateKey)

    const block = 172;
    var bigBlock = block*weight
    var spectator = null
    var participant  = null
    if(weight*block<fileBytes.length){
        spectator = fileBytes.slice(bigBlock)
        participant = fileBytes.substr(0, bigBlock)
    }else{
        spectator = ""
        participant = fileBytes
    }

    var list = []
    var tick = 0;
    while(tick++<weight&&participant.length>0){
        var part = participant.substr(0, block)
        participant = participant.slice(block)

        var cleartext = rsa.decrypt(part)
        list.push(cleartext)
    }
    list.push(spectator)

    var result = ""
    tick = 0;
    while(tick++<list.length){
        result += list[tick];
    }

    return result;
}

generateUuid:function() {
    var s = [];
    var hexDigits = "0123456789abcdef";
    for (var i = 0; i < 36; i++) {
        s[i] = hexDigits.substr(Math.floor(Math.random() * 0x10), 1);
    }
    s[14] = "4"; // bits 12-15 of the time_hi_and_version field to 0010
    s[19] = hexDigits.substr((s[19] & 0x3) | 0x8, 1); // bits 6-7 of the clock_seq_hi_and_reserved to 01
    s[8] = s[13] = s[18] = s[23] = "-";

    var uuid = s.join("");
    return uuid;
}
```

