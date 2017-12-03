# Google Gson

## Class JsonParser

A parser to parse Json into a parse tree of [`JsonElement`](http://static.javadoc.io/com.google.code.gson/gson/2.8.0/com/google/gson/JsonElement.html)s

### Method Summary

| `JsonElement` | `parse(JsonReader json)`Returns the next value from the JSON stream as a parse tree |
| ------------- | ---------------------------------------- |
| `JsonElement` | `parse(Reader json)`Parses the specified JSON string into a parse tree |
| `JsonElement` | `parse(String json)`Parses the specified JSON string into a parse tree |

典型用法：

将一个Json数组类型的字符串转换成jsonArray对象：

先首先转换成JsonElement对象，再转换成JsonArray对象；

```java
jsonArray = new JsonParser().parse(response.getBody().toString()).getAsJsonArray();
```

