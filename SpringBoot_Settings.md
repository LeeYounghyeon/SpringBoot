# 2. 스프링 부트 환경 설정

- JDK설치
- Intellij 사용

- 예제 -> HelloWorld 띄우기


```java
@ResController --1
@SpringBootApplication
public class CommunityApplication{
  public static void main(String[] args){
    SpringApplication.run(Application.class,argd);
  }

  @GetMapping --2
  public String HelloWorld(){
    return "Hello World";
  }
}
```

1. `@ResController` : @Controller 와  @ResponseBody를 합쳐놓은 역할을 수행. 어노테이션을 사용하면 스프링은 반환값이 ResponseBody 부분에 자동으로 바인딩된다. 스프링에서 RESTful 웹 서비스를 만들 때 주로 사용한다.
2. `@GetMapping` : Get 방식으로 경로를 받는 매핑 어노테이션이다. value 값을 별도로 지정하지 않으면 기본값인 빈 갑이 들어간다.


### 2.2.3 얼티미트 버전에서 스프링 부트 사용하기
- 프로젝트 생성


1. Create New Project 선택한다.
2. Spring Initializr 을 선택하고 Next누른다.
3. 프로젝트의 그룹,빌드 타입, 자바 버전, 기타 설정을 선택하고 Next
4. Web 의존성을 선택한다.
5. 프로젝트 명 지정 후 Finish를 누르면 완료


- 프로젝트 생성 및 실행



1. 인텔리제이 메인 창에서 오른쪽 상단에 있는 'Edit Configurations'를 선택한다.
2. 왼쪽 상단의 + 버튼을 선택하고 Spring Boot를 선택해 스프링 부트 플러그인을 추가한다.
3. 추가된 스프링 부트 플러그인을 선택하고 항목을 채워 넣는다.

        Main Class : Community Application
        Use classpath of modules : Community_main
4. Run을 누르면 실행된다.

## 2.3 그레이들 설치 및 빌드하기
- 그레이들은 앤트로부터 기본적인 빌드 도구의 기능을, 메이븐으로부터 의존 라이브러리 관리 기능을 차용했다.
- 설정 주입 방식을 사용해 훨씬 유연하게 빌드 환경을 구성할 수 있다.

### 2.3.1 그레이들 래퍼
- 그레이들을 설치한다.
      - gradle : 리눅스 및 맥 OS용 셸 스크립트
      - gradlew.bat : 윈도우용 배치 스크립트
      - gradle/wrapper/gradle-wrapper.jar : Wrapper JAR
      - gradle/wrapper/gradle-wrapper.properties: 그레이들 설정 정보 프로퍼티 파일

      *실행 권한 추가 -> $chmod 755 gradlew
      *버전 올리기 -> $./gradlew wrapper --gradle-version 4.8.1
      *버전 확인 -> $./gradlew -v

### 2.3.2 그레이들 멀티 프로젝트 구성하기
- 공통 코드를 하나의 프로젝트로 분리하고 이를 재사용할 때 유용하다.


1. `settings.gradle`: 그레이들 설정파일
  - 루트 프로젝트 추가
          rootProject.name = 'demo'
2. 루트 경로에서 New -> Module선택 -> Gradle 선택 -> Java 선택 -> Next
3. Add as modile to 에서 Community 프로젝트를 선택 -> ArtifactId에 demo-web을 입력 후 Next로 모듈 생성
4. 기본 패키지 경로를 수동으로 생성
        - src/main/java : 자바 소스 경로
        - src/test/java : 스프링 부트의 테스트 코드 경로
        - src/main/resources/static : static한 파일의 디폴트 경로
        - src/main/resources/templates : thymeleaf,freemarker 및 기타 서버 사이드 템플릿 파일의 경로


- 인텔리제이에서 모듈 생성 기능을 사용하면 `settings.gradle`에 자동으로 생성된 모듈명이 인클루드된다.
      rootProject.name = 'demo'
      include 'demo-web'



## 2.4 환경 프로퍼티 파일 설정하기
- 정적인 값을 키값 형식으로 관리한다.
- application.properties에 `server.port: 80`을 입력하면 80번으로 서버 포트 설정을 변경한다.
- 최근엔 표현의 한계로 YAML파일을 더 많이 사용한다. -> application.yml을 생성
       server:
            port: 80


