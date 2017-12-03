# 使用Integer.toHexString时遇到的问题

在读一段关于md5加密的代码时看到一句代码不是很理解，先看一下原始的加密代码：

```java
 public static String md5Signature(String message) {

    try {
      MessageDigest md = MessageDigest.getInstance("MD5");
      byte[] array = md.digest(message.getBytes());
      StringBuffer sb = new StringBuffer();
      for (int i = 0; i < array.length; ++i) {
        sb.append(Integer.toHexString((array[i] & 0xFF) | 0x100).substring(1, 3));
      }
      return sb.toString();
    } catch (NoSuchAlgorithmException e) {
    }
    return null;
  }
```

这是一段md5家吗代码，先用mds将加密的字符串转向成byte数组，然后再将此byte数组转换成16进制字符串；

产生疑问的代码就是在byte数组转换成16进制字符串的那句代码:

```java
   for (int i = 0; i < array.length; ++i) {
        sb.append(Integer.toHexString((array[i] & 0xFF) | 0x100).substring(1, 3));
      }
```

前面的 `array[i] & 0xFF `是可以理解的，是跟计算机对负数用补码的存储方式有关的，网上很多资料都有讲到这个，关键是后面的那部分，`array[i] & 0xFF`之后还和 `0x100`做了或运算之后取得到的字符的后两个字符，后来经过自己做测试对比才发现了最终的问题：

问题出在`Integer.toHexString()方法在处理 ascii码的0-15` 这个区间内时:

Integer.toHexString()方法是将输入的Int类型的数据用16进制的字符表示，就是说将十进制的数字转换成十六进制，然后用字符表示（要么用String字符串表示，要么就是惨惨的0x***表示了），比如说字符97，其用十六进制表示为61，通过Integer.toHexString(97)得到的就是“61”；同样以字符`a`为例， 其ascii码为97，经过将`a`转换成byte的类型再通过`& 0xFF`得到的结果为`10010111`，转换成Integer类型也是97，所以经过Integer.toHexString()处理后得到的结果也是“61”；

下面看下Integer.toHexString()在处理 ascii码的 0-15之间时会发生什么;

```java
    @Test
    public void test(){
        char[] a= {0,1,2,3,4,5,6,7,8,9};
        System.out.println(a);
        for(int i = 0; i < a.length; i++){
            System.out.print(Integer.toHexString(a[i] & 0xFF) + " ");
        }
        System.out.println();
        for(int i = 0; i < a.length; i++){
            System.out.print(Integer.toHexString(a[i] & 0xFF | 0x100).substring(1,3) + " ");
        }
    }

    /*
    0 1 2 3 4 5 6 7 8 9 a b c d e f 
	00 01 02 03 04 05 06 07 08 09 0a 0b 0c 0d 0e 0f 
    */
```

通过测试结果就可以看出问题了，Integer.toHexString()在处理0-15之间的ascii码时用一位16进制的字符就可以表示了，最终得到的转换结果也是1位，但是我们更希望要的结果是下面的表示的结果，用两位十六进制数表示，哪怕高位为0，这正是

```java
Integer.toHexString(a[i] & 0xFF | 0x100).substring(1,3)
```

这行代码的作用；

