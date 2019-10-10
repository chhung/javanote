# Stream  
stream method有兩種類型，***Intermediate*** 和 ***Terminal***。  
一個stream source後面可以接著零個或多個Intermediate的方法。  
一個stream source後面最多只能接著一個Terminal方法。  

*很重要(容易疏忽)，一個 stream source 只要執行過 terminal 方法，則該 stream source 就不能再繼續使用，否則會報*
<font color=red><strong>java.lang.IllegalStateException: stream has already been operated upon or closed</strong></font>

通常在 Java API Document中會有以下提示該操作是哪一類的方法。  
<font color=red>**This is a terminal operation**</font>

取得或建立stream source的常見幾種方式  
```java=1
// 1. Individual values
Stream stream = Stream.of("xa", "ac", "b", "a");

// 2. Arrays
String [] strArray = new String[] {"a", "b", "c"};
stream = Stream.of(strArray);
stream = Arrays.stream(strArray);

// 3. Collections
List<String> list = Arrays.asList(strArray);
stream = list.stream();
```

目前stream已經準備好三種類型的stream，IntStream、LongStream、DoubleStream。  
雖然可以用Stream\<Integer\>、Stream\<Long\>、Stream\<Double\>，但是 boxing 和 unboxing 會很耗時。  
```java=1
IntStream.of(new int[]{1, 2, 3}).forEach(System.out::println);
IntStream.range(1, 3).forEach(System.out::println);
IntStream.rangeClosed(1, 3).forEach(System.out::println);
```

keep study  
[https://www.ibm.com/developerworks/cn/java/j-lo-java8streamapi/index.html](https://www.ibm.com/developerworks/cn/java/j-lo-java8streamapi/index.html)
