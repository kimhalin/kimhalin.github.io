---
layout: post
title: Spring boot와 GraphQL
# description: >
#   Howdy! This is an example blog post that shows several types of HTML content supported in this theme.
sitemap: false
hide_last_modified: true
---









![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/570ea375-9fee-43aa-8818-40cd72400957/Untitled.png)

Spring boot와 GraphQL을 사용해 아주 간단한 게시판 CRUD를 작성해봤다.

생각보다 Spring boot과 GraphQL 자료가 많이 없어 처음에 프로젝트 구조를 짤 때 어떻게 해야 할지 감이 잘 잡히지 않았다. 

Netflix에서 만든 프레임워크 dgs도 있었고, Spring for GraphQL도 있었고, 등등 아직 생긴지 얼마 되지 않아 아직 이게 최고다! 하는 그런 게 없어서 보는 인터넷마다 build.gradle 내용이 너무나도 다르고, 프로젝트 구조도 달랐다.. 

나는 Spring for GraphQL을 사용해 작성하였다. 이게 Spring에서 주로 사용하는 MVC에 사용할 수 있도록 해주기 때문에 생각보다 REST API와 비슷하게 코드를 작성할 수 있어서 사용했다.

나중에 혹시 모를 상황을 대비해 기록

### Build.gradle

```java
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
    implementation 'org.springframework.boot:spring-boot-starter-graphql'
    implementation 'org.springframework.boot:spring-boot-starter-web'

    compileOnly 'org.projectlombok:lombok'
    developmentOnly 'org.springframework.boot:spring-boot-devtools'
    runtimeOnly 'org.postgresql:postgresql'
    annotationProcessor 'org.projectlombok:lombok'

    testImplementation 'org.springframework.boot:spring-boot-starter-test'
    testImplementation 'org.springframework:spring-webflux'
    testImplementation 'org.springframework.graphql:spring-graphql-test'
}
```

### Board

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
@Builder
@Entity
public class Board {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)

    private Long id;
    private String title;
    private String content;
}

```

### BoardRepository

```java
@GraphQlRepository
public interfaceBoardRepositoryextendsJpaRepository<Board, Long> {
}
```

### BoardService

```java
@Slf4j
@RequiredArgsConstructor
@Service
public class BoardService {
    private finalBoardRepositoryboardRepository;

    // 게시글 1개 가져오기
    public Board getBoard(Long id) {
        return boardRepository.findById(id)
                .orElseThrow(EntityNotFoundException::new);
    }

    // 게시글 목록 가져오기
    publicList<Board> getBoardList() {
        return boardRepository.findAll();
    }

    public Board save(BoardDto boardDto) {
        System.out.println("boardDto = " + boardDto);
        Board board = Board.builder()
                .title(boardDto.getTitle())
                .content(boardDto.getContent()).build();
        return boardRepository.save(board);
    }

    // 게시글 수정
    public Board update(BoardDto boardDto) {
        Board board= boardRepository.findById(boardDto.getId())
                .orElseThrow(EntityNotFoundException::new);
        board.setTitle(boardDto.getTitle());
        board.setContent(boardDto.getContent());

        return boardRepository.save(board);
    }

    // 게시글 삭제
    public void delete(Long id) {
        Board board = boardRepository.findById(id)
                .orElseThrow(EntityNotFoundException::new);
        boardRepository.delete(board);
    }

}

```

### BoardController

```java
@Slf4j
@RequiredArgsConstructor
@Controller
public class BoardController {

    private final BoardService boardService;

    @QueryMapping
    public Board Board(@Argument Long id) {
        return boardService.getBoard(id);
    }

    @QueryMapping
    public List<Board> Boards() {
        return boardService.getBoardList();
    }

    @MutationMapping
    public Board createBoard(@Argument BoardDto boardInput) {
        return boardService.save(boardInput);
    }

    @MutationMapping
    public Board updateBoard(@Argument BoardDto boardInput) {
        return boardService.update(boardInput);
    }

    @MutationMapping
    public Boolean deleteBoard(@Argument Long id) {
        boardService.delete(id);
        return true;
    }
}
```

### schema.graphqls

```graphql
type Query {
    board(id: ID): Board
    boards: [Board]
}

type Board {
    id: ID
    title: String
    content: String
}

type Mutation {
    createBoard(boardInput: BoardInput): Board
    updateBoard(boardInput: BoardInput): Board
    deleteBoard(id: ID!): Boolean
}

input BoardInput {
    id: ID
    title: String
    content: String
}
```

### application.yml

```yaml
spring:
# Database
datasource:
    driver-class-name: org.postgresql.Driver
    url: jdbc:postgresql://localhost:5432/postgres
    username: admin_hl
    password:

# jpa properties
jpa:
    hibernate:
      ddl-auto: create-drop# When you launch the application for the first time - switch "none" at "create"
show-sql: true
    database:postgresql
database-platform: org.hibernate.dialect.PostgreSQLDialect
    open-in-view: false
    generate-ddl: true

  graphql:
    graphiql:
      enabled: true
    cors:
      allowed-origins: "*"
    schema:
      printer:
        enabled: true
```

GraphQL도 너무나도 처음이고, Spring boot도 아직 익숙치 않아 많이 헤맸었다. 아직 GraphQL 관련 annotation에 관하여 잘 모르는 상태로 사용했기 때문에 이 부분에 대하여 더 공부해야겠다.


