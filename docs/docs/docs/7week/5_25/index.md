# 第7周-5.25

## 1 StringSubstitutor

[String 工具之StrSubstitutor, 字符替换](https://blog.csdn.net/zhaoxj_2017/article/details/92839873)

字符替换，可以使用正则表达式对匹配的字符进行替换，这种方式比较灵活。

### 1.1 简单使用

```
Map valuesMap = new HashMap();
valuesMap.put("animal", "quick brown fox");
valuesMap.put("target", "lazy dog");
String templateString = "The ${animal} jumped over the ${target}.";
StringSubstitutor sub = new StringSubstitutor(valuesMap);
String resolvedString = sub.replace(templateString);
System.out.println(resolvedString);

//output
The quick brown fox jumped over the lazy dog.
```

### 1.2 默认值

如果在Map实体中没找到对应的Key值，变量就不会被替换，而是直接以字符串展示，

但是StringSubstitutor提供取值分隔符 `:-`，添加到变量的后面，可以为该值提供默认值。

>StrSubstitutor提供了setValueDelimiterMatcher(StrMatcher), setValueDelimiter(char) or setValueDelimiter(String)三种方法，可以自定义默认取值分隔符。

```
templateString = "The ${animal} jumped over the ${target}. ${undefined.number:-1234567890}.";
String resolvedString2 = sub.replace(templateString);
System.out.println(resolvedString);

//output
The quick brown fox jumped over the lazy dog. 1234567890.
```

### 1.3 自定义匹配符

如果字符串中的变量形式不是 `${}`，而是`&()`，StringSubstitutor提供了不同的构造器以及`setVariablePrefix(char)`，`setVariableSuffix(char)`等方法自定义匹配符。

```
String templateString = "The &(animal) jumped over the &(target). &(undefined.number:-1234567890).";
StrSubstitutor sub = new StrSubstitutor(valuesMap, "&(", ")", '&');
String resolvedString = sub.replace(templateString);
System.out.println(resolvedString);

//output
The quick brown fox jumped over the lazy dog. 1234567890.
```

### 1.4 自定义实体类

在系统中装载数据的实体有可能不是`Map`,而是其它的数据实体，例如`com.fasterxml.jackson.databind.node.JsonNode`。

```
public class StrSubstitutorTest {
    @Test
    public void test() {
        StrSubstitutor substitutor = new StrSubstitutor();
        ObjectNode context = JsonNodeFactory.instance.objectNode();
        context.put("animal", "quick brown fox");
        context.put("target", "lazy dog");
        
        String templateString = "The ${animal} jumped over the ${target}. ${undefined.number:-1234567890}.";
 
        substitutor.setVariableResolver(new JsonPathContextLookup(context));
        String resolvedString = substitutor.replace(templateString);
        
        System.out.println(resolvedString);
        //output: The quick brown fox jumped over the lazy dog. 1234567890.
    }
 
    class JsonPathContextLookup extends StrLookup<String> {
        private final JsonNode context;
 
        JsonPathContextLookup(JsonNode context) {
            super();
            this.context = context;
        }
 
        @Override
        public String lookup(String key) {
            return Optional.ofNullable(context.get(key)).map(JsonNode::asText).orElse(null);
        }
    }
}
```