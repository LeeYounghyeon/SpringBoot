# 3. 스프링 부트 테스트

## 3.1 @SpringBootTest
- `@SpringBootTest`는 통합 테스트를 제공하는 기본적인 스프링 부트 테스트 어노테이션
- 설정을 임의로 바꾸어서 테스트 할 수 있다.
- 실제 국동되는 애플리케이션과 똑같이 애플리케이션 컨텍스트를 로드하여 테스트한다.
- `@RunWith`:JUint에 내장된 러너를 사용하는 대신 어노테이션에 정의된 러너 클래스를 사용한다. !꼭 붙여야한다!
- 파라미터

      - value : 테스트가 실행되기 전에 적용할 프로퍼티를 주입시킬 수 있다.
      - properties : 테스트가 실행되기 전에 {key = value}형식으로 프로퍼티를 추가할 수 있다.
      - classes : 애플리케이션 컨텍스트에 로드할 클래스를 지정할 수 있다. 따로 지정하지 않으면 `@SpringBootConfiguration`을 찾아서 로드
      - webEnvironment : 애플리케이션이 실행될 때의 웹 환경을 설정할 수 있다.


## 3.2 @WebMvcTest
- 웹에서 테스트하기 힘든 컨트롤러를 테스트하는 데 적합하다.
- 요청과 응답에 대해 테스트할 수 있다.
- 자동으로 테스트하며 수동으로 추가/삭제 가능

/com/havi/domain/Book.java
```java
import java.time.LocalDateTime;

import lombok.Builder;
import lombok.Getter;
import lombok.NoArgsConstructor;

@NoArgsConstructor //파라미터가 없는 생성자를 만들어준다.
@Getter
public class Book{
  private Integer idx;
  private String title;
  private LocalDateTime publishedAt;

  @Builder //Builder패턴을 적용한 객체 생성 메소드/클래스를 만들어준다.
  public Book(String title, LocalDateTime publishedAt){
    this.title = title;
    this.publishedAt = publishedAt;
  }
}
```

/com/havi/controller/BookController.java
```java
package com.havi.controller;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;

@Controller
public class BookController {

    @Autowired
    private BookService bookService;

    @GetMapping("/books")
    public String getBookList(Model model){
        model.addAttribute("bookList",bookService.getBookList());
        return "book";
    }
}
```
- /books로 요청 시 현재 BookService 클래스에 책 목록을 요청하여 "bookList" 라는 키값으로 데이터값을 넘기는 컨트롤러를 만든다. 반환되는 뷰 이름은 'book'이다.

/com/havi/service/BookService.java
```java
package com.havi.service;

import com.havi.domain.Book;

import java.util.List;

public interface BookService {
    List<Book> getBookList();
}
```
- 구현체를 만들지 않고 목 데이터를 이용해 테스트를 진행한다.


/com/havi/BookController.java
```java
package com.havi;

import com.havi.controller.BookController;
import com.havi.domain.Book;
import com.havi.service.BookService;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
import org.springframework.boot.test.mock.mockito.MockBean;
import org.springframework.test.context.junit4.SpringRunner;
import org.springframework.test.web.servlet.MockMvc;

import java.time.LocalDateTime;
import java.util.Collections;

import static org.hamcrest.Matchers.contains;
import static org.mockito.BDDMockito.given;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;

@RunWith(SpringRunner.class)
@WebMvcTest(BookController.class)
public class BookControllerTest {

    @Autowired
    private MockMvc mvc;

    @MockBean //각자 객체로 대체 (목 객체)
    private BookService bookService;

    @Test
    public void Book_MVC_테스트() throws Exception{
        Book book = new Book("Spring Boot Book", LocalDateTime.now());
        given(bookService.getBookList()).willReturn(Collections.singletonList(book));
        //getBookList()메서드의 실행에 대한 반환값을 미리 정의

        mvc.perform(get("/books"))
                .andExpect(status().isOk()) //HTTP상태값이 200인지 테스트
                .andExpect(view().name("book"))//반환되는 뷰의 이름이 'book'인지 테스트
                .andExpect(model().attributeExists("bookList"))//모델의 프로퍼티 중 'bookList'라는 프로퍼티가 존재하는지 테스트
                .andExpect(model().attribute("bookList",contains(book)));//모델의 프로퍼티 중 'bookList'라는 프로퍼티에 book 객체가 담겨져 있는지 테스트
    }
}
```

- MockMvc는 컨트롤러 테스트 시 모든 의존성을 로드하는 것이 아닌 BookController 관련 빈만 로드하여 가벼운 MVC테스트를 수행한다.


## 3.3 @DataJpaTest
- JPA 관련 테스트 설정만 로드한다.
- 데이터소스의 설정이 정상적인지, 데이터를 제대로 생성,수정,삭제 하는지 등의 테스트가 가능하다.
- 실제 데이터를 사용하지 않고 테스트 가능하다.
- `@AutoConfigureTestDatabase`
  - Replace.Any : 사용하면 기본적으로 내장된 데이터 소스를 사용한다.
  - Replace.NONE : @ActiveProfiles에 설정한 프로파일 환경값에 따라 데이터 소스가 적용된다.
- 자동 설정 방식
  - application.yml 설정을 spring.test.database.replace: NONE 으로 변경
- `@DataJpaTest` : 테스트가 끝날 때마다 자동으로 테스트에 사용한 데이터 롤백
- 데이터 베이스 사용 설정
  - spring.test.database.connection: H2
  - `@AutoConfigureTestDatabase(connection = H2)`
- `@Entity` : 해당 클래스가 엔티티임을 알리기 위해 사용,자동검색을 통하여 이 어노테이션이 선언 된 클래스들은 엔티티 빈으로 등록한다.
- `@Table` : 데이터의 저장소, 테이블을 의미한다.
- `@Id` : 엔티티의 기본키를 의미한다.
- `@GeneratedValue` : DB에 의해 자동으로 생성된 값. 실제 데이터가 저장될 때 생성되는 값이다.
- `@Column` : 필드와 테이블의 컬럼을 매핑시켜준다.
