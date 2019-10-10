# Misc
## MessageFormat  
```java=1
String msg = MessageFormat.format("==={0, date, short}===", new Date());
System.out.println(msg);
```
---
## Monitor memory  
```java=1
private void monitorMemory(int loop, long millis) {
    Runtime runtime = Runtime.getRuntime();
    //runtime.gc();
    long memory = 0;
    memory = runtime.totalMemory() - runtime.freeMemory();
    System.out.println("Used memory is bytes: " + memory);
    System.out.println("Used memory is megabytes: "
            + bytesToMegabytes(memory));
}
```

```java=1
private static long bytesToMegabytes(long bytes) {
    return bytes / (1024L * 1024L);
}
```

---
## Jackson JSON  
#### Maven pom.xml  
```xml
<properties>
  ...

  <!-- Use the latest version whenever possible. -->
  <jackson.version>2.9.8</jackson.version>
  ...
</properties>

<dependencies>
  ...
  <dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>${jackson.version}</version>
  </dependency>
  ...
</dependencies>
```
Jackson有三個JAR檔(core/annotations/databind)，其中只要引入databind，其他兩個就會一併被引入，因為databind相依其他兩個library。  

### Jackson使用ObjectMapper來處理json格式的Serialize序列化與Deserialize反序列  
```java=1
ObjectMapper mapper = new ObjectMapper(); // create once, reuse
```

#### Deserialize反序列化 (json to POJOs)  
```java=1
public class MyValue {
    public String name;
    public Integer age;
}
```

```java==1
// 從檔案讀取
MyValue value = mapper.readValue(new File("data.json"), MyValue.class);
// 從網路讀取
MyValue value = mapper.readValue(new URL("http://some.com/api/entry.json"), MyValue.class);
// 從字串讀取
MyValue value = mapper.readValue("{\"name\":\"Bob\", \"age\":13}", MyValue.class);
```

#### Serialize序列化 (POJOs to json)  
```java=1
MyValue myResultObject = new MyValue();
myResultObject.name = "Batty";
myResultObject.age = 25;

// 寫到檔案
mapper.writeValue(new File("result.json"), myResultObject);
// 變成 byte array
byte[] jsonBytes = mapper.writeValueAsBytes(myResultObject);
// 變成字串
String jsonString = mapper.writeValueAsString(myResultObject);
```

#### Generic collections   
```java=1
String jsonSource = "{\"Andy\":89, \"Batty\":99}";

// 直接傳回Map object
Map<String, Integer> scoreByName = mapper.readValue(jsonSource, Map.class);
scoreByName.forEach((k, v) -> log.info("{}:{}", k, v));

// 直接傳回List object
jsonSource = "[\"Andy\", \"Batty\"]";
List<String> names = mapper.readValue(jsonSource, List.class);
names.forEach(x -> log.info("{}", x.toString()));
		
// 原本接到什麼就寫入什麼
mapper.writeValue(new File("names.json"), names);
```

names.json
```
[ "Andy","Batty" ]
```

### @JsonGetter   
@JsonGetter annonation，用來表示一個json pojo model的某個屬性getter method是哪一個mothod, 亦即，jackson解析時，預設會先去找setter/getter method，即便我們不需要給任何的annonation
意思是，只要是get/set開頭的method，jackson都會掃描，而預設屬性名稱為去掉get/set後，第一個字母變小寫  
EX.  
```java
private String Name;
public String getName() { return Name; }  –>屬性名稱為name
public String getAbcd() { return Name; }  –>屬性名稱為abcd
```
而且這兩個屬性都會被輸出  

回到JsonGetter上面來，jackson都可以自己解析了，何需用此註解？  
有一種情況會需要，就是取得屬性的方法名稱不會命名為getter方式，則用此註解來告訴Jackson，這個方法需要輸出成json的一部份  
EX.
```java
public String printName() { return Name; } –>Jackson parser時不會理會這個方法
```

而加上註解後  
```java
@JsonGetter(“name")
public String printName() { return Name; } –>json syntax is { "name" : xxxx }
```
```java
@JsonGetter()
public String printName() { return Name; } –>json syntax is { "printName" : xxxx }
```

### @JsonAnyGetter  
這個註解是把Map資料型能屬性當成一般的資料型態屬性 舉例來說，json語法為name:value的組合，而Map也是key:value的組合兩者很像，但是，Map是一種集合，所以不加任何註解在Jackson解析下，會保留屬性名稱，而內容值則為Map的所有集合  
```java=1
public class JsonPojo { 
    public String name; private Map properties;
    @JsonAnyGetter(enabled = true)
    public Map getProperties() {
        return properties;
    }

    public void add(String k, String v) {
        if (properties == null)
            properties = new TreeMap();

        properties.put(k, v);
    }
}
```

