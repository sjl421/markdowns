# Get传递多个不定数目参数

## 问题描述：

就是在一次GET请求中传递给server端不定数目的参数，此时可以有两种情况：

1. 参数同名： 即一次传递多个同名参数，例如ID，Client一次请求多个ID不同的同类型资源，需要将所有要请求的ID都传递过来，此时ID是同一中参数类型，但是所对应的值却不同；
2. 参数不同名：这种是在第一种情况的基础上使得参数不相同，即同时传递读多个不同的参数，参数的个数不定，每个参数所对应的值的数目也不定：有多个参数，每个参数又对应多个值；

## 解决方法

Tomcat的HttpServletRequest中提供了几个相关的方法：

先看官方的文档：

| ` java.lang.String`      | `getParameter(java.lang.String name)`           Returns the value of a request parameter as a `String`, or `null` if the parameter does not exist. |
| ------------------------ | ---------------------------------------- |
| ` java.util.Map`         | `getParameterMap()`           Returns a java.util.Map of the parameters of this request. |
| ` java.util.Enumeration` | `getParameterNames()`           Returns an `Enumeration` of `String` objects containing the names of the parameters contained in this request. |
| ` java.lang.String[]`    | `getParameterValues(java.lang.String name)`           Returns an array of `String` objects containing all of the values the given request parameter has, or `null` if the parameter does not exist. |

1. getparameter(String name) 获取指定针对给定的参数名所对应的参数，此时这个参数对应一个值；
2. getpameterMap() 将Request中所有参数以Map\<key, value\>的形式返回；
3. getParameterName() : 将Request中所有的参数名以一个Enumeration的形式返回，此时包含了所有参数的名字；
4. getParameterValues(String name): 就Request中指定名字（name）所有对应的参数以String数组的形式返回；

可以看到，方法4，方法3是我们解决这个问题的关键：

* 当只有参数时，我们可以直接通过方法4来得到所有的值，
* 当有多个参数时，我们可以先通过方法3来得到有的参数名字，然后在通过方法4类得到每个参数名进而对应的其所对应的值的String数组；

```java
<form action="request04.jsp" method="post">  
    姓名： <input type="text" name="name"><br>  
    兴趣： <input type="checkbox" name="**inst" value="游泳">游泳  
            <input type="checkbox" name="**inst" value="唱歌">唱歌  
            <input type="checkbox" name="**inst" value="跳舞">跳舞  
    <br><input type="submit" value="显示">  
    <input type="hidden" name="info" value="MLDN">  
</form>  
```

```java
<%@ page contentType="text/html;charset=GBK"%>  
<%@ page import="java.util.*"%>  
<%  
    request.setCharacterEncoding("GBK") ;           // 按中文接收  
    Enumeration enu = request.getParameterNames() ; // 接收参数的名称  
%>  
<%  
    while(enu.hasMoreElements()){  
        String paramName = (String)enu.nextElement() ;  
%>  
        <h2><%=paramName%>  
            -->   
<%  
        if(paramName.startsWith("**")){  
            // 按数组接收  
            String temp[] = request.getParameterValues(paramName) ;  
            for(int i=0;i<temp.length;i++){  
%>  
                <%=temp[i]%>、  
<%  
            }  
        }else{  
%>  
            <%=request.getParameter(paramName)%>  
<%  
        }  
%>  
        </h2>  
<%  
    }  
%>  
```



## 补充：

request中有getParameter() 和 getAttribute()方法，其中getAttribute()有对应的setAttibute()方法，但是getParameter()方法没有对应的setAttribute()方法；

同时可以查一下两个方法的不同来完整这份文档；

