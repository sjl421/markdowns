# 理解PostgreSQL数据库、模式、表、空间、用户间的关系

**数据库中的schema和tablespaces**

什么是schema?
这里只讨论数据库中的schema，而不讨论XML中的schema。在wiki上，这样解释schema：
In a relational database, the schema defines the tables, views, indexes, packages,procedures, functions, queues, triggers, types, sequences, materialized views, synonyms,database links, directories, Java, XML schemas, and other elements.

而实际上，schema就是数据库对象的集合。

我们先来看一下schema的定义：
A schema is a collection of database objects (used by a user.).Schema objects are the logical structures that directly refer to the database’s data.A user is a name defined in the database that can connect to and access objects.Schemas and users help database administrators manage database security.

从定义中我们可以看出schema为数据库对象的集合，为了区分各个集合，我们需要给这个集合起个名字，这些名字就是我们在企业管理器的方案下看到的许多类似用户名的节点，这些类似用户名的节点其实就是一个schema，schema里面包含了各种对象如tables, views, sequences, stored procedures, synonyms, indexes, clusters, and database links。

一个用户一般对应一个schema，该用户的schema名等于用户名，并作为该用户缺省schema。这也就是我们在企业管理器的方案下看到schema名都为数据库用户名的原因。Oracle数据库中不能新创建一个schema，要想创建一个schema，只能通过创建一个用户的方法解决(Oracle中虽然有create schema语句，但是它并不是用来创建一个schema的)，在创建一个用户的同时为这个用户创建一个与用户名同名的schem并作为该用户的缺省shcema。即schema的个数同user的个数相同，而且schema名字同user名字一一对应并且相同，所有我们可以称schema为user的别名，虽然这样说并不准确，但是更容易理解一些。

一个用户有一个缺省的schema，其schema名就等于用户名，当然一个用户还可以使用其他的schema。如果我们访问一个表时，没有指明该表属于哪一个schema中的，系统就会自动给我们在表上加上缺省的sheman名。比如我们在访问数据库时，访问freeoa用户下的emp表，通过select * from emp; 其实，这sql语句的完整写法为select * from freeoa.emp。在数据库中一个对象的完整名称为schema.object，而不属user.object。类似如果我们在创建对象时不指定该对象的schema，在该对象的schema为用户的缺省schema。这就像一个用户有一个缺省的表空间，但是该用户还可以使用其他的表空间，如果我们在创建对象时不指定表空间，则对象存储在缺省表空间中，要想让对象存储在其他表空间中，我们需要在创建对象时指定该对象的表空间。

为什么schema有存在的必要？
为了区分各个集合，我们需要给这个集合起个名字，其实这个名字就是schema。

举例说明：访问freeoa用户下的emp表，通过select from emp 其实这条sql语句的完整写法为select from freeoa.emp。对于数据库来说，不同的用户，有不同schema。有不同的表。实际在使用上，schema和user完全一样，没有什么区别，在出现schema名的地方也可以出现user名。

什么是模式？
数据库中的模式指的就是schema。可以在不同模式下创建相同表名，访问表对象时使用模式名.表对象，对于不指明模式的表对象以当前登录用户模式作为隐含模式访问。

什么是表空间？
表空间是实际的数据存储的地方。一个数据库schema可能存在于多个表空间，相似地，一个表空间也可以为多个schema服务。

表空间的作用：
通过使用表空间，管理员可以控制磁盘的布局。表空间的最常用的作用是优化性能，例如，一个最常用的索引可以建立在非常快的硬盘上，而不太常用的表可以建立在便宜的硬盘上，比如用来存储用于进行归档文件的表。

----------------------------------------------------------------------------------------
**PostgreSQL表空间、数据库、模式、表、用户、角色之间的关系**

**角色与用户的关系**

在PostgreSQL中，存在两个容易混淆的概念：角色/用户。之所以说这两个概念容易混淆，是因为对于PostgreSQL来说，这是完全相同的两个对象。唯一的区别是在创建的时候：