Test  
```java=1
public class JsonPojoTest { 
    @Test 
    public void test() { 
        JsonPojo jp = new JsonPojo();
        jp.name = "Betty";
        jp.add("attr1", "val1");
        jp.add("attr2", "val2");

        String result = null;
        try {
            result = new ObjectMapper().writeValueAsString(jp);
        } catch (JsonProcessingException e) {
            e.printStackTrace();
        }

        assertThat(result, containsString("attr1"));
        assertThat(result, containsString("val1"));
        System.out.println(result);
        jp.getProperties().forEach((k, v) -> {System.out.println("key:" + k + "\tvalue:" + v);});
    }
}
```

有加JsonAnyGetter的輸出  
```
{"name":"Betty","attr1":"val1","attr2":"val2"} 
key:attr1 value:val1 
key:attr2 value:val2
```

沒有加 JsonAnyGetter的輸出  
```
{"name":"Betty","properties":{"attr1":"val1","attr2":"val2"}} 
key:attr1 value:val1 
key:attr2 value:val2
```

### json名稱開頭大小寫通吃  
json syntax:
```json
{ "name":"Andy" }
```

java class:
```java=1
public class JsonModel {
    private String name; 
    public void setName(String s) { 
        this.name = s;
    }
    public String getName() {
        return this.name;
    }
}
```

在java裡創建一個class，用來接上面的json，使之成為一個object供程式直接來使用，這個過程稱反序列化如果現在把上面那段json轉成java object時，可用下面方式，不用加任何Jackson annotation  
```java=1
ObjectMapper jsonOmp = new ObjectMapper();
jsonOmp.readValue([json_string_source], JsonModel.class);
```

重點來了，json syntax中的name，對於有些人來說，name與Name的意思是相同的，但是這在做反序列化可行不通，若要兩者都可以吃下來，則在class要多加annonation和method  

例如  
```java=1
public class JsonModel { 
    private String name;
    public void setName(String s) { 
        this.name = s; 
    }
    public String getName() { 
        return this.name; 
    }
}
```

如果原本就是接受json name屬性是小寫，突然有一天，全部都要改大寫的話，就直接在java class 屬性加上@JsonProperties("Name") 如此一來，就只能接受開頭大寫  

---
## SimpleDate  
給字串轉成date物件(要先設定好接收的時間字串的格式)  
```java=1
// format likes this 8/12/2010 9:32:33 PM
String pattern = "M/d/yyyy h:m:s a"; 

// Locale.{} will change the display am/pm by localization
SimpleDateFormat strDateTime = new SimpleDateFormat(pattern, Locale.ENGLISH); 
System.out.println(strDateTime.format(new Date()));

try {
    Date dt = strDateTime.parse("8/12/2010 9:32:33 PM");
    System.out.println(strDateTime.format(dt));
} catch (ParseException e) {
    e.printStackTrace();
}
```

---
## Collection  
list \<--\> array, array \<--\> string

java array to list:
Arrays.asList(List Object)

java list to array:
array[] array_obj = new array[List_Object.size()]
array_obj = List_Object.toArray(array_obj);

array to stream:
String[] myArray = {"dd", "ss", "ee"}
Stream\<String\> myStream = Arrays.stream(myArray)  


---
## Math  
BigDecimal
```java=1
public class Maths {
    private static final int DIV_SCAL = 10;
	
    private Maths() {}
	
    public static double add(double a, double b) {
        BigDecimal x = BigDecimal.valueOf(a);
        BigDecimal y = BigDecimal.valueOf(b);

        return x.add(y).doubleValue();
    }
	
    public static double sub(double a, double b) {
        BigDecimal x = BigDecimal.valueOf(a);
        BigDecimal y = BigDecimal.valueOf(b);

        return x.subtract(y).doubleValue();
    }
	
    public static double mul(double a, double b) {
        BigDecimal x = BigDecimal.valueOf(a);
        BigDecimal y = BigDecimal.valueOf(b);

        return x.multiply(y).doubleValue();
    }

    public static double div(double a, double b) {
        BigDecimal x = BigDecimal.valueOf(a);
        BigDecimal y = BigDecimal.valueOf(b);

        return x.divide(y, DIV_SCAL, BigDecimal.ROUND_HALF_UP).doubleValue();
    }
}
```

---
## POM.xml info  
```java=1
public String getMvnVersion() {
    String ver = "";
    ver = getClass().getPackage().getImplementationVersion();

    return ver;
}
```

---
## 

