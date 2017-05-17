# Java 写自己的 Exception

有时候需要编写自己的Exception类，用来完成更具体的异常处理；

## Java源码的实现

Java中的Exception继承自Throwable对象，而Throwable类实现了Serializable接口;

Java 的源码中Exception类中共有5个Exception的构造器，所以如果构建自己的Exception类需要覆盖这5个构造器；

区别这5个构造器的参数主要是：

```java
String message;	//表示的当前异常的详细信息，通过在抛出异常时由用户设定；
Throwable cause; //对异常进行包装，目的是为了出了问题的时候能够追根究底。
```

>字数太多，发到这里。
>不是为了实现哪一句代码的。initCause()这个方法就是对异常来进行包装的，目的就是为了出了问题的时候能够追根究底。因为一个项目，越往底层，可能抛出的异常类型会用很多，如果你在上层想要处理这些异常，你就需要挨个的写很多catch语句块来捕捉异常，这样是很麻烦的。如果我们对底层抛出的异常捕获后，抛出一个新的统一的异常，会避免这个问题。但是直接抛出一个新的异常，会让最原始的异常信息丢失，这样不利于排查问题。举个例子，在底层会出现一个A异常，然后在中间代码层捕获A异常，对上层抛出一个B异常。如果在中间代码层不对A进行包装，在上层代码捕捉到B异常后就不知道为什么会导致B异常的发生，但是包装以后我们就可以用getCause()方法获得原始的A异常。这对追查BUG是很有利的。
>
>```
>class A{
>    try{
>        ...
>    }catch(AException a){
>     throw new BException();
>    }
>}
>...
>class B{
>    try{
>        ...
>    }catch(BException b){
>        //这时候你需要去看b异常式什么问题导致的，你在A类里面
>        //没有对AException进行包装，所以你无法知道是A导致的B
>    }
>}
>```
>
>如果包装以后：
>
>```java
>class A{
>    try{
>        ...
>    }catch(AException a){
>        BException b = new BEexception()；
>        b.initCause(a);
>        throw b;
>    }
>}
>...
>class B{
>    try{
>        ...
>    }catch(BException b){
>        //什么导致了b呢？
>        b.getCause()；//得到导致B异常的原始异常
>    }
>}
>```

### Java 实现的5个构造器

1. 默认构造器：

   ```java
   public Exception(){
     super();
   }
   ```

   Java的默认构造器，没有设置详细的异常信息，同时cause 参数也没有显式初始化cause; 

2. 带有message参数：

   ```java
   public Exception(String message){
     super(message);
   }
   ```

3. 带有message 和cause参数：

   ```java
   public Excepion(String message, Throwable cause){
     super(message, cause);
   }
   ```

4. 带有cause参数：

   ```java
   public Exception(Throwable cause){
     super(casue);
   }
   ```

5. 带有很多的参数：

   ```java
   protected Exception(String message, Throwalbe cause, boolean enableSuppression, boolean writableStackTrace){
     super(message, cause, enableSupperssion, writableStackTrace);
   }
   ```

### 实现自己Exception

构建自己的Exception需要将上面提到的5个构造器全部重写，只有这样才能保证自己在使用自己的Exception的时候能够和java系统无缝的对接起来（这个是自己在重写java内置方法时都应该遵循的原则）；

其实实现的Exception是比较简单的，就让自己的Exception 集成 Java的Exception，然后像Java 自己的Exception实现那样就可以了的；

```java
    later retrieval by the {@link #getMessage()} method.
     */
    public InvaildReqException(String message) {
        super(message);
    }

    /**
     * Constructs a new exception with the specified detail message and
     * cause.  <p>Note that the detail message associated with
     * {@code cause} is <i>not</i> automatically incorporated in
     * this exception's detail message.
     *
     * @param message the detail message (which is saved for later retrieval
     *                by the {@link #getMessage()} method).
     * @param cause   the cause (which is saved for later retrieval by the
     *                {@link #getCause()} method).  (A <tt>null</tt> value is
     *                permitted, and indicates that the cause is nonexistent or
     *                unknown.)
     * @since 1.4
     */
    public InvaildReqException(String message, Throwable cause) {
        super(message, cause);
    }

    /**
     * Constructs a new exception with the specified cause and a detail
     * message of <tt>(cause==null ? null : cause.toString())</tt> (which
     * typically contains the class and detail message of <tt>cause</tt>).
     * This constructor is useful for exceptions that are little more than
     * wrappers for other throwables (for example, {@link
     * PrivilegedActionException}).
     *
     * @param cause the cause (which is saved for later retrieval by the
     *              {@link #getCause()} method).  (A <tt>null</tt> value is
     *              permitted, and indicates that the cause is nonexistent or
     *              unknown.)
     * @since 1.4
     */
    public InvaildReqException(Throwable cause) {
        super(cause);
    }

    /**
     * Constructs a new exception with the specified detail message,
     * cause, suppression enabled or disabled, and writable stack
     * trace enabled or disabled.
     *
     * @param message            the detail message.
     * @param cause              the cause.  (A {@code null} value is permitted,
     *                           and indicates that the cause is nonexistent or unknown.)
     * @param enableSuppression  whether or not suppression is enabled
     *                           or disabled
     * @param writableStackTrace whether or not the stack trace should
     *                           be writable
     * @since 1.7
     */
    public InvaildReqException(String message, Throwable cause, boolean enableSuppression, boolean writableStackTrace) {
        super(message, cause, enableSuppression, writableStackTrace);
    }
}
```

也许会有人文你这样实现的跟调用java自己的Exception会有什么用，这个问题又回到了那个很普遍的问题，我们实现了自己的异常，这样等到我们在将来的某个时刻需要更新自己的异常时只需要更新InvaidReqException类就可以了，客户端的代码（或者使用InvalidReqException的代码）不需要做任何修改；