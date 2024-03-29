---
title: 大白话讲解 https 的三次握手过程
date: 2022-04-02
tags:
 - https
categories:
 - 计网
---

## 前置知识

### 对称加密

简单的加密方式,用同一个密钥进行加密解密

最大的问题就是**这个密钥怎么让传输的双方知晓，同时不被别人知道**

### 非对称加密

有两把密钥，通常一把叫做公钥、一把叫做私钥，用公钥加密的内容必须用私钥才能解开，同样，私钥加密的内容只有公钥能解开。相对于对称加密，非对称加密算法耗时更久。


### 数字证书
在开启 https 服务之前，我们服务器需要有一个证书。

**证书由两部分组成：**
- 明文信息 : 公钥 , 域名, 证书签名算法, 过期时间等其他信息
- 数字签名: 用证书签名算法将明文信息转成摘要(二进制数), 再用证书颁发机构(CA)的私钥加密摘要得到的结果

**证书有两种方式可以获得：**
1. 去阿里云购买并下载，个人试用免费。
2. 本地用命令行工具生成，如：
```bash
openssl genrsa -des3 -out sgfblog.com.key 1024
openssl req -new -key sgfblog.com.key -out sgfblog.com.csr
openssl rsa -in sgfblog.com.key -out sgfblog.com.nopass.key
```


## 主要流程
### 第一次握手
客户端：hi，这是我本地生成的一个随机字符串 `client_random_string_1` 和可支持的加密算法 A~D，你从中选好一种加密算法再告诉我哈。


### 第二次握手
服务端：好的。我这边已将  `client_random_string_1`  保存在本地。我选择了 A 加密算法，下面是我这边生成的随机字符串 `server_random_string_1`，以及我这边的证书，请注意查收~

### 第三次握手
客户端：收到。我先校验一下这个证书的可用性。

首先我使用 CA 机构提供的公钥解密一下这个证书里头的数字签名，得到信息摘要 a。然后用证书里提到的证书签名算法将明文信息加密得到 b，对比 a 和 b ，判断证书是否被篡改。

如果没有发生篡改，还要比较一下证书中的域名和当前请求的域名是否一致，以确保证书不会被掉包。

以上都通过校验后，我从证书中得到的公钥是 `public_key` 。我这边再次生成一个随机字符串 `client_random_string_2`，并且使用  `public_key` 加密，形成一个 `secret_key`，已经发给你了，请查收~

### 后续
服务端收到加密后的 `secret_key` 后，可以使用自己的私钥进行解密，得到 `client_random_string_2`。

至此，三次握手已经完成。客户端和服务端都会使用  `client_random_string_1`  + `client_random_string_2` + `server_random_string_1` ,配合加密算法 A，在本地生成一个对称性密钥
`symmetric_key` 。此时客户端和服务端的 `symmetric_key` 能保证一致，后续的通讯都是使用 `symmetric_key` 进行加密和解密。

### 图解
![https](/article/https.jpeg)

## 小剧场
### 为什么需要额外生成一个对称性密钥？
客户端： 伙计，我持有公钥，你持有私钥，咱们只用这一对非对称性密钥去加密和解密不行吗？为什么需要额外生成一个对称性密钥？

服务端： 公钥和私钥一开始都是存放在我这边的，怎么保证传输的公钥不被中间人窃取呢？既然无法保证公钥的隐秘性，那我们可以用公钥去加密一个字符串，这样这个字符串只有用我的私钥才能解密出来，只有天知地知你知我知，咱们再配合这个字符串重新生成一个对称性密钥，这样就可以保证这个对称性密钥的安全性。并且，使用对称性加密性能会更好。


### 生成一个对称性密钥为什么需要三个随机数？
客户端：上面的观点我很认同，但是我还有个疑问： 生成一个对称性密钥为什么需要三个随机数？我只用最后一个加密的随机字符串去生成不行吗？

服务端：这样更能保证安全性，不管是客户端还是服务器，都需要随机数，这样生成的密钥才不会每次都一样。由于SSL协议中证书是静态的，因此十分有必要引入一种随机因素来保证协商出来的密钥的随机性。SSL协议不信任每个主机都能产生完全随机的随机数，如果随机数不随机，那么第三个随机数就有可能被猜出来，那么只使用第三个随机数作为密钥就不合适了，因此必须引入新的随机因素。客户端的两个随机数和服务器上的一个随机数一同生成的密钥就不容易被猜出了，一个伪随机可能完全不随机，可是是三个伪随机就十分接近随机了，每增加一个自由度，随机性增加的可不是一。


### 证书的内容是否可以篡改？
某次偶然的机会，中间人得知客户端和服务端会发起一次重大的交易，中间人想通过篡改证书以伪装自己是服务端，从而窃取商业机密。在第二次握手的过程中，中间人截取证书内容，直接修改证书上的明文信息，将公钥换成自己的，企图蒙混过关。

客户端收到证书后，会验证数字证书，这时会发现证书上的数字签名和明文信息不对应。中间人的行动以失败告终。

中间人又试了一次，这次他不仅修改了明文信息，还配合自己的私钥（中间人并没有 CA 机构的私钥）和签名算法，将新的明文信息转变成新的数字签名，以此替换掉原来的数字签名。

这次，客户端收到证书后，仍然会验证数字证书，这时客户端会使用CA 机构的公钥去解密证书中的数字签名，发现明文信息和数字签名又对应不上。中间人的行动以失败告终。

### 证书是否可以掉包？
中间人不服。自己生成的假证书不行，那我用真实有效的证书替换总可以了吧？

客户端表示 too young too simple。在我解密证书的过程中会校验证书里记录的域名和我真实请求的域名是否一致，一个域名对应一个证书，结果肯定是不一致的，最后中间人又败了。

### 随机字符串是否可以篡改？
中间人还是不服，心想既然不能伪装服务端，那我伪装客户端试试。客户端生成的两个随机字符串都替换成我自己的，如此一来不是可以伪装客户端了吗？

中间人美滋滋的同时客户端已经和服务器建立起了 https 链接。可最后他也是只能听懂服务端在说什么，客户端说的话中间人完全没法听明白。因为客户端的随机数和中间人不一样，导致最后生成的对称性密钥不一样，自然中间人也没法解密客户端的消息，最终还是以失败告终。
