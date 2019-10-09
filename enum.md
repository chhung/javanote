# Enum  
宣告enum class，並增加一個方法，用來回傳字串物，用意在有些常數中，背後會另外代表某個數字，而數字並不能用來定義變數。  
```java=1
enum Level {
    ABS, BALL, CLOS, DEL, EAT, FULL;
    
    public String getVersion() {
        String ver = "";
        
        switch (this) {
        case ABS:
            ver = "304";
            break;
            
        case BALL:
            ver = "305";
            break;

        case CLOS:
            ver = "306";
            break;
            
        case DEL:
            ver = "308";
            break;
            
        case EAT:
            ver = "309";
            break;
        
        case FULL:
            ver = "310";
            break;
        default:
            break;
        }
        
        return ver;
    }
}
```
```java=1
public class App {
public void testEnum(Level v) {
    System.out.println(v.toString());
}
    
public static void main( String[] args ) {
        new App().testEnum(Level.ABS);
        System.out.println(Level.valueOf(Level.class, "CLOS"));
        for (Level v : Level.values())
            System.out.println(v.toString());
        
        for (Level v : Level.values())
            System.out.println(v.getVersion());
        
        System.out.println(Level.EAT.getVersion());
    }
}
```
Console output:  
```shell=1
ABS
CLOS
ABS
BALL
CLOS
DEL
EAT
FULL
304
305
306
308
309
310
309
```

### 另外還有EnumMap class, 可以用來建立key, value的對映, 如同使用Map<key,value>  
```java=1
enum ErrorCode {IO, NULLP, ILLEG_ARG, NET_CON_FAIL}

EnumMap<ErrorCode, String> errMsg = new EnumMap<ErrorCode, String>(ErrorCode.class);
        
errMsg.put(ErrorCode.IO, "IO Execption");
errMsg.put(ErrorCode.NULLP, "There is null pointer");
errMsg.put(ErrorCode.ILLEG_ARG, "Wrong arguments");
errMsg.put(ErrorCode.NET_CON_FAIL, "Network connection fail");
        
for (ErrorCode s : ErrorCode.values()) {
    System.out.println(errMsg.get(s));
}
```

Console output:  
```shell=1
IO Execption
There is null pointer
Wrong arguments
Network connection fail
```

### EnumSet and more Enum多種用法  
```java=1
enum MapEnum {
    MIS(10), NET(22), DOCKER("docker"), BOOK("o'reilly"), LIGHT("ikea"), DESK(87), COMPUTER(1);
    
    private int value = 0;
    private String name = "";
    
    MapEnum(int v) {
        value = v;
    }
    
    MapEnum(String s) {
        name = s;
    }
    
    public String getDescription() {
        switch (this) {
        case MIS:
        case NET:
        case DESK:
        case COMPUTER:
            return String.valueOf(value);
            
        case DOCKER:
        case BOOK:
        case LIGHT:
            return name;
        }
        
        return "";
    }
}

enum VerEnum {
    V304("304"), V305("305"), V306("306"), V307("307");
    
    private String version = "";
    
    VerEnum(String s) {
        this.version = s;
    }
    
    public String getVersion() {
        return version;
    }
}

interface Features {
    // 只要定義基本行為, 供enum使用
    public String getVersion();
}

enum VerEnumV2 implements Features {
    V304, V305, V306, V307;


    @Override
    public String getVersion() {
        switch (this) {
        case V304 : return "304";
        case V305 : return "305";
        case V306 : return "306";
        case V307 : return "307";
        default :    return "didn't define in here.";
        }
    }
}

class Gothrough implements Consumer<MapEnum> {
    @Override
    public void accept(MapEnum t) {
        System.out.println(t.toString() + ":" + t.getDescription());
    }
}

public class EnumSample {
    public static void main(String[] args) {
        MapEnum m = MapEnum.MIS;
        System.out.println("toString() = " + m.toString());
        System.out.println("*******************************");
        
        System.out.println("toString() = " + VerEnum.V305.toString());
        System.out.println("getVersion() = " + VerEnum.V305.getVersion());
        System.out.println("*******************************");
        
        VerEnum v = VerEnum.V307;
        System.out.println("toString() = " + v.toString());
        System.out.println("getVersion() = " + v.getVersion());
        System.out.println("*******************************");
        
        for (VerEnumV2 e : VerEnumV2.values())
            System.out.println("getVersion() = " + e.getVersion());
        System.out.println("*******************************");
        
        EnumSet<VerEnumV2> t = EnumSet.allOf(VerEnumV2.class);
        System.out.println("is true = " + t.contains(VerEnumV2.V304));
        System.out.println("*******************************");
        
        EnumSet<MapEnum> t2 = EnumSet.of(MapEnum.DOCKER, MapEnum.NET, MapEnum.LIGHT, MapEnum.DESK);
        System.out.println("we add the command below: [constant:description]");
        t2.forEach(new Gothrough());
        System.out.println("---------------------");
        System.out.println("Is NET include in? " + t2.contains(MapEnum.NET));
        System.out.println("Is MIS include in? " + t2.contains(MapEnum.MIS));
        System.out.println("Is DOCKER include in? " + t2.contains(MapEnum.DOCKER));
        System.out.println("Is COMPUTER include in? " + t2.contains(MapEnum.COMPUTER));
        System.out.println("*******************************");
    }
}
```

Console output:  
```shell=1
toString() = MIS
*******************************
toString() = V305
getVersion() = 305
*******************************
toString() = V307
getVersion() = 307
*******************************
getVersion() = 304
getVersion() = 305
getVersion() = 306
getVersion() = 307
*******************************
is true = true
*******************************
we add the command below: [constant:description]
NET:22
DOCKER:docker
LIGHT:ikea
DESK:87
---------------------
Is NET include in? true
Is MIS include in? false
Is DOCKER include in? true
Is COMPUTER include in? false
*******************************
```