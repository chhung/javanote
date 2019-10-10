# Annoation  
## 自訂義  
我們會build兩個專案，一個給annotation，一個是測試用，自訂的Annotation會有三個步驟要做  
1. 先定義annotation  
2. 實作annotation，繼承AbstractProcessor  
3. 建立META-INF\services\javax.annotation.processing.Processor檔案，並在裡面寫入實作的package+class name  

![](https://i.imgur.com/bTcRS30.png)
### pom.xml  
```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>cc.personal.zack</groupId>
    <artifactId>annonat</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <packaging>jar</packaging>

    <name>annonat</name>
    <url>http://maven.apache.org</url>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <maven.compiler.target>1.8</maven.compiler.target>
        <maven.compiler.source>1.8</maven.compiler.source>
    </properties>

    <dependencies>
        <!-- https://mvnrepository.com/artifact/com.google.auto.service/auto-service -->
        <dependency>
            <groupId>com.google.auto.service</groupId>
            <artifactId>auto-service</artifactId>
            <version>1.0-rc6</version>
        </dependency>

        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>3.8.1</version>
            <scope>test</scope>
        </dependency>
    </dependencies>
</project>
```

#### Output.java  
```java=1
package cc.personal.zack.annonat.tags;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Output {
    String value() default "";
}
```

#### OutputProcessor.java  
```java=1
package cc.personal.zack.annonat.processor;

import java.util.Set;
import javax.annotation.processing.AbstractProcessor;
import javax.annotation.processing.ProcessingEnvironment;
import javax.annotation.processing.Processor;
import javax.annotation.processing.RoundEnvironment;
import javax.annotation.processing.SupportedAnnotationTypes;
import javax.annotation.processing.SupportedSourceVersion;
import javax.lang.model.SourceVersion;
import javax.lang.model.element.Element;
import javax.lang.model.element.TypeElement;
import com.google.auto.service.AutoService;
import cc.personal.zack.annonat.tags.Output;

@SupportedSourceVersion(SourceVersion.RELEASE_8)
@SupportedAnnotationTypes("cc.personal.zack.annonat.tags.Output")
@AutoService(Processor.class)
public class OutputProcessor extends AbstractProcessor {
    @Override
    public synchronized void init(ProcessingEnvironment env) {
        System.out.println("init in OutputProcessor");
    }

    @Override
    public boolean process(Set<? extends TypeElement> annotations, RoundEnvironment roundEnv) {
        for (TypeElement te : annotations) {
            for (Element e : roundEnv.getElementsAnnotatedWith(te)) {
                Output output = e.getAnnotation(Output.class);
                if (output != null) {
                    System.out.println("output is:" + output.hashCode());
                    final String value = output.value();
                    System.out.println(">>>>>>>>>>>> util value: " + value);
                }
            }
        }
        return true;
    }
}
```

```bash=1
mvn clean package
```

### 測試專案  
![](https://i.imgur.com/RU9D9na.png)
#### pom.xml  
```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>cc.personal.zack</groupId>
    <artifactId>TestAnnot</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <packaging>jar</packaging>

    <name>TestAnnot</name>
    <url>http://maven.apache.org</url>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <maven.compiler.target>1.8</maven.compiler.target>
        <maven.compiler.source>1.8</maven.compiler.source>
    </properties>

    <dependencies>
        <dependency>
            <groupId>cc.personal.zack</groupId>
            <artifactId>annonat</artifactId>
            <scope>system</scope>
            <version>0.0.1-SNAPSHOT</version>
            <systemPath>D:\zack\coding\java\e2019-06r-workspace\annonat\target\annonat-0.0.1-SNAPSHOT.jar</systemPath>
        </dependency>

        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>3.8.1</version>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.5.1</version>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                </configuration>
            </plugin>

            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-dependency-plugin</artifactId>
                <executions>
                    <execution>
                        <id>copy-dependencies</id>
                        <phase>prepare-package</phase>
                        <goals>
                            <goal>copy-dependencies</goal>
                        </goals>
                        <configuration>
                            <outputDirectory>
                                ${project.build.directory}/libs
                            </outputDirectory>
                        </configuration>
                    </execution>
                </executions>
            </plugin>

            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-jar-plugin</artifactId>
                <version>3.1.2</version>
                <configuration>
                    <archive>
                        <manifest>
                            <addClasspath>true</addClasspath>
                            <classpathPrefix>libs/</classpathPrefix>
                            <mainClass>cc.personal.zack.TestAnnot.App</mainClass>
                        </manifest>
                    </archive>
                </configuration>
                <executions>
                    <execution>
                        <phase>package</phase>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
</project>
```

#### App.java  
```java=1
package cc.personal.zack.TestAnnot;

import cc.personal.zack.annonat.tags.Output;

public class App {
    @Output("ABC")
    public void Tst() {
        System.out.println("CHECK OUT!");
    }
    
    public static void main(String[] args) {
        System.out.println("Hello World!");     
        new App().Tst();
    }
}
```

```bash=1
mvn clean package
```

### 編譯時就會出現我們所設定的訊息   
![](https://i.imgur.com/X7R6hYv.png)

http://hannesdorfmann.com/annotation-processing/annotationprocessing101
