# Http 读取 Request Body

在http请求中，有Header和body之分，读取header使用request.getHeader("..."),;

读取Body使用request.getReader(), 但是getReader获取的是BufferReader，需要把他转换成字符串：

```java
  public static String getBodyString(BufferedReader br) {
  	String inputLine;
    String str = "";
     try {
       while ((inputLine = br.readLine()) != null) {
        str += inputLine;
       }
       br.close();
     } catch (IOException e) {
       System.out.println("IOException: " + e);
     }
     return str;
 }
```

注：request能通过getxxx获取不同类型的reader和writer，到时候可以更具合适的应用场景来获取合适的reader和writer;

```java
import com.google.gson.Gson;
import org.apache.log4j.Logger;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.io.InputStream;
import java.io.PrintWriter;

public class HttpUtil {
    private static Gson gson = new Gson();
    private static Logger logger = Logger.getLogger(HttpUtil.class);


    public static void output(HttpServletResponse response, String dataOutput) {
        try {
            response.setCharacterEncoding("UTF-8");
            response.setContentType("text/html");
            PrintWriter out = response.getWriter();
            out.print(dataOutput);
            out.flush();
        } catch (Exception e) {
            logger.info(e.getStackTrace());
        }
    }

    public static void output(HttpServletResponse response, CommonResponse cr) {
        try {
            output(response,gson.toJson(cr));
        } catch (Exception e) {
            logger.info(e.getStackTrace());
        }
    }

    public static String readToString(HttpServletRequest request) {
        InputStream in = null;
        String str = "";
        byte[] bytes = new byte[1024];
        StringBuilder sb = new StringBuilder();
        try {
            in = request.getInputStream();
            while(true) {
                int len = in.read(bytes);
                if(len<0) {
                    break;
                }
                str = new String(bytes,0,len);
                sb.append(str);
                bytes = new byte[1024];
                str = "";
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
        return sb.toString();
    }
}
```

可以将读取request body和写入response body的操作封装成一个方法，然后在java web的项目中使用；