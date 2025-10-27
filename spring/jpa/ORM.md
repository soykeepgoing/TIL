작성일 | 251027

# 0. Spring Framework 안에서

![image.png](https://docs.spring.io/spring-framework/docs/3.2.x/spring-framework-reference/html/images/spring-overview.png)

# **1. ORM**

> ***Object Relation Mapping*** 
자바 객체와 RDB 테이블을 자동으로 연결하고 변환해, 객체 지향 코드로 데이터를 조작할 수 있게 하는 기술
> 

## 1.1. 등장 배경

**자바 객체와 RDB의 데이터 구조는 서로 패러다임이 다르다.** 

- 자바 : 객체 지향 (클래스, 상속, 연관관계)
- DB : 관계형 (테이블, FK, 조인)

**JDBC로 직접 SQL을 작성하면** 

- JDBC는 자바에서 DB와 직접 연결할 수 있는 저수준 API
    
    JAVA Database Connectivity 
    
    자바에서 표준화된 방법으로 DB에 연결하는 api 
    
    Connection → Statement → Resultset 구조로 동작 
    
    SQL과 자바 객체를 직접 연결 
    

→ 매번 `INSERT`, `UPDATE`, `SELECT`를 수동 작성해야 하고

→ 객체 ↔ 테이블 매핑 코드를 직접 써야 한다. 

➡️ 이런 과정을 자동화하기 위해서 ORM 등장 

## 1.2. 핵심 역할

**객체를 데이터베이스 테이블로 자동으로 매핑 (Object ↔ Table)** 

- SQL 작성 없이 자바 코드로 데이터 조작 가능
- `entityManager.persist(user)` ↔ `INSERT INTO user …`

**Boilerplate Code 생성을 막아줌** 

- Boilerplate Code란 구조적으로 동일하게 반복되는 코드 패턴
- 만약 ORM을 사용하지 않는 경우, 아래의 보일러플레이트 코드가 필수적으로 사용됨
    
    ```java
    Connection conn = DriverManager.getConnection(...);
    PreparedStatement ps = conn.prepareStatement("INSERT INTO ...");
    ps.setString(1, user.getName());
    ps.executeUpdate();
    ps.close();
    conn.close();
    ```
    

➡️ 이런 과정을 자동화하기 위해서 ORM 등장 

# 2. JPA (Java Persistence API)

> ORM을 위한 표준 명세로 ORM을 이렇게 규약해야 한다는 규약을 제공한다. 
대표적인 구현체는 `Hibernate`가 존재한다.
> 

## 2.1. 역할

### JPA는 ORM을 위한 표준 명세이다.

- ORM 기술은 이렇게 만들어야 한다는 인터페이스 / 규약의 집합이다.
- 직접 동작하지 않는다.

https://jakarta.ee/specifications/persistence/3.2/

### Hibernate는 JPA의 대표적인 구현체이다.

![image.png](https://docs.hibernate.org/orm/7.1/introduction/html_single/images/api-overview.png)

JPA의 대표적인 구현체로 JPA의 인터페이스를 실제 코드로 구현한 라이브러이이다. 

JPA 확장 기능을 다수 제공한다. 

> https://docs.hibernate.org/orm/7.1/introduction/html_single/#hibernate-and-jpa
> 
> 
> As an application developer, you must decide whether to:
> 
> - write your program in terms of `Session` and `SessionFactory`, or
> - maximize portability to other implementations of JPA by, wherever reasonable, writing code in terms of `EntityManager` and `EntityManagerFactory`, falling back to the native APIs only where necessary.
> 
> Whichever path you take, you will use the JPA-defined mapping annotations most of the time, and the Hibernate-defined annotations for more advanced mapping problems.
> 

Session을 사용할 것인지 EntityManager를 사용해서 JPA를 사용할 것인지 결정해야한다..는 내용 

### SPRING Data JPA는 JPA 위에 올라간 추상화 프레임워크이다.

https://docs.spring.io/spring-data/jpa/reference/

Repository 계층에서 자동 구현한다. 

쿼리 메서드 findByName으로 SQL없이 바로 조회할 수 있다. 

트랜잭션, 엔터티 매니저 관리가 자동화된다. 

## 2.2. 핵심 개념 구조

```
[Spring Data JPA]
   ↑
   │  (자동 구현, Repository, @Transactional)
   │
[JPA]  ← 표준 명세 (javax.persistence.* / jakarta.persistence.*)
   ↑
   │  (interface 정의: EntityManager, Query 등)
   │
[Hibernate] ← 실제 ORM 구현체
   ↑
   │  (JDBC 호출, SQL 생성, 캐시 관리)
   │
[JDBC] ← DB 연결 표준
   ↑
[DBMS] (MySQL, PostgreSQL 등)
```

**📌 JPA는 표준 spec을 제공한다.** 

- `EntityManger`, `@Entity`, `@Id`, `@Table`, `@Query`등의 API 인터페이스만 정의
- 실제 동작은 없고 JPA 만으로는 프로그램을 실행할 수 없다.
- Hibernate(JPA 구현체)가 이 인터페이스를 구현해서 실제 동작을 수행한다.

```java
public interface EntityManager{
	void persist(Object entity);
	<T> T find(Class<T> entityClass, Object primaryKey);
}
```

**📌 Hibernate는 구현체이다.**

- 위 EntityManager 인터페이스를 실제 코드로 구현한 라이브러리
- 내부적으로 JDBC를 사용해서 SQL을 생성하고 DB에 전달한다.
- JPA 명세를 완전히 구현하면서 추가 기능도 제공한다.
    - HQL
    - 2차 캐시
    - Fetch 전략 제어 (EAGER / LAZY)
    - Dirty Checking, Prozy
- Spring boot에서 `spring-boot-starter-data-jpa`를 사용하면 Hibernate가 자동으로 설정된다.

# 3. Entity

> 자바의 객체 클래스로 `@Entity` 어노테이션과 함께 데이터베이스의 테이블과 매핑되어, 각 인스턴스는 하나의 데이터 행을 표현할 수 있다.
> 

## 3.1. 어떤 역할을 하는가?

ORM은 자바 객체와 RDB의 패러다임이 다르다는 간극을 채우기 위해서 등장했다. 

- 자바 객체는 상속, 연관관계 등 객체 지향적 구조를 가진다.
- RDB는 조인관계로 테이블 간 관계를 표현할 수 있다.

그래서 우리는 자바 객체 클래스를 RDB 테이블에 저장될 데이터화하여 애플리케이션 계층과 데이터베이스 계층의 데이터를 전달하고 싶다. 

이를 위해서 자바 객체 클래스 `Entity`의 필드와 메서드를 RDB 테이블의 컬럼으로 표현하고, `Entity`의 인스턴스를 생성해 테이블의 행 데이터를 표현하고 관리한다. 

## 3.2. 어떻게 사용할 수 있는가?

1. 자바의 객체 클래스로 데이터베이스의 컬럼을 표현한다. 
2. 해당 객체 클래스에 `@Entity` 라는 어노테이션을 붙인다. 
3. 이외에 `@Id`로 기본키를 지정하고, `@GeneratedValue`로 자동 생성 전략을 설정한다. 
4. 이 어노테이션이 붙는다면 해당 객체 클래스의 인스턴스는 JPA를 통해 데이터베이스의 테이블에 자동으로 매핑된다. 
5. 해당 인스턴스를 통해서 자바 ↔ RDBMS의 데이터 연결이 가능하다. ⇒ CRUD 연산이 가능하다.

# 4. EntityManager

> JPA에서 엔터티의 생명주기를 관리하는 핵심 인터페이스 
영속성 컨텍스트를 통해 엔터티를 등록, 조회, 수정, 삭제하는 역할
> 

**📌 영속성 (Persistence)** 

> 프로그램이 종료되어도 사라지지 않고 계속 유지되는 성질
> 

데이터베이스의 경우 휘발되는 메모리가 아니라 영속적으로 관리되는 디스크에서 데이터를 관리함으로서 영속성을 보존한다. 

## 4.1. 왜 필요한가

1. ORM은 자바 객체(Entity)를 테이블의 데이터로 자동 변환하는 기술이다. JPA는 ORM의 표준 명세이다.
2. Entity 클래스는 생명주기를 가지며 JPA는 이 생명주기의 변화를 추적한다. 
3. 생명 주기의 변화를 추적하기 위해 Entity 객체를 영속성 컨텍스트라는 메모리 공간에 저장해둔다.
4. EntityManager는 내부적으로 영속성 컨텍스트에서 엔터티의 상태를 관리하고, 개발자는 이 인터페이스를 통해 엔터티를 저장하거나 조회할 수 있다. 

**📌 Entity의 생명주기** 

![image.png](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdna%2F9TmXh%2Fbtq4cf7NDf1%2FAAAAAAAAAAAAAAAAAAAAAKDxaye6FXGVwjrXaB94Y3r_srpqA-WjE5yRScSTvRj5%2Fimg.png%3Fcredential%3DyqXZFxpELC7KVnFOS48ylbz2pIh7yKj8%26expires%3D1761922799%26allow_ip%3D%26allow_referer%3D%26signature%3DH8hOapGzZyMGQkbbJ7b4HWfwZFY%253D)

참고: https://girawhale.tistory.com/122

`New` - 순수한 객체 상태, 영속성 컨텍스트와 무관

`Managed` - EntityManager를 통해 엔터티를 영속성 컨텍스트에 저장하고 관리하는 상태 

`Detached` - 영속성 컨텍스트에 저장되었다가 분리된 상태

`Removed` - 엔터티를 영속성 컨텍스트와 데이터베이스에서 삭제된 상태 

## 4.2. 주요 메서드

`persist(entity)` 

- 엔터티를 영속성 컨텍스트에 등록
- SQL 실행 안함 → 트랜잭션 커밋 시 Insert 발생

`find(EntityClass, id)`

- pk 기반으로 엔터티 조회
- 1차 캐시(영속성 컨텍스트) → DB 순서로 조회

`remove(entity)` 

- 엔터티 삭제
- 트랜잭션 커밋 시 DELETE 발생

`flush()`

- 변경 내용을 DB에 동기화