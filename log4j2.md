# Log4j2  
### Maven pom.xml  
```xml=1
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
```

### 配置檔設定，放在src/main/resources目錄下，log4j自動會去找log4j2.xml  
```xml=1
<?xml version="1.0" encoding="UTF-8"?>

<!-- 優先等級: OFF > FATAL > ERROR > WARN > INFO > DEBUG > TRACE > ALL -->
<!--Configuration後面的status(status="Debug"),這個用於設置log4j2自身內部的信息輸出,可以不設置,當設置成trace時,你會看到log4j2內部各種詳細輸出 -->
<!--monitorInterval：Log4j能夠自動檢測修改配置 文件和重新配置本身,設置間隔秒數 -->
<Configuration status="OFF" monitorInterval="30">
    <!-- 變數定義 -->
    <properties>
        <property name="LOG_HOME">/log/mis</property>
        <property name="FILE_NAME">mis</property>
    </properties>

    <!--定義appender -->
    <Appenders>
    <!-- 用來定義輸出到控制檯的配置 -->
        <Console name="STDOUT" target="SYSTEM_OUT">
            <PatternLayout
                pattern="%d %-5p [%t] %C{2} (%F:%L) - %m%n" />
        </Console>
        <File name="FILE" fileName="${LOG_HOME}/logfile_fileMode.log"
			append="true">
            <PatternLayout
                pattern="%d %-5p [%t] %C{2} (%F:%L) - %m%n" />
        </File>

        <!-- 打印root中指定的level級別以上的日誌到文件 -->
        <RollingRandomAccessFile name="MIS_PROJECT"
            fileName="${LOG_HOME}/${FILE_NAME}.log"
            filePattern="${LOG_HOME}/$${date:yyyy-MM}/${FILE_NAME}-%d{yyyy-MM-dd}-%i.log.gz">

            <PatternLayout
                pattern="%d %-5p [%t] %C{2} (%F:%L) - %m%n" />
            <Policies>
                <TimeBasedTriggeringPolicy />
                <!-- 單一檔最大為10MB -->
                <SizeBasedTriggeringPolicy size="10 MB" />
            </Policies>
            <!-- 壓縮包最多保留20個 -->
            <DefaultRolloverStrategy max="20" />
        </RollingRandomAccessFile>

    </Appenders>

    <Loggers>
        <!--过滤掉spring和mybatis的一些无用的DEBUG信息 -->
        <logger name="org.springframework" level="INFO"></logger>
        <logger name="org.mybatis" level="INFO"></logger>

        <Root level="debug">
            <AppenderRef ref="STDOUT" />
        </Root>
        <!-- <Root level="WARN"> <AppenderRef ref="MIS_PROJECT" /> </Root> -->
    </Loggers>
</Configuration>
```

### Use it  
```java=1
import org.apache.logging.log4j.LogManager;
import org.apache.logging.log4j.Logger;

public class App {
    // 宣告log4j2
    private static final Logger log = LogManager.getLogger(App.class);
	
    public static void main(String[] args) {
        log.info("Hello Log");
    }
}
```


---
## 一支穿雲箭，千萬馬來相見  
![](https://i.imgur.com/6SB90tM.png)
## Configure file  
### 最簡單的log4j2.xml  
#### logging into Console  
```xml=1
<?xml version="1.0" encoding="UTF-8"?>
<Configuration status="WARN" monitorInterval="30">
    <Appenders>
        <Console name="console" target="SYSTEM_OUT">
        <PatternLayout
            pattern="[%-5level] %d{yyyy-MM-dd HH:mm:ss.SSS} [%t] %c{1} - %msg%n" />
        </Console>
    </Appenders>
    <Loggers>
        <Root level="trace" additivity="false">
            <AppenderRef ref="console" />
        </Root>
    </Loggers>
</Configuration>
```

Configuration是最上層結點，底下有兩種結點，第一個是Appenders和Loggers  
Configuration有兩個屬性，status代表log4j本身會輸出的訊息級別；monitorInterval表示log4j每隔多少時間重新讀取log4j2.xml設定檔，重新設定，單位時間為秒。  

#### 小插曲-探討Logger繼承  
在Loggers tag中會有兩種，第一就是常用的Root tag，第二種是自訂義的Logger tag，這兩種的區別是Logger有name屬性，這個屬性可以代表 java package，所以我們可以把特定的類別個別輸出到指定的地方(Appenders)。  
其中還有一個屬性，additivity="true/false"，因為Root是根節點也就是最後收到所有event的地方。而additivity決定了收到event的Logger要不要再繼續往上送(最後送到Root)，true就表示自己處理完後要再往上送。  

### logging into rolling files  
以下會有一個Logger處理name="idv.zack.maven.log4jj.myhome"的classpath送出的level="debug"等級以上的message交給\<appender-ref ref="fileLogger" level="debug" /\>也就是RollingFile name="fileLogger"處理存放到檔案；之後看到additivity="true"，就再把envnt往上送。如果是false就不再往上送了。  

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Configuration status="WARN" monitorInterval="30">
    <Properties>
        <Property name="basePath">logs</Property>
    </Properties>

    <Appenders>
        <RollingFile name="fileLogger"
            filePattern="${basePath}/app-info-%d{yyyy-MM-dd}.log">
            <PatternLayout>
                <pattern>[%-5level] %d{yyyy-MM-dd HH:mm:ss.SSS} [%t] %c{1} - %msg%n
                </pattern>
            </PatternLayout>
            <Policies>
                <!-- TimeBasedTriggeringPolicy interval="1" modulate="true" /-->
                <SizeBasedTriggeringPolicy size="20MB" />
            </Policies>
            <!--<DefaultRolloverStrategy min="1" max="4"/>-->
            <DirectWriteRolloverStrategy maxFiles="4"/>
        </RollingFile>

        <Console name="console" target="SYSTEM_OUT">
            <PatternLayout
                pattern="[%-5level] %d{MM-dd HH:mm:ss.SSS} [%t] %c{1} - %msg%n" />
        </Console>
    </Appenders>
    <Loggers>
        <Logger name="idv.zack.maven.log4jj.myhome" level="debug"
			additivity="true">
            <appender-ref ref="fileLogger" level="debug" />
        </Logger>

        <Root level="trace" additivity="false">
            <AppenderRef ref="console" />
        </Root>
    </Loggers>
</Configuration>
```

如果要把console輸出的直接也都送一份到file時，只要把上述的xml file中的Logger裡面的<appender-ref ref="fileLogger" level="debug" />放到Root tag裡面，而原本的Logger拿掉即可，注意寫到file的event level可以分開設定。範例如下：  
```xml
<Root level="trace" additivity="false">
    <AppenderRef ref="console" />
    <AppenderRef ref="fileLogger" level="trace"/>
</Root>
```