1.我用下面的psql创建了角色freeoa：
CREATE ROLE freeoa PASSWORD 'freeoa';
接着我使用新创建的角色freeoa登录，PostgreSQL给出拒绝信息：

FATAL：role 'freeoa' is not permitted to log in.
说明该角色没有登录权限，系统拒绝其登录。

2.我又使用下面的psql创建了用户freeoa2:
CREATE USER freeoa2 PASSWORD 'freeoa2';
接着我使用freeoa2登录，登录成功。难道这两者有区别吗？查看文档，又这么一段说明："CREATE USER is the same as CREATE ROLE except that it implies LOGIN."----CREATE USER除了默认具有LOGIN权限之外，其他与CREATE ROLE是完全相同的。

为了验证这句话，修改freeoa的权限，增加LOGIN权限：ALTER ROLE freeoa LOGIN;再次用freeoa登录，成功！那么事情就明了了：CREATE ROLE freeoa PASSWORD 'freeoa' LOGIN 等同于CREATE USER freeoa PASSWORD 'freeoa'.这就是ROLE/USER的区别。

**数据库与模式的关系**

模式(schema)是对数据库(database)逻辑分割。

在数据库创建的同时，就已经默认为数据库创建了一个模式--public，这也是该数据库的默认模式。所有为此数据库创建的对象(表、函数、试图、索引、序列等)都是常见在这个模式中的：
1.创建一个数据库dba----CREATE DATABASE dba;

2.用freeoa角色登录到dbtt数据库,查看dbtt数据库中的所有模式：\dn; 显示结果是只有public一个模式。

3.创建一张测试表----CREATE TABLE test(id integer not null);

4.查看当前数据库的列表：\d; 显示结果是表test属于模式public.也就是test表被默认创建在了public模式中。

5.创建一个新模式freeoa，对应于登录用户freeoa：CREATE SCHEMA freeoa OWNER freeoa;

6.再次创建一张test表，这次这张表要指明模式----CREATE TABLE freeoa.test (id integer not null);

7.查看当前数据库的列表：\d; 显示结果是表test属于模式freeoa.也就是这个test表被创建在了freeoa模式中。

得出结论是：数据库是被模式(schema)来切分的，一个数据库至少有一个模式，所有数据库内部的对象(object)是被创建于模式的。用户登录到系统，连接到一个数据库后，是通过该数据库的search_path来寻找schema的搜索顺序，可以通过命令SHOW search_path；具体的顺序，也可以通过SET search_path TO 'schema_name'来修改顺序。

官方建议是这样的：在管理员创建一个具体数据库后，应该为所有可以连接到该数据库的用户分别创建一个与用户名相同的模式，然后将search_path设置为"$user"，这样，任何当某个用户连接上来后，会默认将查找或者定义的对象都定位到与之同名的模式中。对于一个应用程序分很多相对独立的模块，每个模块有相对独立的数据结构，可以采用每个模块一个数据库用户及与其名字相同的schema来组织数据库，并且整个的物理数据库放在一个单独的表空间中。使用这种数据库管理模式，可以撤销掉对public schema的访问许可，甚至把public schema直接移除，这样每个用户就真正的限定在了他们自己的schema里。

**表空间与数据库的关系**

数据库创建语句CREATE DATABASE dbname 默认的数据库所有者是当前创建数据库的角色，默认的表空间是系统的默认表空间--pg_default。

为什么是这样的呢？因为在PostgreSQL中，数据的创建是通过克隆数据库模板来实现的，这与SQL SERVER是同样的机制。由于CREATE DATABASE dbname并没有指明数据库模板，所以系统将默认克隆template1数据库，得到新的数据库dbname。(By default, the new database will be created by cloning the standard system database template1).

而template1数据库的默认表空间是pg_default，这个表空间是在数据库初始化时创建的，所以所有template1中的对象将被同步克隆到新的数据库中。相对完整的语法应该是这样的：CREATE DATABASE dbname OWNER freeoa TEMPLATE template1 TABLESPACE tablespacename;

1.连接到template1数据库，创建一个表作为标记：CREATE TABLE tbl_flag(id integer not null);向表中插入数据INSERT INTO tbl_flag VALUES (1);

