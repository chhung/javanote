# Design Pattern
#### Singleton
```java
class SingletonClass {
    private static SingletonClass singletonObject;
    
    private SingletonClass() {
        
    }
    
    public static synchronized SingletonClass getSingletonClass() {
        if (singletonObject == null)
            singletonObject = new SingletonClass();
        
        return singletonObject;
    }
    
    public Object clone() throws CloneNotSupportedException {
        throw new CloneNotSupportedException();
    }
}

public class App {
    public static void main(String[] args) {
        SingletonClass obj = SingletonClass.getSingletonClass();
        
    }
}
```