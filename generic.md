# Generic  
泛型主要是用來把Collection型別做參數化。  
泛型無法用static來修飾class，因為，泛型是個別實體化的，所以不能以static來實體化class。  
承上述，但是可以用來修飾靜態方法。  
泛型的宣告中，還有一種可以直接用萬用符號 "?"，用了此符號是任何資料型態都可以代入，但是，因為在執行時期並無法檢驗出是哪一種資料型態，所以該變數是無法被修改的(例如，List型別的add()/addAll()/set()方法)，在編譯時期就會禁止。  

## 類別的宣告  
```java=1
class XML<E, T> {
	
}
```

## 方法的宣告  
```java=1
public <T, V, K> void doSomeThing(List<T> e, Map<K, V> a) {
    int index = 0;
    for (T s : e) {
        e.set(index++, (T)"33");
    }
    e.forEach(x -> System.out.println(x.toString()));
}
```

```java=1
public static <T, V, K> void doSomeThing(List<T> e, Map<K, V> a) {
    int index = 0;
    for (T s : e) {
        e.set(index++, (T)"33");
    }
    e.forEach(x -> System.out.println(x.toString()));
}
```

## 限定某個型別  
例如，現在要寫一個加總的方法，只限定數值型態，整數，浮點數都可以使用。  
```java=1
public <T extends Number> double sum(List<T> x) {
    double s = 0.0;
    for (Iterator<T> e = x.listIterator(); e.hasNext(); ) {
        s += e.next().doubleValue();
    }
		
    return s;
}

List<Float> g = new ArrayList<Float>();
for (int y = 1; y <= 100; y++) {
    g.add((float)y);
}
System.out.println(sum(g));
```
也可以替換成萬用字元  
```java=1
public double sum(List<? extends Number> x) {
    double s = 0.0;
    for (Iterator<? extends Number> e = x.listIterator(); e.hasNext(); ) {
        s += e.next().doubleValue();
    }

    return s;
}
```
基本上泛型要再更深入理解，以上只是一些皮毛！  
下面這個link，有說明Class\<T\> and Class\<\?\>的區別  
[https://www.jianshu.com/p/95f349258afb](https://www.jianshu.com/p/95f349258afb)