2.创建一个表空间:CREATE TABLESPACE tsfreeoa OWNER freeoa LOCATION '/tmp/data/tsfreeoa';在此之前应该确保目录/tmp/data/tsfreeoa存在，并且目录为空。

3.创建一个数据库，指明该数据库的表空间是刚刚创建的tsfreeoa：CREATE DATABASE dbfreeoa TEMPLATE template1 OWNERE freeoa TABLESPACE tsfreeoa;

4.查看系统中所有数据库的信息：\l；可以发现，dbfreeoa数据库的表空间是tsfreeoa,拥有者是freeoa;

5.连接到dbfreeoa数据库，查看所有表结构:\d;可以发现，在刚创建的数据库中居然有了一个表tbl_flag,查看该表数据，输出结果一行一列，其值为1，说明，该数据库的确是从template1克隆而来。

仔细分析后，不难得出结论：在PostgreSQL中，表空间是一个目录，里面存储的是它所包含的数据库的各种物理文件。

**总结一下它们之间的关系**

表空间是一个存储区域，在一个表空间中可以存储多个数据库，尽管PostgreSQL不建议这么做，但我们这么做完全可行。一个数据库并不知直接存储表结构等对象的，而是在数据库中逻辑创建了至少一个模式，在模式中创建了表等对象，将不同的模式指派该不同的角色，可以实现权限分离，又可以通过授权，实现模式间对象的共享，并且还有一个特点就是：public模式可以存储大家都需要访问的对象。

既然一个表在创建的时候可以指定表空间，那么是否可以给一个表指定它所在的数据库表空间之外的表空间呢？

答案是肯定的，这么做完全可以：那这不是违背了表属于模式，而模式属于数据库，数据库最终存在于指定表空间这个网的模型了吗？

是的，看上去这确实是不合常理的，但这么做又是有它的道理的，而且现实中，我们往往需要这么做：将表的数据存在一个较慢的磁盘上的表空间，而将表的索引存在于一个快速的磁盘上的表空间。但我们再查看表所属的模式还是没变的，它依然属于指定的模式。所以这并不违反常理。实际上PostgreSQL并没有限制一张表必须属于某个特定的表空间，我们之所以会这么认为，是因为在关系递进时，偷换了一个概念：模式是逻辑存在的，它不受表空间的限制。

表空间、数据库、角色、模式及表之间的关系

表空间用于定义数据库对象在物理存储设备上的位置，不特定于某个单独的数据库。数据库是数据库对象的物理集合，而schema则是数据库内部用于组织管理数据库对象的逻辑集合，schema名字空间之下则是各种应用程序会接触到的对象，比如表、索引、数据类型、函数、操作符等。

角色(用户)则是数据库服务器(集群)全局范围内的权限控制系统，用于各种集群范围内所有的对象权限管理。因此角色不特定于某个单独的数据库，但角色如果需要登录数据库管理系统则必须连接到一个数据库上。角色可以拥有各种数据库对象。对模式还是有疑问，那继续读下面的章节吧。

**---------------------------------再说一遍模式(Schema)**

一个 PostgreSQL 数据库集群包含一个或多个命名的数据库。用户和用户组在整个集群的范围内是共享的，但是其它数据并不是共享的。任何给定的与服务器的客户连接都只能访问在一个数据库里的数据，就是那个在连接请求里声明的。

注意：一个集群的用户并不一定要有访问集群内所有数据库的权限。 共享用户名的意思是不能有同名用户，也就是在同一个集群里的两个数据库里都有叫 oafree 的用户，但是系统可以配置成只允许 oafree 访问某些数据库。

一个数据库包含一个或多个命名的模式，模式又包含表。模式还包含其它命名的对象，包括数据类型、函数、序列以及操作符。同一个对象名可以在不同的模式里使用而不会导致冲突； 比如，schema1 和 myschema 都可以包含叫做 mytable 的表。和数据库不同，模式不是严格分离的：一个用户可以访问他所连接的数据库中的任意模式中的对象，只要他有权限。

