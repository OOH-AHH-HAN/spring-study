# 1. H2 데이터 베이스 설치

### 링크
![Untitled](https://user-images.githubusercontent.com/62877858/209468712-95391357-8d81-4e22-9575-8f403390acb1.png)

### 연결
![Untitled 1](https://user-images.githubusercontent.com/62877858/209468724-ed716b26-8fc4-4317-be1f-0df7bf4d8fb7.png)


JDBC URL : 파일 경로를 말함.

jdbc:h2:tcp://localhost/~/test

이렇게 접근을 하면 파일에 직접 접근이 아니라, 소켓을 통해서 접근하게 된다.

### 연결 시 오류사항

1) 디렉토리를 못 찾는다고 경고문이 뜰 때, 

H2 데이터베이스를 종료하고 다시 시작한다.

2) 연결이 안될 때,

앞에 ip주소 앞에 `localhost:` 를 적고 엔터 하면 됨.

### 연결 완료

![Untitled 2](https://user-images.githubusercontent.com/62877858/209468734-0d4f0eb9-cba0-471a-aaa3-345e3a3db626.png)

# 2. 순수 JDBC

자바는 db와 붙으려면 JDBC가 있어야한다.

### jdbc, h2 데이터베이스 라이브러리 추가 : build.gradle

```
dependencies{
		implementation 'org.springframework.boot:spring-boot-starter-jdbc'
		runtimeOnly 'com.h2database:h2'
}
```

### 접속정보 : application.properties

```
spring.datasource.url=jdbc:h2:tcp://localhost/~/test
spring.datasource.driver-class=org.h2.Driver
spring.datasource.username=sa
```

### JdbcMemberRepository

```java
package hello.hellospring.repository;
import hello.hellospring.domain.Member;
import org.springframework.jdbc.datasource.DataSourceUtils;
import javax.sql.DataSource;
import java.sql.*;
import java.util.ArrayList;
import java.util.List;
import java.util.Optional;

public class JdbcMemberRepository implements MemberRepository {
        //DB에 붙으려면 DataSource가 필요함.
        private final DataSource dataSource;

        //스프링한테 주입 받아야함.
        //스프링한테 dataSource를 주입 받으면 됨.
        public JdbcMemberRepository(DataSource dataSource) {
            this.dataSource = dataSource;
        }

        @Override
        public Member save(Member member) {
            String sql = "insert into member(name) values(?)";
            Connection conn = null;
            PreparedStatement pstmt = null;
            ResultSet rs = null; // 결과를 받는 것
            try {
                conn = getConnection(); // 커넥션 가지고 오는 것
                pstmt = conn.prepareStatement(sql, Statement.RETURN_GENERATED_KEYS);

                pstmt.setString(1, member.getName()); // sql문에 물음표에 담긴다.

                pstmt.executeUpdate();
                rs = pstmt.getGeneratedKeys(); //아이디 자동으로 들어간 것을 꺼내줌.

                if (rs.next()) {
                    member.setId(rs.getLong(1));
                } else {
                    throw new SQLException("id 조회 실패");
                }
                return member;
            } catch (Exception e) {
                throw new IllegalStateException(e);
            } finally {
                close(conn, pstmt, rs);
                //DB가 계속 연결 되는 것을 방지하기 위해 필수로 close를 해야한다.
            }
        }

        @Override
        public Optional<Member> findById(Long id) {
            String sql = "select * from member where id = ?";

            Connection conn = null;
            PreparedStatement pstmt = null;
            ResultSet rs = null;

            try {
                conn = getConnection();
                pstmt = conn.prepareStatement(sql);
                pstmt.setLong(1, id);

                rs = pstmt.executeQuery();// 조회

                if(rs.next()) {
                    Member member = new Member();
                    member.setId(rs.getLong("id"));
                    member.setName(rs.getString("name"));
                    return Optional.of(member);
                } else {
                    return Optional.empty();
                }
            } catch (Exception e) {
                throw new IllegalStateException(e);
            } finally {
                close(conn, pstmt, rs);
            }
        }

        @Override
        public List<Member> findAll() {
            String sql = "select * from member";
            Connection conn = null;
            PreparedStatement pstmt = null;
            ResultSet rs = null;
            try {

                conn = getConnection();
                pstmt = conn.prepareStatement(sql);
                rs = pstmt.executeQuery();

                //리스트에 루프 돌려서 담는다.
                List<Member> members = new ArrayList<>();
                while(rs.next()) {
                    Member member = new Member();
                    member.setId(rs.getLong("id"));
                    member.setName(rs.getString("name"));
                    members.add(member);
                }
                return members;
            } catch (Exception e) {
                throw new IllegalStateException(e);
            } finally {
                close(conn, pstmt, rs);
            }
        }

        @Override
        public Optional<Member> findByName(String name) {
            String sql = "select * from member where name = ?";
            Connection conn = null;
            PreparedStatement pstmt = null;
            ResultSet rs = null;
            try {
                conn = getConnection();
                pstmt = conn.prepareStatement(sql);
                pstmt.setString(1, name);
                rs = pstmt.executeQuery();
                if(rs.next()) { Member member = new Member();
                    member.setId(rs.getLong("id"));
                    member.setName(rs.getString("name"));
                    return Optional.of(member);
                }
                return Optional.empty();
            } catch (Exception e) {
                throw new IllegalStateException(e);
            } finally {
                close(conn, pstmt, rs);
            }
        }

        private Connection getConnection() {
            return DataSourceUtils.getConnection(dataSource);
        }

        private void close(Connection conn, PreparedStatement pstmt, ResultSet rs)
        {
            try {
                if (rs != null) {
                    rs.close();
                }
            } catch (SQLException e) {
                e.printStackTrace();
            }
            try {
                if (pstmt != null) {
                    pstmt.close();
                }
            } catch (SQLException e) {
                e.printStackTrace();
            }
            try {
                if (conn != null) {
                    close(conn);
                }
            } catch (SQLException e) { e.printStackTrace();
            }
        }

        private void close(Connection conn) throws SQLException {
            DataSourceUtils.releaseConnection(conn, dataSource);
        }//닫을 때는, DataSourceUtils 를 통해서 릴리즈를 해줘야 함.
    }
```

`DataSourceUtils` 통해서 커넥션을 얻어야함. 그래야 이전에 트랜잭션이 걸릴 수가 있는데 데이터베이스 연결을 똑같이 유지 시켜줘야함. 데이터베이스 연결을 유지를 위해

```java
private Connection getConnection() {
            return DataSourceUtils.getConnection(dataSource);
        }
```

닫을 때는, `DataSourceUtils` 를 통해서 릴리즈를 해줘야 함.

```java
private void close(Connection conn) throws SQLException {
            DataSourceUtils.releaseConnection(conn, dataSource);
        }
```

애플리케이션 조립하는 코드만 조립하면 손댈 것이 없는 것이 스프링의 장점

members

기능을 완전히 변경을 해도 수정을 하지 않아도 됨

`DataSourceUtils` 통해서 커넥션을 얻어야함. 그래야 이전에 트랜잭션이 걸릴 수가 있는데 데이터베이스 연결을 똑같이 유지 시켜줘야함. 데이터베이스 연결을 유지를 위해

```java
private Connection getConnection() {
            return DataSourceUtils.getConnection(dataSource);
        }
```

닫을 때는, `DataSourceUtils` 를 통해서 릴리즈를 해줘야 함.

```java
private void close(Connection conn) throws SQLException {
            DataSourceUtils.releaseConnection(conn, dataSource);
        }
```

### SpringConfig

```java
package hello.hellospring;

import hello.hellospring.repository.JdbcMemberRepository;
import hello.hellospring.repository.MemberRepository;
import hello.hellospring.repository.MemoryMemberRepository;
import hello.hellospring.service.MemberService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import javax.sql.DataSource;

@Configuration
public class SpringConfig {

    private final DataSource dataSource;
    //스프링 부트가 자체적으로 bean을 만들어주고,datasource를 만들어주고 주입을 해줌.

    @Autowired
    public SpringConfig(DataSource dataSource){
        this.dataSource = dataSource;
    }

    @Bean
    public MemberService memberService(){
        return new MemberService(memberRepository());
    }

    @Bean
    public MemberRepository memberRepository(){
        //return new MemoryMemberRepository();
        return new JdbcMemberRepository(dataSource);
    }
}
```

애플리케이션 조립하는 코드만 조립하면 손댈 것이 없는 것이 스프링의 장점

### 스프링 컨테이너
![Untitled 3](https://user-images.githubusercontent.com/62877858/209468768-5f15e3d9-59df-46f5-bd0b-f13a462eb9be.png)

- 개방-폐쇄 원칙

확장에는 열려있고, 수정, 변경에는 닫혀있다.

# 스프링 통합 테스트

### MemberServiceIntegrationTest.java

```java
package hello.hellospring.service;
import hello.hellospring.domain.Member;
import hello.hellospring.repository.MemberRepository;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.transaction.annotation.Transactional;
import static org.assertj.core.api.Assertions.assertThat;
import static org.junit.jupiter.api.Assertions.assertEquals;
import static org.junit.jupiter.api.Assertions.assertThrows;

@SpringBootTest
@Transactional
class MemberServiceIntegrationTest {
    //테스트 케이스는 필드 기반으로 Autowired 받는게 편할 것이다.
    @Autowired MemberService memberService;
    @Autowired MemberRepository memberRepository;

    @Test
    public void 회원가입() throws Exception {
        //Given
        Member member = new Member();
        member.setName("hello");
        //When
        Long saveId = memberService.join(member);
        //Then
        Member findMember = memberRepository.findById(saveId).get();
        assertEquals(member.getName(), findMember.getName());
    }
    @Test
    public void 중복_회원_예외() throws Exception {
        //Given
        Member member1 = new Member();
        member1.setName("spring");
        Member member2 = new Member();
        member2.setName("spring");
        //When
        memberService.join(member1); IllegalStateException e = assertThrows(IllegalStateException.class,
                () -> memberService.join(member2));//예외가 발생해야 한다.
        assertThat(e.getMessage()).isEqualTo("이미 존재하는 회원입니다.");
    }
}
```

`@SpringBootTest` : 스프링 컨테이너와 테스트를 함께 실행한다.

### 트랜잭션

데이터베이스는 트랜잭션이라는 개념이 있는데, 커밋을 해줘야 반영이 된다.

커밋하기전까지의 반영은 되지 않는다.

`@Transctional`

애노테이션을 테스트 케이스에 달면, 테스트가 끝나면 롤백을 해줘서 db에 넣었던 데이터가 반영이 되지 않고 지워진다.

기존처럼 지우는 코드를 넣지 않고 테스트를 계속 반복할 수 있다는 장점이 있다.

### 순수한 단위테스트 좋은 테스트일 확률이 크다.

좋은 테스트라고 표현하기 애매하지만 좋은 테스트일 확률이 크다.

컨테이너까지 올리게 되면 테스트가 잘못 될 수 있는 가능성이 크다.

# 스프링 JdbcTemplate

순수 Jdbc와 동일한 환경설정을 하면 된다.

스프링 JdbcTemplate과 MyBatis 같은 라이브러리는 JDBC API에서 본 반복 코드를 대부분
제거해준다. 하지만 SQL은 직접 작생해야 한다.

JDBC 템플릿은 실무에서 많이 사용함.

### JdbcTemplateMemberRepository

```java
package hello.hellospring.repository;
import hello.hellospring.domain.Member;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.jdbc.core.RowMapper;
import org.springframework.jdbc.core.namedparam.MapSqlParameterSource;
import org.springframework.jdbc.core.simple.SimpleJdbcInsert;
import javax.sql.DataSource;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.util.HashMap;
import java.util.List;import java.util.Map;
import java.util.Optional;

public class JdbcTemplateMemberRepository implements MemberRepository {

    private final JdbcTemplate jdbcTemplate;

    public JdbcTemplateMemberRepository(DataSource dataSource) {
        jdbcTemplate = new JdbcTemplate(dataSource);
    }

    @Override
    public Member save(Member member) {
        SimpleJdbcInsert jdbcInsert = new SimpleJdbcInsert(jdbcTemplate);
        jdbcInsert.withTableName("member").usingGeneratedKeyColumns("id");

        Map<String, Object> parameters = new HashMap<>();
        parameters.put("name", member.getName());

        Number key = jdbcInsert.executeAndReturnKey(new MapSqlParameterSource(parameters));
        member.setId(key.longValue());
        return member;
    }

    @Override
    public Optional<Member> findById(Long id) {
        List<Member> result = jdbcTemplate.query("select * from member where id = ?", memberRowMapper(), id);
        return result.stream().findAny();
    }

    @Override
    public List<Member> findAll() {
        return jdbcTemplate.query("select * from member", memberRowMapper());
    }

    @Override
    public Optional<Member> findByName(String name) { List<Member> result = jdbcTemplate.query("select * from member where name = ?", memberRowMapper(), name);
        return result.stream().findAny();
    }

    private RowMapper<Member> memberRowMapper() {
        return (rs, rowNum) -> {
            Member member = new Member();
            member.setId(rs.getLong("id"));
            member.setName(rs.getString("name"));
            return member;
        };
    }
}
```

### SpringConfig

```java
package hello.hellospring;

import hello.hellospring.repository.JdbcMemberRepository;
import hello.hellospring.repository.JdbcTemplateMemberRepository;
import hello.hellospring.repository.MemberRepository;
import hello.hellospring.repository.MemoryMemberRepository;
import hello.hellospring.service.MemberService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import javax.sql.DataSource;

@Configuration
public class SpringConfig {

    private final DataSource dataSource;
    //스프링 부트가 자체적으로 bean을 만들어주고,datasource를 만들어주고 주입을 해줌.

    @Autowired
    public SpringConfig(DataSource dataSource){
        this.dataSource = dataSource;
    }

    @Bean
    public MemberService memberService(){
        return new MemberService(memberRepository());
    }

    @Bean
    public MemberRepository memberRepository(){
        //return new MemoryMemberRepository();
        /*return new JdbcMemberRepository(dataSource);*/
        return new JdbcTemplateMemberRepository(dataSource);
    }
}
```

순수한 JDBC와 비교했을 때, 소스코드 라인 수부터 크게 차이가 난다.

### 테스트 코드의 비중

실무 개발자들은 실제로 60% 테스트 코드 작성하는데 비중을 둔다.

시스템 장애가 발생시 금액 손실의 규모가 클 수가 있기에 장애를 사전에 방지하기 위해서 테스트 코드에 비중이 높다.

# JPA

SQL은 개발자가 직접 작성해야하는 문제

JPA는 SQL 쿼리도 자동으로 처리해준다.

엔티티를 매핑해야한다.

jpa라는 것은 인터페이스를 제공하는 것

구현체는 hibernate라는 것을 쓰는 것이다.

ORM

Object

Relational

Mapping

DB가 알아서 생성해주는 것은 IDENTITY

### 컬럼 매핑

아이디

```java
@Id @GeneratedValue(strategy = GenerationType.IDENTITY)
private Long id;
```

아이디가 아닌 컬럼명

```java
@Column(name = userName)
private String userName;
```

### build.gradle

```java
plugins {
	id 'java'
	id 'org.springframework.boot' version '2.7.6'
	id 'io.spring.dependency-management' version '1.0.15.RELEASE'
}

group = 'hello'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = '11'

repositories {
	mavenCentral()
}

dependencies {
	implementation 'org.springframework.boot:spring-boot-starter-thymeleaf'
	implementation 'org.springframework.boot:spring-boot-starter-web'
	/*implementation 'org.springframework.boot:spring-boot-starter-jdbc'*/
	implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
	runtimeOnly 'com.h2database:h2'
	testImplementation 'org.springframework.boot:spring-boot-starter-test'
}

tasks.named('test') {
	useJUnitPlatform()
}
```

### JpaMemberRepository

```java
package hello.hellospring.repository;

import hello.hellospring.domain.Member;

import javax.persistence.EntityManager;
import java.util.List;
import java.util.Optional;

public class JpaMemberRepository implements MemberRepository {

    private final EntityManager em;

    public JpaMemberRepository(EntityManager em){
        this.em = em;
    }

    @Override
    public Member save(Member member) {
        em.persist(member);
        return member;
    }

    @Override
    public Optional<Member> findById(Long id) {
        Member member = em.find(Member.class, id);
        return Optional.ofNullable(member);
    }

    @Override
    public Optional<Member> findByName(String name) {
        List<Member> result = em.createQuery("select m from Member m where m.name = :name", Member.class)
                .setParameter("name", name)
                .getResultList();

        return result.stream().findAny();
    }

    @Override
    public List<Member> findAll() {
        return em.createQuery("select m from Member m", Member.class)
                .getResultList();
    }
}
```

### MemberService

```java
package hello.hellospring.service;

import hello.hellospring.domain.Member;
import hello.hellospring.repository.MemberRepository;
import hello.hellospring.repository.MemoryMemberRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import java.util.List;
import java.util.Optional;

@Transactional
public class MemberService {

    private final MemberRepository memberRepository;

    public MemberService(MemberRepository memberRepository){
        this.memberRepository = memberRepository;
    }
    // 직접 new로 생성하는게 아니라 외부에서 넣어주도록 바꿔줘야함.
    // DI

    /**
     * 회원 가입
     */
    public Long join(Member member){

        validateDuplicateMember(member); // 중복회원검증
        memberRepository.save(member);
        return member.getId();
    }

    private void validateDuplicateMember(Member member) {
        memberRepository.findByName(member.getName())
                .ifPresent(m -> {
                    throw new IllegalStateException("이미 존재하는 회원이 입니다.");
                });
    }

    /**
     * 전체 회원 조회
     */
    public List<Member> findMembers(){
        return memberRepository.findAll();
    }

    public Optional<Member> findOne(Long memberId){
        return memberRepository.findById(memberId);
    }
}
```

### SpringConfig

```java
package hello.hellospring;

import hello.hellospring.repository.*;
import hello.hellospring.service.MemberService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import javax.persistence.EntityManager;
import javax.sql.DataSource;

@Configuration
public class SpringConfig {
/*

    private final DataSource dataSource;
    //스프링 부트가 자체적으로 bean을 만들어주고,datasource를 만들어주고 주입을 해줌.

    @Autowired
    public SpringConfig(DataSource dataSource){
        this.dataSource = dataSource;
    }
*/
    private EntityManager em;

    @Autowired
    public SpringConfig(EntityManager em){
        this.em = em;

    }

    @Bean
    public MemberService memberService(){
        return new MemberService(memberRepository());
    }

    @Bean
    public MemberRepository memberRepository(){
        //return new MemoryMemberRepository();
        /*return new JdbcMemberRepository(dataSource);*/
        /*return new JdbcTemplateMemberRepository(dataSource);*/
        return new JpaMemberRepository(em);
    }
}
```

JPA는 EntityManager로 통해서 동작한다.

다른 것과는 다르게 테이블 대상이 아니라,

객체 대상으로 쿼리를 날리는 것

PK 기반이 아닌 것들은 jpaql을 작성해줘야 함.

JPA를 사용할거면 트랜잭션이 항상 있어야 한다.

# 스프링 데이터 JPA

JPA를 먼저 학습한 후에, 스프링 데이터 JPA를 학습해야한다.

스프링 데이터 JPA는 JPA를 편리하게 사용하도록 도와주는 기술이다.

### SpringDataJpaMemberRepository

```java
package hello.hellospring.repository;

import hello.hellospring.domain.Member;
import org.springframework.data.jpa.repository.JpaRepository;

import java.util.Optional;

public interface SpringDataJpaMemberRepository extends JpaRepository<Member, Long>, MemberRepository {

    @Override
    Optional<Member> findByName(String name);
}
```

### SpringConfig

```java
package hello.hellospring;

import hello.hellospring.repository.*;
import hello.hellospring.service.MemberService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import javax.persistence.EntityManager;
import javax.sql.DataSource;

@Configuration
public class SpringConfig {
/*

    private final DataSource dataSource;
    //스프링 부트가 자체적으로 bean을 만들어주고,datasource를 만들어주고 주입을 해줌.

    @Autowired
    public SpringConfig(DataSource dataSource){
        this.dataSource = dataSource;
    }
*/
  /*
    private EntityManager em;

    @Autowired
    public SpringConfig(EntityManager em){
        this.em = em;

    }
*/

    private final MemberRepository memberRepository;

    @Autowired
    public SpringConfig(MemberRepository memberRepository){
        this.memberRepository = memberRepository;
    }

    @Bean
    public MemberService memberService(){
        return new MemberService(memberRepository);
    }
/*
    @Bean
    public MemberRepository memberRepository(){
        //return new MemoryMemberRepository();
        *//*return new JdbcMemberRepository(dataSource);*//*
        *//*return new JdbcTemplateMemberRepository(dataSource);*//*
        *//*return new JpaMemberRepository(em);*//*
    }
    */
}
```

- 구현체가 자동으로 만들어줘서 스프링 빈에 자동으로 등록이 된다.
- SpringDataJpa는 구현체를 자동으로 만들어서 빈에 자동으로 등록을 한다.
- SpringDataJpa는 JPA 가져다 쓰는 것
- JpaRepository에서 기본적인 CRUD, 단순조회가 제공이 되서 가져와서 쓰면 된다.
- 객체의 이름이 다를 수가 있다. 이것은 공통화가 불가능 하다.
- 그래서 메서드 이름만으로도 조회 기능 제공해준다.
- 실무에서 JPA와 스프링 데이터 JPA를 기본으로 사용하고, 복잡한 동적쿼리는 Querydsl이라는 라이브러리를 사용하면 된다.
