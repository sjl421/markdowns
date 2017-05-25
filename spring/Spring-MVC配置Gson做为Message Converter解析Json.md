# Spring-MVC配置Gson做为Message Converter解析Json

在学习Spring的时候看到可以使用@RequestBody 和@ResponseBody注解来是的Spring自动将http 其中的 body（json格式）部分和java内部的类进行转换。同时由于Google Gson的强大一般开发的时候会用的比较多，但是由于Spring内部默认使用的json的message Converter 并不是gson，所以这里需要配置一下才能使其生效；

Spring其实已经在实现了对gson的支持，但是如果想在项目中使用，还需要通过配置一下才可以：

配置文件如下：

```xml
    <!--<mvc:annotation-driven/>-->
	<mvc:annotation-driven>
        <mvc:message-converters>
            <bean class="org.springframework.http.converter.json.GsonHttpMessageConverter"/>
        </mvc:message-converters>
    </mvc:annotation-driven>
```

其中`org.springframework.http.converter.json.GsonHttpMessageConverter`是Spring已经实现了的，在spring的包中能找到这个类。

注：如果想要某中类型的数据能被Message Converter转换，需要自己实现`AbstractGenericHttpMessageConverter`这个接口，然后在通过配置文件将这个类加载进你的项目中（比如说alibaba:fastjson，在它包中也实现了度spring message converter 的支持）。



备注： 网上有很多实现自己的Message Converter的教程，如果后续需要可以搜一下，实现起来有两个步骤：

1. 需要自己实现`AbstractGenericHttpMessageConverter`这个接口；

2. 通过xml或者其他方法配置一下

   ```xml
   <mvc:annotation-driven>
       <mvc:message-converters>
           <bean class="org.springframework.http.converter.json.GsonHttpMessageConverter"/>
       </mvc:message-converters>
   </mvc:annotation-driven>

   #在pom下加入依赖
   <dependency>
       <groupId>com.google.code.gson</groupId>
       <artifactId>gson</artifactId>
       <version>2.3.1</version>
   </dependency>
   ```

   ```java
   @Configuration
   @ConditionalOnClass(Gson.class)
   @ConditionalOnMissingClass(name = "com.fasterxml.jackson.core.JsonGenerator")
   @ConditionalOnBean(Gson.class)
   protected static class GsonHttpMessageConverterConfiguration {

       @Bean
       @ConditionalOnMissingBean
       public GsonHttpMessageConverter gsonHttpMessageConverter(Gson gson) {
           GsonHttpMessageConverter converter = new GsonHttpMessageConverter();
           converter.setGson(gson);
           return converter;
       }

   }

   <dependency>
       <groupId>com.google.code.gson</groupId>
       <artifactId>gson</artifactId>
       <version>2.3.1</version>
   </dependency>
   ```