### 2.4.1 프로파일에 따른 환경 구성 분리
- 프로퍼티 설정을 ---을 기준으로 설정값을 나눈다.
      server:
          port: 80
      ---
      spring:
          profiles: local
      server:
          port: 8080
      ---
      spring:
          profiles: dev
      server:
          port: 8081
      ---
      spring:
          profiles: real
      server:
          port: 8082

- 다른 방법은 application-{profile}.yml을 이용하는 것이다.
- 예를들어, local,dev,real과 같이 각각의 프로파일값을 따로 지정하여 애플리케이션을 실행한다고 가정한다.

      $java -jar ... -D spring.profiles.active=dev

### 2.4.2 YAML 파일 매핑하기
- YAML 파일을 사용하면 깊이에 따라 관계를 구분짓는다.
- 각 기능
  - 유연한 바인딩 : 프로퍼티값을 객체에 바인딩할 경우 필드를 낙타 표기법으로 선언하고 프로퍼티의 키는 다양한 형식, 언더바 표기법으로 선언하여 바인딩할 수 있다.
  - 메타데이터 지원 : 프로퍼티의 키에 대한 정보를 메타데이터 파일로 제공합니다. 키의 이름, 타입, 설명, 디폴트값 등 키 사용에 앞서 힌트가 되는 정보를 얻을 수 있다.
  - SpEL 평가 : SpEL은 런타임 객체 참조에 대해 질의하고 조작하는 기능을 지원하는 언어이다. @Value만 사용가능하다.

#### `@Value` 살펴보기
  - 프로퍼티의 키를 사용하여 특정한 값을 호출할 수 있다.
  - 키를 정확히 입력해야 하며 값이 없을 경우에 대해 예외 처리를 해주어야 합니다.
  - application.yml 파일에 테스트 프로퍼티값을 추가한다. -> {키:값}
        property:
            test:
                name: property depth test
        propertyTest: test
        propertyTestList: a,b,c

- `@Value`을 다양한 방식으로 매핑

- /test/java/com/demo/AutoconfigurationApplicationTests.Java




  ```java
package com.example.demo2;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

import java.util.List;

import static org.hamcrest.CoreMatchers.is;
import static org.junit.Assert.assertThat;

@RunWith(SpringRunner.class)
@SpringBootTest

public class Demo2ApplicationTests {
    @Value("${property.test.name}")
    private String propertyTestName;

    @Value("${propertyTest}")
    private String propertyTest;

    @Value("${noKey:default value}")
    private String defaultValue;

    @Value("${propertyTestList}")
    private String[] propertyTestArray;

    @Value("#{'${propertyTestList}'.split(',')}")
    private List<String> propertyTestList;

    @Test
    public void testValue(){
        assertThat(propertyTestName, is("property depth test"));
        assertThat(propertyTest, is("test"));
        assertThat(defaultValue,is("default value"));

        assertThat(propertyTestArray[0], is("a"));
        assertThat(propertyTestArray[1], is("b"));
        assertThat(propertyTestArray[2], is("c"));

        assertThat(propertyTestList.get(0), is("a"));
        assertThat(propertyTestList.get(1), is("b"));
        assertThat(propertyTestList.get(2), is("c"));
    }
}
```

- assertThat(): 첫 번째 파라미터와 두 번째 파라미터가 일치해야 테스트가 성공한다.

- `@Value` 매핑 방식
      - @Value("${property.test.name}")
        -> 깊이가 존재하는 키값에 대해 '.'로 구분하여 해당 값을 매핑한다.
      - @Value("${propertyTest}")
        -> 단일 키값을 매핑한다.
      - @Value("${noKey:default value}")
        -> YAML 파일에 키값이 존재하지 않으면 디폴트 값이 매핑되도록 설정한다.
      - @Value("${propertyTestList}")
        -> 여러 값을 나열할 때는 배열형으로 매핑합니다.
      - @Value("#{'${propertyTestList}'.split(',')}")
        -> SpEL을 사용하여 ','을 기준으로 List에 매핑합니다.


#### `@ConfigurationProperties`살펴보기
  -  프로퍼티를 사용하여 다양한 형의 프로퍼티값을 매핑할 수 있다.
  - 접두사를 사용하여 값을 바인딩한다.
        fruit:
            list:
              - name: banana
                color: yellow
              - name: apple
                color: red
              - name: water melon
                color: green