我们需要模式的原因有好多：

*允许多个用户使用一个数据库而不会干扰其它用户。把数据库对象组织成逻辑组，让它们更便于管理。第三方的应用可以放在不同的模式中， 这样它们就不会和其它对象的名字冲突。模式类似于操作系统层次的目录，只不过模式不能嵌套。*

创建一个schema

创建一个模式(schema)使用CREATE SCHEMA命令，如：
create schema freeoa_schema;

在指定模式里创建表，如：
CREATE TABLE myschema.mytable (
...
);

删除一个空的schema，如：
drop schema myschema;

删除一个模式以及模式里面所有的对象，如：
drop schema myschema CASCADE;

**pulic schema(public 模式)**

在创建表时，如果没有指定schema，则表会自动被归属到一个叫做'public‘的模式中，每一个数据库中都会有一个这样的模式，它会自动创建。下面两种创建表的方式是等效的：

CREATE TABLE tableName(...);
和
CREATE TABLE public.tableName(...);

**模式和权限**

缺省时，用户看不到模式中不属于他们所有的对象。为了让他们看得见，模式的所有者需要在模式上赋予 USAGE 权限。为了让用户使用模式中的对象，我们可能需要赋予额外的权限， 只要是适合该对象的。

用户也可以允许在别人的模式里创建对象。要允许这么做， 我们需要赋予在该模式上的 CREATE 权限。 请注意，缺省每个人都在 public 模式上 有 CREATE 权限。这样就允许所有可以连接到 指定数据库上的用户在这里创建对象。如果你不允许这么做， 你可以撤销这个权限：
REVOKE CREATE ON public FROM PUBLIC;

(第一个 "public" 是模式，第二个 "public" 意思是"所有用户"。 第一句里它是个标识符，而第二句里是个关键字，所以有不同的大小写)

**模式搜索路径**

全称的名字写起来非常费劲，并且我们最好不要在应用里直接 写上特定的模式名。因此，表通常都是用未修饰的名字引用的，这样的名字里只有表名字。系统通过查找一个搜索路径 来判断一个表究竟是哪个表，这个路径是一个需要查找的模式列表。在搜索路径里找到的第一个表将被当作选定的表。如果在搜索路径中没有匹配表，那么就报告一个错误，即使匹配表的名字在数据库其它的 模式中存在也如此。

在搜索路径中的第一个模式叫做当前模式。除了是搜索的第一个模式之外， 它还是在 CREATE TABLE 没有声明模式名的时候，新建表所在的地方。这个与shell的PATH变量很相似。

查看当前搜索路径，使用命令：
SHOW search_path;

在缺省的设置中，返回下面的东西：
 search_path
--------------
 $user,public

第一个值声明将要搜索一个和当前用户同名的模式。 因为还没有这样的模式存在，所以这条记录被忽略。第二个值指向public模式。

要把新的模式放到路径中来，我们用
SET search_path TO myschema,public;

**使用方式**

模式可以以多种方式组织你的数据。下面是一些建议使用的模式， 它们也很容易在缺省配置中得到支持：
如果你没有创建任何模式，那么所有用户隐含都访问public模式。这样就模拟了还没有模式的时候的情景。这种设置建议主要用在只有一个用户或者数据库里只有几个合作用户的情形。这样的设置也允许我们平滑地从无模式的环境过渡。

你可以为每个用户创建一个模式，名字和用户相同。要记得缺省的搜索路径从$user开始，它会解析为用户名。如果每个用户都有一个独立的模式，那么他们缺省时访问他们自己的模式。如果你使用了这样的设置，那么你可能还想撤销对public模式的访问(或者一并删除)，因此用户就真的限制于他们自己的模式中了。

要安装共享的应用(被所有人使用的表，第三方提供的额外的函数等等)，可以把它们放到独立的模式中，只要记得给访问它们的用户赋予合适的权限就可以了。然后用户就可以通过用一个模式名修饰这些名字来使用这些额外的对象，或者可以把额外的模式放到他们的搜索路径中，由用户来定。