3. 一下贴点源码：

   ```java
   package org.springframework.http.converter.json;

   import com.google.gson.Gson;
   import com.google.gson.JsonIOException;
   import com.google.gson.JsonParseException;
   import com.google.gson.reflect.TypeToken;
   import java.io.IOException;
   import java.io.InputStreamReader;
   import java.io.OutputStreamWriter;
   import java.lang.reflect.Type;
   import java.nio.charset.Charset;
   import org.springframework.http.HttpHeaders;
   import org.springframework.http.HttpInputMessage;
   import org.springframework.http.HttpOutputMessage;
   import org.springframework.http.MediaType;
   import org.springframework.http.converter.AbstractGenericHttpMessageConverter;
   import org.springframework.http.converter.HttpMessageNotReadableException;
   import org.springframework.http.converter.HttpMessageNotWritableException;
   import org.springframework.util.Assert;

   public class GsonHttpMessageConverter extends AbstractGenericHttpMessageConverter<Object> {
       public static final Charset DEFAULT_CHARSET = Charset.forName("UTF-8");
       private Gson gson = new Gson();
       private String jsonPrefix;

       public GsonHttpMessageConverter() {
           super(new MediaType[]{MediaType.APPLICATION_JSON, new MediaType("application", "*+json")});
           this.setDefaultCharset(DEFAULT_CHARSET);
       }

       public void setGson(Gson gson) {
           Assert.notNull(gson, "'gson' is required");
           this.gson = gson;
       }

       public Gson getGson() {
           return this.gson;
       }

       public void setJsonPrefix(String jsonPrefix) {
           this.jsonPrefix = jsonPrefix;
       }

       public void setPrefixJson(boolean prefixJson) {
           this.jsonPrefix = prefixJson?")]}', ":null;
       }

       public boolean canRead(Class<?> clazz, MediaType mediaType) {
           return this.canRead(mediaType);
       }

       public boolean canWrite(Class<?> clazz, MediaType mediaType) {
           return this.canWrite(mediaType);
       }

       protected boolean supports(Class<?> clazz) {
           throw new UnsupportedOperationException();
       }

       protected Object readInternal(Class<?> clazz, HttpInputMessage inputMessage) throws IOException, HttpMessageNotReadableException {
           TypeToken<?> token = this.getTypeToken(clazz);
           return this.readTypeToken(token, inputMessage);
       }

       public Object read(Type type, Class<?> contextClass, HttpInputMessage inputMessage) throws IOException, HttpMessageNotReadableException {
           TypeToken<?> token = this.getTypeToken(type);
           return this.readTypeToken(token, inputMessage);
       }

       protected TypeToken<?> getTypeToken(Type type) {
           return TypeToken.get(type);
       }

       private Object readTypeToken(TypeToken<?> token, HttpInputMessage inputMessage) throws IOException {
           InputStreamReader json = new InputStreamReader(inputMessage.getBody(), this.getCharset(inputMessage.getHeaders()));

           try {
               return this.gson.fromJson(json, token.getType());
           } catch (JsonParseException var5) {
               throw new HttpMessageNotReadableException("Could not read JSON: " + var5.getMessage(), var5);
           }
       }

       private Charset getCharset(HttpHeaders headers) {
           return headers != null && headers.getContentType() != null && headers.getContentType().getCharset() != null?headers.getContentType().getCharset():DEFAULT_CHARSET;
       }

       protected void writeInternal(Object o, Type type, HttpOutputMessage outputMessage) throws IOException, HttpMessageNotWritableException {
           Charset charset = this.getCharset(outputMessage.getHeaders());
           OutputStreamWriter writer = new OutputStreamWriter(outputMessage.getBody(), charset);

           try {
               if(this.jsonPrefix != null) {
                   writer.append(this.jsonPrefix);
               }

               if(type != null) {
                   this.gson.toJson(o, type, writer);
               } else {
                   this.gson.toJson(o, writer);
               }

               writer.close();
           } catch (JsonIOException var7) {
               throw new HttpMessageNotWritableException("Could not write JSON: " + var7.getMessage(), var7);
           }
       }
   }
   ```

4. 然后是alibaba:fastjson的实现

   ```json
   package com.alibaba.fastjson.support.spring;

   import com.alibaba.fastjson.JSON;
   import com.alibaba.fastjson.parser.Feature;
   import com.alibaba.fastjson.serializer.SerializerFeature;
   import java.io.ByteArrayOutputStream;
   import java.io.IOException;
   import java.io.InputStream;
   import java.io.OutputStream;
   import java.nio.charset.Charset;
   import org.springframework.http.HttpInputMessage;
   import org.springframework.http.HttpOutputMessage;
   import org.springframework.http.MediaType;
   import org.springframework.http.converter.AbstractHttpMessageConverter;
   import org.springframework.http.converter.HttpMessageNotReadableException;
   import org.springframework.http.converter.HttpMessageNotWritableException;

   public class FastJsonHttpMessageConverter extends AbstractHttpMessageConverter<Object> {
       public static final Charset UTF8 = Charset.forName("UTF-8");
       private Charset charset;
       private SerializerFeature[] features;

       public FastJsonHttpMessageConverter() {
           super(new MediaType[]{new MediaType("application", "json", UTF8), new MediaType("application", "*+json", UTF8)});
           this.charset = UTF8;
           this.features = new SerializerFeature[0];
       }

       protected boolean supports(Class<?> clazz) {
           return true;
       }

       public Charset getCharset() {
           return this.charset;
       }

       public void setCharset(Charset charset) {
           this.charset = charset;
       }

       public SerializerFeature[] getFeatures() {
           return this.features;
       }

       public void setFeatures(SerializerFeature... features) {
           this.features = features;
       }

       protected Object readInternal(Class<? extends Object> clazz, HttpInputMessage inputMessage) throws IOException, HttpMessageNotReadableException {
           ByteArrayOutputStream baos = new ByteArrayOutputStream();
           InputStream in = inputMessage.getBody();
           byte[] buf = new byte[1024];

           while(true) {
               int len = in.read(buf);
               if(len == -1) {
                   byte[] bytes = baos.toByteArray();
                   return JSON.parseObject(bytes, 0, bytes.length, this.charset.newDecoder(), clazz, new Feature[0]);
               }

               if(len > 0) {
                   baos.write(buf, 0, len);
               }
           }
       }

       protected void writeInternal(Object obj, HttpOutputMessage outputMessage) throws IOException, HttpMessageNotWritableException {
           OutputStream out = outputMessage.getBody();
           String text = JSON.toJSONString(obj, this.features);
           byte[] bytes = text.getBytes(this.charset);
           out.write(bytes);
       }

   ```