- com/demo/pojo/FruitProperty.java


```java
package com.example.demo2.pojo;

import lombok.Data;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.stereotype.Component;

import java.util.List;
import java.util.Map;

@Data
@Component
@ConfigurationProperties("fruit")
public class FruitProperty {
    private List<Fruit> list;
}
```
- application.yml의 프로퍼티 값들은 @ConfigurationProperties를 활용한 FruitProperty 클래스의 list 필드로 바인딩된다.
- @Data : 롬복 설정으로, 컴파일 시점에 getter와 setter을 클래스 내부 필드에 자동 생성한다.
- @ConfigurationProperties을 사용하려면 해당 클래스를 @Componenet로 선언해야 한다.

- test/java/com/demo/pojo/PropertyTest.java

```java
package com.example.demo2.pojo;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

import java.util.List;
import java.util.Map;

import static org.hamcrest.Matchers.is;
import static org.junit.Assert.assertThat;

@RunWith(SpringRunner.class)
@SpringBootTest
public class PropertyTest {

    @Autowired
    FruitProperty fruitProperty;

    @Test
    public void test(){
        List<Map> fruitData = fruitProperty.getList();

        assertThat(fruitData.get(0).get("name"),is("banana"));
        assertThat(fruitData.get(0).get("color"),is("yellow"));

        assertThat(fruitData.get(1).get("name"),is("apple"));
        assertThat(fruitData.get(1).get("color"),is("red"));

        assertThat(fruitData.get(2).get("name"),is("water melon"));
        assertThat(fruitData.get(2).get("color"),is("green"));
    }
}
```

#####  pojo를 생성하고 적용
- Fruit POJO 생성하기 : /com/demo/pojo/Fruit.java
```java
@Data
public class Fruit{
  private String name;
  private String color;
}
```

- FruitProperty 클래스 코드에 Fruit POJO 타입 반영하기
```java
@Data
@Component
@ConfigurationProperties("fruit")
public class FruitProperty{
  private List<Fruit> list;
}
```
- Fruit POJO 매핑 테스트


```java
package com.example.demo2.pojo;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

import java.util.List;
import java.util.Map;

import static org.hamcrest.Matchers.is;
import static org.junit.Assert.assertThat;

@RunWith(SpringRunner.class)
@SpringBootTest
public class PropertyTest {

    @Autowired
    FruitProperty fruitProperty;

    @Test
    public void test(){
        List<Fruit> fruitData = fruitProperty.getList();

        assertThat(fruitData.get(0).getName(),is("banana"));
        assertThat(fruitData.get(0).getColor(),is("yellow"));

        assertThat(fruitData.get(1).getName(),is("apple"));
        assertThat(fruitData.get(1).getColor(),is("red"));

        assertThat(fruitData.get(2).getName(),is("water melon"));
        assertThat(fruitData.get(2).getColor(),is("green"));
    }
}
```

-  @`@ConfigurationProperties`의 유연한 바인딩
  - 유연한 바인딩이란 프로퍼티값을 객체에 바인딩할 경우 필드를 낙타 표기법으로 선언하고 프로퍼티의 키는 다양한 형식으로 선언하여 바인딩 할 수 있다.

## 2.5 자동 환경 설정 이해하기
- 자동 환경 설정은 스프링 부트의 장점이다.
- 자동 설정은 `@EnableAutoConfiguration`또는 `@SpringBootApplication` 중 하나를 사용하면 된다.

### 2.5.1 자동 환경 설정 어노테이션
- SpringBootApplication
  - @SpringBootConfiguration : 스프링 부트의 설정을 나타내는 어노테이션이다. 스프링 부트 전용으로 사용한다. 필수 어노테이션 중 하나이다.
  - @EnableAutoConfiguration : 자동 설정의 핵심 어노테이션이다. 클래스 경로에 지정된 내용을 기반으로 영리하게 설정 자동화를 수행한다. 특별한 설정값을 추가하지 않으면 기본값으로 작동한다.
  - @ComponenetScan : 특정 패키지 경로를 기반으로 @Configuration에서 사용할 @Component 설정 클래스를 찾습니다. 별도의 경로를 설정하지 않으면 @ComponenetScan이 위치한 패키지 루트 경로로 설정된다.
- `@SrpingBootApplication` + `@EnableAutoConfiguration` + `@ComponenetScan` = `@SpringBootApplication`
