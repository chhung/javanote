# Spring  
## Spring boot Template  
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
    	<artifactId>spring-boot-starter-parent</artifactId>
    	<version>2.1.9.RELEASE</version>
    	<relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.example</groupId>
    <artifactId>demo</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>demo</name>
    <description>Demo project for Spring Boot</description>

    <properties>
        <java.version>1.8</java.version>
    </properties>

    <dependencies>
        <dependency>
    	    <groupId>org.springframework.boot</groupId>
    	    <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <scope>runtime</scope>
            <optional>true</optional>
        </dependency>

        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>

        <!-- https://mvnrepository.com/artifact/junit/junit -->
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
            <scope>test</scope>
        </dependency>

        <!-- https://mvnrepository.com/artifact/org.apache.logging.log4j/log4j-core -->
        <dependency>
            <groupId>org.apache.logging.log4j</groupId>
            <artifactId>log4j-core</artifactId>
            <version>2.12.1</version>
        </dependency>

        <!-- https://mvnrepository.com/artifact/org.apache.logging.log4j/log4j-api -->
        <dependency>
            <groupId>org.apache.logging.log4j</groupId>
            <artifactId>log4j-api</artifactId>
            <version>2.12.1</version>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
```

---
## Exception  
#### Scenario  
我們寫了一個RESTful API，再回傳給使用者前可以有一些錯誤檢查，且如果有問題則回傳錯誤訊息。(四則運算)  
request form:
```json
{"function":"add", "number1":3, "number2":5}
```
response form:
```json
{"isSuccess":true, "result":8}
```
```json
{"isSuccess":false, "errorCode":520, "errorMessage":"It can't be operation with number."}
```
### @ExceptionHandler  
![繼承RuntimeException類別](https://i.imgur.com/gXAZ9pW.png)
@ExceptionHandler所宣告處理的方法，只限定在該類別當中，意思是，若其他類別也丟出定義的exception class，那麼將不會被處理；因為找不到，例如，ArithmeticController.java裡面就拋出MathError class之後，會去找要攔截此例外的方法是否在ArithmeticController.java裡面，如果有就按方法的流程處理，如果沒有就回應http status 500 Internal Server Error。  

最上面的那張圖指的是，我們會透過自訂的錯誤類別，再去繼承RuntimeException類別，這樣在程式中拋出例外才不會繼續往下執行；因此，自訂的錯誤類別有主要兩個功用，第一個是在抛出例外前把該錯誤碼及錯誤訊息寫入到錯誤類別；第二個是錯誤分級，可以讓@ExceptionHandler攔截不同的錯誤類別。  

@ExceptionHandler預設是攔截目前所在的類別範圍抛出的任何例外，即Exception及其子類別，若要指定某個例外才要在這個方法處理的話就加上該類別，例如  
```java
@ExceptionHandler(MathError.class)
```

如果有多種不同的例外要在同一個方法裡處理  
```java
@ExceptionHandler({MathError.class, MathError2.class})
```

#### Source code  
ArithmeticController.java  
```java=1
@RestController
@RequestMapping("/operation/v1/")
public class ArithmeticController {
	
    @PostMapping(path = "/math", 
            produces = MediaType.APPLICATION_JSON_UTF8_VALUE, 
            consumes = MediaType.APPLICATION_JSON_UTF8_VALUE)
    public RespMath click(@RequestBody ReqMath body) {
        RespMath response = new RespMath();
		
        if (!body.getFunction().equalsIgnoreCase("add")) {
            MathError ret = new MathError();
            ret.setIsSuccess(false);
            ret.setErrorCode(520);
            ret.setErrorMessage("It can't be operation with number.");
            throw ret;
        }

        response.setIsSuccess(true);
        response.setResult(body.getNumber1() + body.getNumber2());

        return response;
    }

    @ExceptionHandler(MathError.class)
    private ResponseEntity<String> errorHandle(MathError ex) {
        MultiValueMap<String, String> headers = new LinkedMultiValueMap<String, String>();
        headers.add("Content-Type", "application/json;charset=UTF-8");

        ResponseEntity<String> response = 
                new ResponseEntity<String>(ex.toString(), headers, HttpStatus.BAD_REQUEST);

        return response;
    }
}
```

ReqMath.java  
```java=1
public class ReqMath {
    private String function;
    private Integer number1;
    private Integer number2;

