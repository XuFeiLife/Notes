# maven依赖

```xml
<dependency>  
    <groupId>org.opensearch.client</groupId>  
    <artifactId>spring-data-opensearch-starter</artifactId>  
    <version>1.3.0</version>  
</dependency>
```

# 配置文件

```yml
spring:  
  application:  
    name: demo  
  
  opensearch:  
    uris: http://localhost:9200  
    connection:  
      connect-timeout: 5s  
      socket-timeout: 30s
```

# 代码示例

Index

```java
package com.hans.springboot.demo.opensearch;  
  
import lombok.AllArgsConstructor;  
import lombok.Builder;  
import lombok.Data;  
import lombok.NoArgsConstructor;  
import org.springframework.data.annotation.Id;  
import org.springframework.data.elasticsearch.annotations.Document;  
import org.springframework.data.elasticsearch.annotations.Field;  
import org.springframework.data.elasticsearch.annotations.FieldType;  
  
@Data  
@AllArgsConstructor  
@NoArgsConstructor  
@Builder  
@Document(indexName = "book")  
public class Book {  
    @Id  
    private String id;  
  
    @Field(type = FieldType.Text, name = "title")  
    private String title;  
  
    @Field(type = FieldType.Integer, name = "page")  
    private Integer page;  
  
    @Field(type = FieldType.Text, name = "isbn")  
    private String isbn;  
  
    @Field(type = FieldType.Text, name = "description")  
    private String description;  
  
    @Field(type = FieldType.Text, name = "language")  
    private String language;  
  
    @Field(type = FieldType.Double, name = "price")  
    private Double price;  
}
```

Repository
```java
package com.hans.springboot.demo.opensearch;  
  
import org.springframework.data.elasticsearch.repository.ElasticsearchRepository;  
import org.springframework.stereotype.Repository;  
  
@Repository  
public interface BookRepository extends ElasticsearchRepository<Book, String> {  
}
```

Service and ServiceImpl
```java

package com.hans.springboot.demo.opensearch;  
  
  
  
import java.util.List;  
  
public interface BookService {  
    List<Book> getAll();  
  
    Book add(Book book);  
  
    Book getById(String id);  
  
    Book update(Book book, String id);  
  
    void deleteById(String id);  
}
```

```java
package com.hans.springboot.demo.opensearch;  
  
import lombok.extern.slf4j.Slf4j;  
import org.springframework.stereotype.Service;  
  
import java.util.List;  
import java.util.Spliterators;  
import java.util.stream.StreamSupport;  
  
@Slf4j  
@Service  
public class BookServiceImpl implements BookService {  
  
    private final BookRepository repository;  
  
    public BookServiceImpl(BookRepository repository) {  
        this.repository = repository;  
    }  
  
    @Override  
    public List<Book> getAll() {  
        return StreamSupport.stream(  
                        Spliterators.spliteratorUnknownSize(repository.findAll().iterator(), 0), false)  
                .toList();  
    }  
  
    @Override  
    public Book add(Book book) {  
        log.info("addBook : {} " , book );  
        return repository.save(book);  
    }  
  
    @Override  
    public Book getById(String id) {  
        return repository.findById(id).orElseThrow(() -> new RuntimeException("Book id not found"));  
    }  
  
    @Override  
    public Book update(Book book, String id) {  
         repository.findById(id)  
                .ifPresentOrElse(book1 -> {  
                    book1.setTitle(book.getTitle());  
                    book1.setIsbn(book.getIsbn());  
                    book1.setDescription(book.getDescription());  
                    book1.setLanguage(book.getLanguage());  
                    book1.setPage(book.getPage());  
                    book1.setPrice(book.getPrice());  
                    repository.save(book1);  
                },() -> {throw new RuntimeException("Book id not found");});  
  
        return book;  
    }  
  
    @Override  
    public void deleteById(String id) {  
        repository.deleteById(id);  
    }  
}
```

Web
```java
package com.hans.springboot.demo.opensearch;  
  
import lombok.extern.slf4j.Slf4j;  
import org.springframework.http.HttpStatus;  
import org.springframework.http.ResponseEntity;  
import org.springframework.web.bind.annotation.*;  
  
import java.util.List;  
  
@RestController  
@RequestMapping("/api")  
@Slf4j  
public class BookController {  
  
    private final BookService bookService;  
  
    public BookController(BookService bookService) {  
        this.bookService = bookService;  
    }  
  
    @GetMapping("/book")  
    public ResponseEntity<List<Book>> getAllBooks() {  
        return ResponseEntity.ok().body(bookService.getAll());  
    }  
  
    @PostMapping("/book")  
    @ResponseStatus(HttpStatus.CREATED)  
    public Book addBook(@RequestBody Book book) {  
        return bookService.add(book);  
  
    }  
  
    @DeleteMapping("/book/{id}")  
    @ResponseStatus(HttpStatus.NO_CONTENT)  
    public ResponseEntity<?> deleteBookById(@PathVariable String id){  
        bookService.deleteById(id);  
  
        return ResponseEntity.ok().body("Done");  
    }  
  
    @GetMapping("/book/{id}")  
    public ResponseEntity<Book> getBookById(@PathVariable("id") String id) {  
        return ResponseEntity.ok().body(bookService.getById(id));  
    }  
  
  
    @PutMapping("/book/{id}")  
    @ResponseStatus(HttpStatus.OK)  
    public ResponseEntity<Book> updateBook(@RequestBody Book book, @PathVariable String id) {  
        var updatedBook =  bookService.update(book, id);  
        return ResponseEntity.ok().body(updatedBook);  
  
    }  
  
}
```

# Github地址
