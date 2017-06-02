 # postgresql使用jdbc连接过程的分析



本人在开发pgoneproxy的过程中,需要实现前端登录到pgoneproxy，pgoneproxy针对前端进行校验，校验通过后才能使用连接池中的连接，而连接池中的连接是pgoneproxy通过发送数据包的方式进行的连接。

​    下图是客户端与服务端在建立连接过程中，发送的数据包的情况：

![img](http://al.efabu.net/d/file/PostgreSQL/2016-09-18/ownptblgx4c.png)

​    由于postgresql 9.4已经支持SSL，故jdbc会先来请求是否允许使用SSL的方式进行验证。如果不允许使用SSL的方式进行验证，则服务端会发送一个N字符给客户端，客户端再发送startupMessage包给服务端。本文只讲解非SSL连接的情况，因为pgoneproxy只支持非SSL连接。

​    在startupMessage包中包含了用户名，数据库或者其他的参数名以及这些参数的值。服务端在接收到这些参数时，会根据服务端配置的验证方式向客户端发送Authentication包。目前postgresql支持的验证方式有：Kerberos V5，明文的方式，MD5加密， SCM 方式，GSSAPI认证方式，SSPI认证方式，GSSAPI或者SSPI方式。它们之间的区别通过Authentication包中的第5个字节开始的4个字节进行标识。目前pgoneproxy已经支持明文，trust，MD5的验证方式。

​     客户端在接收到Authentication包后，根据相应的规则生成密码，通过PasswordMessage数据包发送到服务端，服务端进行验证。如果验证通过，则发送AuthenticationOK包给客户端，紧接着发送参数包，BackendKey包给客户端。其中backendkey包中包含了服务端进程的pid以及后端的秘钥，当前端需要取消一个连接时就会使用这个秘钥和pid来进行区分后端连接。如果验证失败，则会直接关闭此连接。