    public String getFunction() {
        return function;
    }
    public void setFunction(String function) {
        this.function = function;
    }
    public Integer getNumber1() {
        return number1;
    }
    public void setNumber1(Integer number1) {
        this.number1 = number1;
    }
    public Integer getNumber2() {
        return number2;
    }
    public void setNumber2(Integer number2) {
        this.number2 = number2;
    }
}
```

RespMath.java  
```java=1
public class RespMath {
    private Boolean isSuccess;
    private Integer result;

    public Boolean getIsSuccess() {
        return isSuccess;
    }
    public void setIsSuccess(Boolean isSuccess) {
        this.isSuccess = isSuccess;
    }
    public Integer getResult() {
        return result;
    }
    public void setResult(Integer result) {
        this.result = result;
    }
}
```

MathError.java  
```java=1
public class MathError extends RuntimeException {
    private static final long serialVersionUID = 3769051713058814440L;

    private Boolean isSuccess;
    private Integer errorCode;
    private String errorMessage;

    @Override
    public String toString() {
        StringBuilder str = new StringBuilder();
        str.append("{")
            .append("\"isSuccess\":").append(isSuccess).append(",")
            .append("\"errorCode\":").append(errorCode).append(",")
            .append("\"errorMessage\":").append("\"").append(errorMessage).append("\"")
            .append("}");

        return str.toString();
    }
    public Boolean getIsSuccess() {
        return isSuccess;
    }
    public void setIsSuccess(Boolean isSuccess) {
        this.isSuccess = isSuccess;
    }
    public Integer getErrorCode() {
        return errorCode;
    }
    public void setErrorCode(Integer errorCode) {
        this.errorCode = errorCode;
    }
    public String getErrorMessage() {
        return errorMessage;
    }
    public void setErrorMessage(String errorMessage) {
        this.errorMessage = errorMessage;
    }
}
```

### HandlerExceptionResolver  
這個適合用在Web MVC，RESTful API就不適合，待續...  

### @ControllerAdvice   
還記得@ExceptionHandler單獨使用只能用在該Controller類別範圍，如果其他Controller類別也會抛出例外時，就得再寫一次相同的程式碼，這樣子程式碼就有太多地方重覆了，很難維護，也不直觀，所以我們加上@ControllerAdvice。  

新增一個會處理所有例外的例外，然後把原本@ExceptionHandler寫的程式碼換到有@ControllerAdvice的類別，原本會抛出例外的代碼片段不用更改，而且其他Controller類別抛出例外也照樣能被攔截到。  

GlobalExceptionHandler.java  
```java=1
@ControllerAdvice
public class GlobalExceptionHandler {
    @ExceptionHandler(MathError.class)
    private ResponseEntity<String> errorHandle(MathError ex) {
        MultiValueMap<String, String> headers = new LinkedMultiValueMap<String, String>();
        headers.add("Content-Type", "application/json;charset=UTF-8");

        ResponseEntity<String> response = 
                new ResponseEntity<String>(ex.toString(), headers, HttpStatus.BAD_REQUEST);

        return response;
    }
}
```

### @Validated所抛出的例外  
這個地方，目前我只能做到攔截所有@Validated所抛出的例外，還無法指定自定義的類別。  
```java=1
@ControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(MathError.class)
    private ResponseEntity<String> errorHandle(MathError ex) {
        MultiValueMap<String, String> headers = new LinkedMultiValueMap<String, String>();
        headers.add("Content-Type", "application/json;charset=UTF-8");

        ResponseEntity<String> response = 
                new ResponseEntity<String>(ex.toString(), headers, HttpStatus.BAD_REQUEST);

        return response;
    }

    @ExceptionHandler(ValidationException.class)
    private ResponseEntity<String> handleValidationException(ValidationException e) {
        MultiValueMap<String, String> headers = new LinkedMultiValueMap<String, String>();
        headers.add("Content-Type", "application/json;charset=UTF-8");

        ResponseEntity<String> response = 
                new ResponseEntity<String>("{\"messag\":\"aaaaaaaaaaa\"}", headers, HttpStatus.BAD_REQUEST);

        return response;
    }

    @ExceptionHandler(MethodArgumentNotValidException.class)
    //@ResponseBody
    private ResponseEntity<String> handleMethodArgumentNotValidException(MethodArgumentNotValidException e) {
        MultiValueMap<String, String> headers = new LinkedMultiValueMap<String, String>();
        headers.add("Content-Type", "application/json;charset=UTF-8");

        ResponseEntity<String> response = 
                new ResponseEntity<String>("{\"messag\":\"xxxxxxxxxx\"}", headers, HttpStatus.BAD_REQUEST);

        return response;
    }
}
```

---
## 