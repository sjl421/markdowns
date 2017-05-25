# Java 加密时可能涉及的类

Class Mac

```java
javax.crypto.Mac
public class Mac
extends Object
implements Cloneable
This class provides the functionality of a "Message Authentication Code" (MAC) algorithm.
A MAC provides a way to check the integrity of information transmitted over or stored in an unreliable medium, based on a secret key. Typically, message authentication codes are used between two parties that share a secret key in order to validate information transmitted between these parties.

A MAC mechanism that is based on cryptographic hash functions is referred to as HMAC. HMAC can be used with any cryptographic hash function, e.g., MD5 or SHA-1, in combination with a secret shared key. HMAC is specified in RFC 2104.

Every implementation of the Java platform is required to support the following standard Mac algorithms:

HmacMD5
HmacSHA1
HmacSHA256
These algorithms are described in the Mac section of the Java Cryptography Architecture Standard Algorithm Name Documentation. Consult the release documentation for your implementation to see if any other algorithms are supported.
oracle: http://docs.oracle.com/javase/7/docs/api/javax/crypto/Mac.html
```

```

```

## Part 2

### [java加密解密研究8、MAC算法家族](http://blog.csdn.net/lonelyroamer/article/details/7656338)

一、概述

MAC[算法](http://lib.csdn.net/base/datastructure)结合了MD5和SHA算法的优势，并加入密钥的支持，是一种更为安全的消息摘要算法。

MAC（Message Authentication Code，消息认证码算法）是含有密钥的散列函数算法，兼容了MD和SHA算法的特性，并在此基础上加入了密钥。日次，我们也常把MAC称为HMAC（keyed-Hash Message Authentication Code）。

MAC算法主要集合了MD和SHA两大系列消息摘要算法。MD系列的算法有HmacMD2、HmacMD4、HmacMD5三种算法；SHA系列的算法有HmacSHA1、HmacSHA224、HmacSHA256、HmacSHA384.HmacSHA512五种算法。

经过MAC算法得到的摘要值也可以使用十六进制编码表示，其摘要值长度与参与实现的摘要值长度相同。例如，HmacSHA1算法得到的摘要长度就是SHA1算法得到的摘要长度，都是160位二进制码，换算成十六进制编码为40位。

二、实现和应用

1、Sun的实现和应用

在java6中，MAC系列算法需要通过Mac类提供支持。java6中仅仅提供HmacMD5、HmacSHA1、HmacSHA256、HmacSHA384和HmacSHA512四种算法。

Mac算法是带有密钥的消息摘要算法，所以实现起来要分为两步：

1）、构建密钥

2）、执行消息摘要