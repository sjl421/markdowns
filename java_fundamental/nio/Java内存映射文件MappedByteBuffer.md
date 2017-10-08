# Java内存映射文件MappedByteBuffer

尽管从JDK 1.4版本开始，Java内存映射文件（Memory Mapped Files）就已经在java.nio包中，但它对很多程序开发者来说仍然是一个相当新的概念。引入NIO后，Java IO已经相当快，而且内存映射文件提供了Java有可能达到的最快IO操作，这也是为什么那些高性能Java应用应该使用内存映射文件来持久化数据。这在一些交易非常频繁的场合已经应用得很多，这些场合要求电子交易系统必须非常快速，单向时延要小于毫秒级。IO一直是那些高性能系统的一个主要关注点，内存映射文件允许你使用direct或者non-direct 字节缓存（Byte buffer）来直接读写内存。内存映射文件的一个关键优势是操作系统负责真正的读写，即使你的程序在刚刚写入内存后就挂了，操作系统仍然会将内存中的数据写入文件系统。另外一个更突出的优势是共享内存，内存映射文件可以被多个进程同时访问，起到一种低时延共享内存的作用。

# 什么是Java内存映射文件/IO

内存映射文件是一种允许Java程序直接从内存访问的特殊文件。通过将整个文件或者文件的一部分映射到内存中、操作系统负责获取页面请求和写入文件，应用程序就只需要处理内存数据，这样可以实现非常快速的IO操作。用于内存映射文件的内存在Java的堆空间以外。Java中的java.nio包支持内存映射文件，可以使用MappedByteBuffer来读写内存。

# 内存映射文件的优缺点

可能内存映射IO的主要优势是性能，内存映射文件比通过普通的IO来访问文件要快，这对于繁忙的电子交易系统来说非常重要。内存映射IO另外一个优势是能够加载普通方式无法访问的大文件，实验表明内存映射IO在大文件处理中表现得更好；但缺点是有增加页面错误（page fault）的可能，因为操作系统仅仅加载一部分文件到内存中，如果被请求的页面不在内存中那就会导致一个页面错误。大多数主流操作系统如Windows, Unix, Solaris和其他类Unix的操作系统都支持内存映射IO，在64位架构下，你几乎可以将任何文件映射到内存中并直接使用Java访问。另外一个优势是这些文件能够共享，在进程间提供共享内存，而且比普通的基于loopback接口的Socket要快10倍。

# Java中MappedByteBuffer读写样例

下面的例子演示了如何使用内存映射文件来读写。我们使用RandomAccessFile打开文件并使用FileChannel的map()方法将它映射到内存，map()方法有三个输入参数：mode, position, size。返回值MappedByteBuffer是用来处理内存映射文件的字节缓存。

```java
import java.io.RandomAccessFile;  
import java.nio.MappedByteBuffer;  
import java.nio.channels.FileChannel;  
  
  
public class MemoryMappedFileInJava {  
  
  
    private static int count = 10485760; // 10 MB  
  
  
    public static void main(String[] args) throws Exception {  
  
  
        RandomAccessFile memoryMappedFile = new RandomAccessFile("largeFile.txt", "rw");  
  
  
        // Mapping a file into memory  
        MappedByteBuffer out = memoryMappedFile.getChannel().map(FileChannel.MapMode.READ_WRITE, 0, count);  
  
  
        // Writing into Memory Mapped File  
        for (int i = 0; i < count; i++) {  
            out.put((byte) 'A');  
        }  
        System.out.println("Writing to Memory Mapped File is completed");  
  
  
        // reading from memory file in Java  
        for (int i = 0; i < 10; i++) {  
            System.out.print((char) out.get(i));  
        }  
        System.out.println("Reading from Memory Mapped File is completed");  
  
  
        memoryMappedFile.close();  
    }  
  
  
```

总结:

下面快速总结一下Java内存映射文件和IO

1）. Java语言通过java.nio包支持内存映射文件和IO。

2）. 内存映射文件用于对性能要求高的系统中，如繁忙的电子交易系统

3）. 使用内存映射IO你可以将文件的一部分加载到内存中

4）. 如果被请求的页面不在内存中，内存映射文件会导致页面错误

5）. 将一个文件区间映射到内存中的能力取决于内存的可寻址范围。在32位机器中，不能超过4GB，即2^32比特。

6）. Java中的内存映射文件比流IO要快(译注：对于大文件而言是对的，小文件则未必）

7）. 用于加载文件的内存在Java的堆内存之外，存在于共享内存中，允许两个不同进程访问文件。顺便说一下，这依赖于你用的是direct还是non-direct字节缓存。

8）. 读写内存映射文件是操作系统来负责的，因此，即使你的Java程序在写入内存后就挂掉了，只要操作系统工作正常，数据就会写入磁盘。

9）. Direct字节缓存比non-direct字节缓存性能要好

10）. 不要经常调用MappedByteBuffer.force()方法，这个方法强制操作系统将内存中的内容写入硬盘，所以如果你在每次写内存映射文件后都调用force()方法，你就不能真正从内存映射文件中获益，而是跟disk IO差不多。

11）. 如果电源故障或者主机瘫痪，有可能内存映射文件还没有写入磁盘，意味着可能会丢失一些关键数据。

12）. MappedByteBuffer和文件映射在缓存被GC之前都是有效的。sun.misc.Cleaner可能是清除内存映射文件的唯一选择。

关于Java内存映射文件和内存映射IO就说这些了。 这相当有用，我希望你能够再深入了解一些。如果你的工作与繁忙的电子交易系统有关，那你很可能用到内存映射文件。

原文链接：http://javarevisited.blogspot.hk/2012/01/memorymapped-file-and-io-in-java.html

译注：与文章主题不太相关的部分内容略有删剪。