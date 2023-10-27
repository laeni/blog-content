---
title: 流式加密（加密超大文件）
author: 'Laeni'
tags: 安全
date: 2023-10-27
updated: 2023-10-27
---

当需要加密时，很多时候都是网络随便搜索一个示例来改一下即可。但是这些示例大部分都只适合加密少数据，当加密数据量过大，比如加密文件时很容易**OOM**。所以当加密大量数据时，不能一次性将待加密的密文一次性读取到内存，然后再一次性加密得到密文，而是应该边读边加密，并且边将得到的密文写到磁盘，这样就能以少量内存使用量加密大量数据。

虽然知道原理，但是具体应该怎么操作还是很有学问。比如以下Java代码可能会经常出现在我们的代码中：

```java
/**
 * 对称加密.
 *
 * @param plaintext 待加密的明文
 * @param key       加密密钥
 * @param iv        随机向量
 * @return 加密后的密文
 */
public byte[] encrypt(byte[] plaintext, byte[] key, byte[] iv) throws Exception {
    final Cipher cipher = Cipher.getInstance("SM4/CBC/PKCS5Padding", "BC");
    final Key key1 = new SecretKeySpec(key, "SM4/CBC/PKCS5Padding");
    final IvParameterSpec spec = new IvParameterSpec(iv);
    cipher.init(Cipher.ENCRYPT_MODE, key1, spec);
    return cipher.doFinal(plaintext);
}
```

在上述代码中，通过`javax.crypto.Cipher#doFinal(byte[])`方法即可得到对应的密文。但是当加密大量数据时，不能一次性加密，而是分为多次加密，那我们能不能反复调用`doFinal`方法，然后将得到的密文数据拼接起来呢？

答案是不行的，原因在于一般加密都是分块加密，即不管数据量多少，都要将数据转换为小块小块的（一般每块大小为`16`字节），以块为单位进行。既然是以块为单位，那最后一块数据量可能不够一块的大小，但是加密需要一整块数据。为了解决这个问题，所以需要对数据进行填充，以保证每块数据都是满的。但是这有可又有新的问题，那就是当最后一块数据刚好满足块大小时，我们并不知道这块数据最后那部分本身就是那样还是经过填充的，所以为了解决这个问题，无论如何都会对原始数据进行填充，即当最后一块不满足块大小要求时直接进行填充即可，如果刚好和块大于一致时，需要在最后增加一块，该块的全部数据都是填充数据。这就导致`doFinal`方法得到的结果一定是经过填充的，但是我们希望仅在最后一步进行填充，中途不填充。这个时候就需要使用其他方法，即在最后一步前使用`update`方法，最后一步使用`doFinal`方法，具体代码如下：

```java
/**
 * 对称加密.
 *
 * @param inputStream  输入流，从中读取明文数据
 * @param outputStream 输出流，加密后将密文写入该流
 * @param key          加密密钥
 * @param iv           随机向量
 */
public void encrypt(InputStream inputStream, OutputStream outputStream, byte[] key, byte[] iv) throws Exception {
    final Cipher cipher = Cipher.getInstance("SM4/CBC/PKCS5Padding", "BC");
    final Key key1 = new SecretKeySpec(key, "SM4/CBC/PKCS5Padding");
    final IvParameterSpec spec = new IvParameterSpec(iv);
    cipher.init(Cipher.ENCRYPT_MODE, key1, spec);

    int len;
    byte[] bytes;
    final byte[] buf = new byte[9];
    while ((len = inputStream.read(buf)) != -1) {
        bytes = cipher.update(buf, 0, len);
        if (bytes != null) {
            outputStream.write(bytes);
        }
    }
    outputStream.write(cipher.doFinal());
}
```

> **注意**：
>
> 1. 一定要对`update`方法返回的结果进行判空，因为并不是每次都会返回加密后的密文。当传入的明文数据不够一块时，`Cipher`实例会先记录下该未加密的明文，然后返回`null`，知道数据够加密了就返回加密后的密文。
> 2. 最后一步需要调用`doFinal`方法来告诉`Cipher`实例本次加密已结束，且调用该方法一定会返回密文（即使不通过参数传入任何明文数据），因为上面提到的填充问题，如果缓冲中有部分未加密的数据，那调用`doFinal`方法将得到该部分数据填充后加密得到的密文，如果缓冲已经空了，那调用`doFinal`方法得到的一整块填充数据加密得到的密文。
> 3. `Cipher`实例是可以复用的，但是它不是线程安全的，并且每次调用`doFinal`方法后才可以加密新数据，以为调用`doFinal`方法后`Cipher`实例会自动进行初始化，如果上一次因为异常情况退出，需要 重新进行加密时，需要手动调用`init`初始化后才可使用。

实际上，JDK中`javax.crypto.CipherOutputStream`类已经实现上述逻辑，当关闭流时会自动调用`doFinal`方法，所以上述代码可以使用`javax.crypto.CipherOutputStream`进行简化：

```java
/**
 * 对称加密.
 *
 * @param inputStream  输入流，从中读取明文数据
 * @param outputStream 输出流，加密后将密文写入该流
 * @param key          加密密钥
 * @param iv           随机向量
 */
public void encrypt(InputStream inputStream, OutputStream outputStream, byte[] key, byte[] iv) throws Exception {
    final Cipher cipher = Cipher.getInstance("SM4/CBC/PKCS5Padding", "BC");
    final Key key1 = new SecretKeySpec(key, "SM4/CBC/PKCS5Padding");
    final IvParameterSpec spec = new IvParameterSpec(iv);
    cipher.init(Cipher.ENCRYPT_MODE, key1, spec);

    try (CipherOutputStream os = new CipherOutputStream(outputStream, cipher);) {
        int len;
        final byte[] buf = new byte[1024];
        while ((len = inputStream.read(buf)) != -1) {
            os.write(buf, 0, len);
        }
    }
}
```