## 以下是在解决这个问题时搜到的比较有用的帖子

[Jackson](http://jackson.codehaus.org/) has been the default json library in springframework until version 4.1 where it added support to use Gson by configuring GsonHttpMessageConverter. Let's take a look at how to configure your spring application to use [Google Gson library's Gson class](https://code.google.com/p/google-gson/).

### Detailed Video Notes

[Gson](https://code.google.com/p/google-gson/) is a java based library that converts java objects into their [JSON](http://www.json.org/) representation and vice versa. A common example is when you are [create a REST end point](https://www.leveluplunch.com/java/tutorials/021-consume-get-request-spring-rest-webservice-jquery/) via a spring `@Controller` where you fetch a [list of objects that you want to convert into an jsonarray](https://www.leveluplunch.com/java/examples/convert-json-array-to-arraylist-gson/). Let's examine a few different configuration options within your spring boot, java config and xml based applications.

## Getting started

[[0:21](http://www.youtube.com/embed/otV5Wqfy_qU?start=21&autoplay=1)]

Before we get started it is important to understand how spring converts objects to json. In typical spring mvc once request exits the `@Controller` it looks for a view to render. When specifying a `@RequestBody` or a `@RestController` we ask spring to by pass this step and write the java objects in the model directly to the response. When doing so spring will look specifically for a `HttpMessageConverter` associated to the mime type to perform the conversion and in our case Json. Spring configures [`MappingJackson2HttpMessageConverter`](http://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/http/converter/json/MappingJackson2HttpMessageConverter.html) by default so we want to swap it with `GsonHttpMessageConverter` so it uses Gson to convert the java objects.

For our first snippet, lets generate a spring boot application from [spring initializr](http://start.spring.io/) web page and import it into eclipse.

## Configuring gson in spring boot

[[1:1](http://www.youtube.com/embed/otV5Wqfy_qU?start=61&autoplay=1)]

### Adding gson to classpath

If you are setting up Gson to work with spring boot you will first want to look at `HttpMessageConvertersAutoConfiguration` as there may be configuration changes. As it exists during the write up of this tutorial the `GsonHttpMessageConverter` will be created if Gson is on your classpath, the application doesn't contain jackson's JsonGenerator class and if the Gson bean doesn't exist already.

```
@Configuration
@ConditionalOnClass(Gson.class)
@ConditionalOnMissingClass(name = "com.fasterxml.jackson.core.JsonGenerator")
@ConditionalOnBean(Gson.class)
protected static class GsonHttpMessageConverterConfiguration {

    @Bean
    @ConditionalOnMissingBean
    public GsonHttpMessageConverter gsonHttpMessageConverter(Gson gson) {
        GsonHttpMessageConverter converter = new GsonHttpMessageConverter();
        converter.setGson(gson);
        return converter;
    }

}
```

```
<dependency>
    <groupId>com.google.code.gson</groupId>
    <artifactId>gson</artifactId>
    <version>2.3.1</version>
</dependency>
```

### Excluding jackson from classpath

[[1:29](http://www.youtube.com/embed/otV5Wqfy_qU?start=89&autoplay=1)]

If you want to be sure that jackson isn't used or if you are experiencing conflicts you can add `@EnableAutoConfiguration(exclude = { JacksonAutoConfiguration.class })` to your application class and exclude it from the `spring-boot-starter-web` dependency.

```
@SpringBootApplication
@EnableAutoConfiguration(exclude = { JacksonAutoConfiguration.class })
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
        <exclusions>
        <exclusion>
            <artifactId>jackson-databind</artifactId>
            <groupId>com.fasterxml.jackson.core</groupId>
        </exclusion>
    </exclusions>
</dependency>
```

If you are trying to eliminate the dependency on jackson we did notice as we worked through this tutorial that it was needed if you are using [spring boot actuator](http://docs.spring.io/spring-boot/docs/current/reference/html/production-ready.html) specifically in `EndpointAutoConfiguration` class and it is a reported [github issue](https://github.com/spring-projects/spring-boot/issues/2247).

```
Exception in thread "main" org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'configurationPropertiesReportEndpoint' defined in class path resource [org/springframework/boot/actuate/autoconfigure/EndpointAutoConfiguration.class]: Bean instantiation via factory method failed; nested exception is org.springframework.beans.BeanInstantiationException: Failed to instantiate [org.springframework.boot.actuate.endpoint.ConfigurationPropertiesReportEndpoint]: Factory method 'configurationPropertiesReportEndpoint' threw exception; nested exception is java.lang.NoClassDefFoundError: com/fasterxml/jackson/databind/ser/BeanSerializerModifier
```

#### Validate request

Let's create a controller that returns a hashmap to validate that `GsonHttpMessageConverter` is being used. We will modify our application.properties to include `logging.level.org.springframework=DEBUG` and make a request to `/`default request mapping. Inspecting the log we can see that the `GsonHttpMessageConverter` is being used.

```
@RestController
public class MyController {

    @RequestMapping(value = "/",
            produces = { MediaType.APPLICATION_JSON_VALUE },
            method = RequestMethod.GET)
    public ResponseEntity<Map<String, String>> getContacts() {

        Map<String, String> dummyData = new HashMap<>();

        dummyData.put("java-examples",
                "http://www.leveluplunch.com/java/examples/");
        dummyData.put("groovy-examples",
                "http://www.leveluplunch.com/groovy/examples/");

        return new ResponseEntity<Map<String, String>>(dummyData, HttpStatus.OK);
    }
}
```

```
2015-01-18 07:33:51.673 DEBUG 18971 --- [nio-8080-exec-6] o.s.w.s.m.m.a.HttpEntityMethodProcessor  : Written [{java-examples=http://www.leveluplunch.com/java/examples/, groovy-examples=http://www.leveluplunch.com/groovy/examples/}] as "application/json" using [org.springframework.http.converter.json.GsonHttpMessageConverter@469047b8]
```

### Customize converters using HttpMessageConverters

[[2:14](http://www.youtube.com/embed/otV5Wqfy_qU?start=134&autoplay=1)]

If you are having troubles with configuration just discussed or interested in customizing an existing converter by overriding a bean, spring-boot provides an alternative configuration option. This method will also allow you to have both gson and jackson on your class path. We will create a `CustomConfiguration` class but this could be performed in any class that contains the `@Configuration` annotation and create a new bean of type `HttpMessageConverters`. In our configuration we will instruct springframework to use the default `HttpMessageConverter` and then append `GsonHttpMessageConverter`.

```
@Configuration
public class CustomConfiguration {

    @Bean
    public HttpMessageConverters customConverters() {

        Collection<HttpMessageConverter<?>> messageConverters = new ArrayList<>();

        GsonHttpMessageConverter gsonHttpMessageConverter = new GsonHttpMessageConverter();
        messageConverters.add(gsonHttpMessageConverter);

        return new HttpMessageConverters(true, messageConverters);
    }
}
```

If we include the gson pom dependency and specify the customConverters it will configure `GsonHttpMessageConverter` to be used. Now you might be asking, if the jackson is also included on the classpath, how does this work? The default behavior of `HttpMessageConverters` when adding a new converter is to add it to the front of the list so since jackson is configured before it works.

## Using java config

[[3:7](http://www.youtube.com/embed/otV5Wqfy_qU?start=187&autoplay=1)]

If you haven't moved to spring boot yet you still configure gson within your application using javaconfig by extending `WebMvcConfigurerAdapter` or if you need more control use `WebMvcConfigurationSupport`. You can read more on on how to [customize the provided spring mvc configuration](http://docs.spring.io/spring/docs/current/spring-framework-reference/htmlsingle/#mvc-config-customize).

```
@Configuration
@EnableWebMvc
public class Application extends WebMvcConfigurerAdapter {

    @Override
    public void configureMessageConverters(List<HttpMessageConverter < ? >> converters) {
        GsonHttpMessageConverter gsonHttpMessageConverter = new GsonHttpMessageConverter();
        converters.add(gsonHttpMessageConverter);
    }
}
```

## Configure Gson using XML configuration

[[3:26](http://www.youtube.com/embed/otV5Wqfy_qU?start=206&autoplay=1)]

I didn't add a specific test scenario for configuring gson in a xml based application but it would look something similar to the following:

```
<bean class="org.springframework.web.servlet.mvc.annotation.AnnotationMethodHandlerAdapter">
    <property name="messageConverters">
        <list>
            <bean class="org.springframework.http.converter.json.GsonHttpMessageConverter"/>
        </list>
    </property>
</bean>
```

There has been a lot of debates on which java library is the fastest JSON processor and many of them might loose out to [java 8 json api](http://www.oracle.com/technetwork/articles/java/json-1973242.html) once support broadens. So until then pick a method and library above.

Thanks for joining in today's [level up](https://www.leveluplunch.com/), have a great day!



Link:http://blog.csdn.net/do_bset_yourself/article/details/51324186

Link:https://www.leveluplunch.com/java/tutorials/023-configure-integrate-gson-spring-boot/