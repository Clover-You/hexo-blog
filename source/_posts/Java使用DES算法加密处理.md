title: Java使用DES算法加密处理
categories: 随笔
tags: [JAVA]
date: 2022-01-02 03:17:00
---
定义所需要用到的变量

```java
private static final String ENCRYPTION_KEY = "HI, I'M UPYOU.";
private static final String CHARSET_NAME = "UTF-8";
private static final String ALGORITHM_NAME = "DES";
private static Key key;
```

将`ENCRYPTION_KEY`作为一个加密密钥，将`ENCRYPTION_KEY`转成`Key`类型。

```java
static {
    try {
        // 生成 ALGORITHM_NAME 算法对象
        KeyGenerator edsAlgorithmInstance = KeyGenerator.getInstance(ALGORITHM_NAME);
        // 使用SHA1安全策略
        SecureRandom sha1PRNG = SecureRandom.getInstance("SHA1PRNG");
        // 设置密钥 ENCRYPTION_KEY
        sha1PRNG.setSeed(ENCRYPTION_KEY.getBytes());
        // 初始化KeyGenerator对象
        edsAlgorithmInstance.init(sha1PRNG);
        //生成 key
        key = edsAlgorithmInstance.generateKey();
    } catch (NoSuchAlgorithmException e) {
        e.printStackTrace();
    }
}
```

创建对字符串加密方法 `encrypetString`

```java
public static String encryptString(){}
```

**`encrypetString`方法中操作**


需要使用`BASE64Encoder`将加密后的结果转成 `String`

```java
BASE64Encoder base64Encoder = new BASE64Encoder();
```

得到待加密字符串基于`CHARSET_NAME`编码的`byte[]`

```java
byte[] bytes = str.getBytes(CHARSET_NAME);
```

创建基于`ALGORITHM_NAME`的加密算法

```java
Cipher cipher = Cipher.getInstance(ALGORITHM_NAME);
```

初始化加密设置,使用无三参初始化方式。
![image.png](http://qiniu-note-image.ctong.top/note/images/202112271118040.png)


```java
cipher.init(Cipher.ENCRYPT_MODE, key);
```

将密码进行加密，此处返回的是加密后的`byte[]`对象

```java
byte[] doFinal = cipher.doFinal(bytes);
```

使用 `BASE64Encoder` 将加密结果转成 `String` 后返回

```java
return base64Encoder.encode(doFinal);
```


```java
public static String encryptString(String str){
    //基于BASE64编码，接收byte[]并转换成String
    BASE64Encoder base64Encoder = new BASE64Encoder();
    try {
        // 按UTF8编码
        byte[] bytes = str.getBytes(CHARSET_NAME);
        // 获取加密对象
        Cipher cipher = Cipher.getInstance(ALGORITHM_NAME);
        // 初始化密码信息
        cipher.init(Cipher.ENCRYPT_MODE, key);
        // 加密
        byte[] doFinal = cipher.doFinal(bytes);
        // byte[]to encode好的String并返回
        return base64Encoder.encode(doFinal);
    } catch (Exception e) {
        throw new RuntimeException(e);
    }
}
```
