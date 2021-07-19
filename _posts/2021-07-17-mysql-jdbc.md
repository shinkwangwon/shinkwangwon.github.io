---
layout: post
title: "MySQL 프로그램 연동"
tags: [mysql]
comments: true
date: 2021-07-17
---

# 13. 프로그램 연동

## 13.1 자바

자바 언어로 MySQL에 접속하려면 자바에서 제공하는 표준 데이터베이스 접속 API인 JDBC 이용

JDBC는 인터페이스일 뿐이며, 실제 각 DBMS에 접속해 필요한 작업을 하는 구현체(implementation)는 각 DBMS 제조사에서 제공하는 JDBC 드라이버임

각 DBMS 제조사가 자바에서 지정한 표준에 맞게 구현체를 구현하므로 실제 JDBC를 이용하는 개발자는 DBMS 벤더에 신경쓰지 않고 DB 프로그램을 개발할 수 있는 것

### 13.1.1 JDBC 버전

- MySQL의 JDBC 드라이버 이름은 Connector/J
- Connector/J 는 JDBC 표준과는 무관하게 자체적인 버전 체계를 가지고 있음.
- Connector/J 버전만 보고 JDBC의 버전을 알기 어렵기 때문에 JDBC의 특정 기능을 사용하기 위해 아래처럼 해당 Connector/J의 버전이 JDBC의 어떤 버전까지 지원하는지 확인하는게 좋음
- Connector/J 의 버전별로 자바 개발 및 실행하는데 필요한 버전이 있음
- [https://dev.mysql.com/doc/connector-j/5.1/en/connector-j-versions.html](https://dev.mysql.com/doc/connector-j/5.1/en/connector-j-versions.html)

![No image](/assets/posts/20210717/Untitled.png)

### 13.1.2 MySQL Connector/J 를 이용한 개발

**MySQL 서버 접속**

- JDBC URL 을 통해 MySQL 서버에 접속. 여기서 URL은 일반적인 HTTP, FTP에 사용하는 URL이 아니라, 접속할 MySQL 서버의 정보를 표준 포맷으로 조합한 문자열 (Connection String 이라고도 함)
- 요즘은 거의 자바의 Connection Pool을 사용하기 때문에 DB에 접속하는 코드를 이렇게 직접 작성하는 경우는 거의 없음

```java
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;

public class JDBCTester {

    public static void main(String[] args) {
        Connection conn = null;

        try {
            conn = (new JDBCTester()).getConnection();
            System.out.println("Connection is ready");

            conn.close();
        } catch (Exception e) {
            System.out.println("Can't open connection, because " + e.getMessage());
        }
    }

    public Connection getConnection() throws Exception {
				// MySQL JDBC driver class 
        String driver = "com.mysql.jdbc.Driver";
				// JDBC URL 
        String url = "jdbc:mysql://localhost:3306/test_db?characterEncoding=UTF-8&serverTimezone=Asia/Seoul";
        String uid = "userid";
        String pwd = "password";
        Class.forName(driver).newInstance();
				// Class.forName(driver).getDeclaredConstructor().newInstance(); // java 9 이상 
        Connection conn = DriverManager.getConnection(url, uid, pwd);

        return conn;
    }
}
```



**SELECT 실행**

Statement 객체

- JDBC를 사용하는 애플리케이션에서 SELECT 뿐만 아니라 모든 SQL과 DDL 문장을 실행하는데 필요한 객체
- executeQuery() : 결과셋(ResultSet)을 반환하는 SELECT 쿼리문장 사용시
- executeUpdate() : 결과셋(ResultSet)을 반환하지 않는 INSERT, UPDATE, DELETE, DDL 문장 사용시. (주로 변경된 건수를 반환하지만, 반환값은 SQL 종류 및 버전에 따라 조금씩 다름)
- execute() : 실행하려는 쿼리가 SELECT인지 INSERT인지 모를 때 사용 가능

ResultSet 객체

- Select 결과를 담기위한 객체
- next() : 결과셋에 아직 읽지 않은 레코드가 있는지 확인하는 메소드
- ResultSet의 결과를 getString(), getInt() 등의 함수로 칼럼 값 가져올 수 있음
- getString("칼럼명 or Alias명"), getString(칼럼의 순번) 방식으로 사용. 순번은 1부터 시작

```java
public static void main(String[] args) {
      Connection conn = null;
      Statement stmt = null;
      ResultSet rs = null;

      try {
          conn = (new JDBCTester()).getConnection();
          stmt = conn.createStatement();
          rs = stmt.executeQuery("SELECT * FROM employees LIMIT 2");

          while (rs.next()) {
              System.out.println("[" + rs.getString(1) + "][" + rs.getString("first_name") + "]");
          }

      } catch (Exception e) {
          System.out.println("Can't open connection, because " + e.getMessage());
      } finally {
          try { if(rs != null) { rs.close(); } } catch (Exception e) {}
          try { if(stmt != null) { stmt.close(); } } catch (Exception e) {}
          try { if(conn != null) { conn.close(); } } catch (Exception e) {}
      }
  }

>> result
[10001][Georgi]
[10002][Bezalel]
```

**INSERT/UPDATE/DELETE 실행**

- 별도의 결과 셋(ResultSet)을 반환하지 않으므로 executeUpdate() 함수를 사용
- executeUpdate() 함수는 쿼리에 의해 변경된 레코드 건수를 반환
    - SQL 종류에 따라 WHERE 조건에 일치하는 레코드 건수를 반환하는 경우도 있음 (MySQL은 변경된 레코드 건수 반환)
- 매 쿼리가 실행될때마다 자동으로 트랜잭션이 commit 되기 때문에, 하나의 트랜잭션으로 여러개의 UPDATE, DELETE 문장을 묶어서 실행하려면 AutoCommit 모드를 FALSE로 설정해야 함

```java
public static void main(String[] args) {
    Connection conn = null;
    Statement stmt = null;
    int affectedRowCount = 0;

    try {
        conn = (new JDBCTester()).getConnection();
        conn.setAutoCommit(false);  // auto commit false
        stmt = conn.createStatement();
        affectedRowCount = stmt.executeUpdate("UPDATE employees SET first_name='Lee' WHERE emp_no=10001");

        if(affectedRowCount == 1) {
            System.out.println("사원명이 변경되었습니다.");
            conn.commit();
        } else  {
            System.out.println("사원을 찾을 수 없습니다.");
            conn.rollback();
        }

    } catch (Exception e) {
        System.out.println("Error!! because " + e.getMessage());
    } finally {
        try { if(stmt != null) { stmt.close(); } } catch (Exception e) {}
        try { if(conn != null) { conn.close(); } } catch (Exception e) {}
    }
}
```

**Statement 와 PreparedStatement의 차이**

쿼리 실행 단계

- 쿼리분석 → 최적화 → 권한체크 → 쿼리실행

Statement의 경우 위의 모든 단계를 매번 거쳐서 쿼리가 실행되는데, `쿼리분석`이나 `최적화`와 같은 작업은 상대적으로 시간이 오래 걸리는 작업임

PreparedStatement의 경우 `쿼리분석`이나 `최적화`의 일부 작업을 처음 한번만 수행해 별도로 MySQL 서버 메모리에 저장해 두고, 다음 요청 쿼리부터는 저장된 분석 결과를 재사용 함 

아래 예시를 보면. PreparedStatement를 사용하고 똑같은 패턴의 쿼리임에도, 위와 같이 상수값을 그대로 사용하면 100개의 쿼리 분석 결과 보관 필요 (메모리 낭비)

```sql
-- WHERE 절의 상수값만 다른 100 개의 쿼리
SELECT * FROM employees WHERE emp_no=10001;
SELECT * FROM employees WHERE emp_no=10002;
SELECT * FROM employees WHERE emp_no=10003;
...
```

PreparedStatement에서 `?` 로 표현하는 바인딩 변수 또는 변수 홀더(Variable holder)라는 기능 제공

바인딩 변수를 사용하면 쿼리를 최대한 템플릿화 할 수 있고, 템플릿화된 문장만 MySQL 서버 메모리에 보관하면 되므로 메모리 낭비를 줄일 수 있음

```sql
SELECT * FROM employees WHERE emp_no=?;
```

예제

```java
public static void main(String[] args) {
    Connection conn = null;
    PreparedStatement pstmt = null;
    ResultSet rs = null;

    try {
        conn = (new JDBCTester()).getConnection();
        pstmt = conn.prepareStatement("SELECT * FROM employees WHERE emp_no=?"); // (1)

        pstmt.setInt(1, 10001);
        rs = pstmt.executeQuery();
        while(rs.next()) System.out.println("First name : " + rs.getString("first_name"));
        rs.close();

        pstmt.setInt(1, 10002); // (2)
        rs = pstmt.executeQuery();
        while(rs.next()) System.out.println("First name : " + rs.getString("first_name"));
        rs.close();
        rs = null;

    } catch (Exception e) {
        System.out.println("Error!! because " + e.getMessage());
    } finally {
        try { if(rs != null) { rs.close(); } } catch (Exception e) {}
        try { if(pstmt != null) { pstmt.close(); } } catch (Exception e) {}
        try { if(conn != null) { conn.close(); } } catch (Exception e) {}
    }
}
```

(1) conn.prepareStatement("SELECT ...") 함수를 호출하면 Connector/J 는 SQL문장을 MySQL 서버로 전송해서 쿼리를 분석하고 그 결과를 저장해 둠. 

그리고 MySQL 서버는 쿼리분석결과의 포인터와 같은 해시 값을 Connector/J 로 반환 

반환받은 해시 값을 이용해 PreparedStatement 객체 생성 후 바인딩 변수의 값만 변경하며 재사용

(2) pstmt.setInt(1, 10002) 두번째 호출. MySQL 서버는 이미 이 쿼리 패턴에 대한 분석정보를 가지고 있으므로 매번 쿼리를 분석하지 않고 단축된 경로로 쿼리를 실행하기 때문에 Statement보다 빠르게 처리됨 

![No image](/assets/posts/20210717/Untitled3.png)

PreparedStatement 장점

- 한 번 실행된 쿼리는 매번 쿼리분석 과정을 거치지 않고 처음 분석된 정보를 재사용
- 바이너리 프로토콜 사용(MySQL 5.0 이상). Statement는 문자열 기반의 프로토콜을 사용하기 때문에 불필요한 타입 변환이 필요. 성능상 PreparedStatement가 좋음
- SQL Injection 방어. PreparedStatement는 사용자 입력값에 홑따옴표나 쌍따옴표, 역슬래시 등과 같은 문자에 대한 이스케이프 처리를 MySQL Connector/J 에서 대신 처리해 줌

```java
// Statement 문제
String query = "SELECT * FROM user WHERE user_id='" + userId + "' AND passwd='" + passwd + "'";

// 사용자 입력 "admin' --"
// SELECT * FROM user WHERE user_id='admin' --' AND passwd='xx';
// 만약 ID가 admin이면 쿼리 통과 됨
```

**Prepare Statement 종류**

Client Prepare Statement

- PreparedStatement 객체를 이용해 변수가 포함된 SQL 문장을 실행할 때, Connector/J 가 자체적으로 SQL문장의 바인딩 변수에 값을 매핑해 하나의 완성된 SQL문장으로 만들어 서버에 전송하는 방식
- 이 방식을 사용하면 개발자는 Prepare Statement를 사용한다고 느끼지만 실제 MySQL 서버는 매번 쿼리 문장을 분석하고 실행계획을 수립해서 쿼리 실행

Server Prepare Statement

- 일반적으로 Prepare Statement라고 하면 Server Prepare Statement 를 뜻함
- 매번 쿼리를 실행할 때마다 클라이언트는 SQL 문장에 바인딩할 변수 값만 전송하고, MySQL 서버는 저장된 쿼리의 분석 정보에 변수 값을 바인딩해서 쿼리를 실행

Why ?

- JDBC 표준에 PrepareStatement 기능이 도입됐을 때, MySQL 서버에는 이걸 처리할만한 기능이 없었음.
- Server Prepare Statement의 기능인것 처럼 모방하기위해 먼저 나온 것이 Client Prepare Statement.
- MySQL 5.0 버전까지는 Server Prepare Statement는 쿼리캐시는 사용 불가능했으니 5.1 버전 이상부터 쿼리캐시 사용 가능
- Server Prepare Statement 를 사용하는 것을 권장
- JDBC URL에 userServerPrepStmts=true 옵션 설정해야 Server Prepare Statement 로 동작
- Spring Boot 2.0 이상에선 HikariDataSource를 사용하기 때문에 hikariConfig 설정해야 함

```yaml
spring:
  datasource:
    hikari:
      data-source-properties:
        useServerPrepStmts: true
```

```java
@Configuration
@EnableTransactionManagement
class DataSourceConfig(
        @Value("\${datasource.api-db.url}") private val dbUrl: String,
        @Value("\${datasource.api-db.username}") private val dbUserName: String,
        @Value("\${datasource.api-db.password}") private val dbPassword: String,
) {
    @Bean
    @ConfigurationProperties(prefix = "spring.datasource.hikari")
    fun hikariConfig(): HikariConfig {
        return HikariConfig()
    }

    @Bean
    fun dataSourceConfigApi(): DataSource {
        return HikariDataSource(HikariConfig().apply {
            jdbcUrl = dbUrl
            username = dbUserName
            password = dbPassword
        })
    }

		@Bean
    fun entityManagerFactory(
            builder: EntityManagerFactoryBuilder,
            beanFactory: ConfigurableListableBeanFactory
    ): LocalContainerEntityManagerFactoryBean {
        return builder.dataSource(dataSourceConfigApi())
                .packages("com.test.batch")
                .properties(DbInfo.jpaProperties(beanFactory))
                .build()
    }
}
```

**스토어드 프로시저 실행**

- skip

**배치 처리**

`들어가기 전에, JPA Entity의 PK 채번 방식을 GenerationType.IDENTITY 로 설정할 경우 Batch Insert 비활성화 한다고 함. INSERT를 수행해야 ID값을 확인할 수 있기 때문`

```java
public static void main(String[] args) {
    Connection conn = null;
    PreparedStatement pstmt = null;
    List empList = null;  // 임의의 empList 가 있다고 가정

    try {
        // (1)
        conn = DriverManager.getConnection("jdbc:mysql://localhost:3306/test_db" +
                "?rewriteBatchedStatements=true&useServerPrepStmts=false", "codedraw", "codedraw");
        
        pstmt = conn.prepareStatement("INSERT INTO employees VALUES (?, ?, ?)");
        for(int idx = 0; idx < empList.size(); idx++) {
            Map empMap = (Map)empList.get(idx);
            pstmt.setString(2, (String) empMap.get("first_name"));
            pstmt.setString(3, (String) empMap.get("last_name"));

            pstmt.addBatch();  // (2)
        }

        pstmt.executeBatch();  // (3)

    } catch (Exception e) {
        System.out.println("Error!! because " + e.getMessage());
    } finally {
        try { if(pstmt != null) { pstmt.close(); } } catch (Exception e) {}
        try { if(conn != null) { conn.close(); } } catch (Exception e) {}
    }
}
```

(1). rewriteBatchedStatements=true로 설정하면, MySQL Connector/J가 addBatch() 함수로 누적된 레코드를 모아 다음과 같은 형태의 구문으로 실행.
rewriteBatchedStatements를 활성화하면 useServerPrepStmts는 비활성해야함. 

```java
INSERT INTO employees VALUES
	(1000000, 'Brandon', 'Lee'),
	(1000001, 'Brandon', 'Kim'),
	(1000002, 'Brandon', 'Choe');
```

(2). addBatch() 메소드는 실제 쿼리를 수행하기 전에 레코드를 누적시키는 함수. 애플리케이션 서버 및 MySQL 서버를 고려하여 적당한 크기만큼만 누적시키도록 해야함

(3). executeBatch() 메소드는,  addBatch() 메소드로 누적된 레코드를 한번에 MySQL 서버로 전달하여 쿼리 실행.

**트랜잭션**

MySQL Connector/J 는 아무설정을 하지 않으면 기본적으로 AutoCommit 모드에서 트랜잭션 처리

이 경우 InnoDB 에서 쿼리 하나하나에 대해서는 트랜잭션이 보장되지만, 연속해서 실행되는 쿼리의 묶음에 대해서는 트랜잭션 보장되지 않음 

```java
public void transfer(String fromId, String toId, int howMuch) throws SQLException {
    Connection conn = null;
    Statement stmt = null;
    int affectedRowCount = 0;

    try {
        conn = (new JDBCTester()).getConnection();
        stmt = conn.createStatement();

        conn.setAutoCommit(false);
        affectedRowCount = stmt.executeUpdate("UPDATE user_point SET point=point-"+howMuch+" WHERE user_id='"+fromId+"'");
        affectedRowCount = stmt.executeUpdate("UPDATE user_point SET point=point-"+howMuch+" WHERE user_id='"+toId+"'");

        conn.commit();

    } catch (Exception e) {
        conn.rollback();
    } finally {
        try { if(stmt != null) { stmt.close(); } } catch (Exception e) {}
        try { if(conn != null) { conn.close(); } } catch (Exception e) {}
    }
}
```

AutoCommit = FALSE 인 상태에서의 트랜잭션

- AutoCommit = FALSE 상태에서는 어떤 쿼리를 실행하든 트랜잭션이 바로 시작되고 고유한 트랜잭션 번호 발급됨
- INSERT, UPDATE, DELETE 같은 문장은 트랜잭션으로 commit 또는 rollback을 해야 데이터가 변경되기 때문에 잊지 않고 실행하게 됨
- SELECT의 경우에는 commit or rollback 해도 데이터가 변경되지 않기 때문에 commit or rollback 없이 그냥 커넥션을 반납하는 형태로 쓰이기도 하지만,  잊지 않고 commit or rollback 을 수행해주어야 함
- MySQL은 트랜잭션 격리 수준이 REPEATABLE-READ이기 때문에, 특정 트랜잭션이 유효한 동안에는 해당 트랜잭션이 처음 시작했던 시점의 데이터를 동일하게 보여줘야 함
- 이를 위해 InnoDB 스토리지 엔진은 다른 트랜잭션에서 그 데이터를 변경했다 하더라도 변경 전 데이터를 계속 Undo 영역에 보관하고 있어야 함.
- 따라서 SELECT만 사용해도 트랜잭션을 끝내지 않으면 불필요한 저장을 하고 있어야하기 때문에 자원소모가 많이 발생하게 됨
- connection.close() 에서 rollback 을 처리하긴 하지만, 메소드가 끝날때까지 계속 트랜잭션을 물고 있어야 하는게 문제

AutoCommit = TRUE인 상태에서의 트랜잭션

- 쿼리 하나가 수행될때마다 자동으로 commit 수행됨
- MySQL의 `BEGIN` 또는 `START TRANSACTION` 명령을 execute() 또는 executeUpdate() 함수로 실행하면 명시적으로 트랜잭션을 시작할 수 있음

```java
try {
    conn.setAutoCommit(true);
    stmt.execute("BEGIN");
    affectedRowCount = stmt.executeUpdate("UPDATE user_point SET point=point-"+howMuch+" WHERE user_id='"+fromId+"'");
    affectedRowCount = stmt.executeUpdate("UPDATE user_point SET point=point-"+howMuch+" WHERE user_id='"+toId+"'");

    conn.commit();
catch(Exception e) {
    conn.rollback();
}
```

**대량 데이터 가져오기**

- JDBC 표준에서 제공하는 Statement.setFetchSize() 라는 함수는 SELECT된 레코드를 애플리케이션으로 가져올 때 한번에 가져올 건수를 설정하는 것임
- 하지만! MySQL의 Connector/J 에서는 위 함수를 지원하지 않음
- 대량의 데이터를 한번에 가져오려면 시간도 오래걸릴 뿐더러 Out Of Memory 발생할 수 있음

ResultSet Streaming

- 쿼리의 결과를 한번에 받아오지 않고, MySQL 서버에서 한건 단위로 읽어서 가져오는 방법
- 건 단위로 통신해야하기 때문에 매우 느림

Server Cursor

- MySQL 서버에 결과셋에 상응하는 크기의 임시테이블을 만들어두고, defaultFetchSize에 설정한 레코드 건수 만큼씩 Connector/J 의 캐시 메모리 영역에 내려받아 애플리케이션에게 제공
- 너무 대용량일 경우 MySQL 서버에 임시테이블을 위한 무리가 발생할 수 있음

⇒ 한번에 모든 데이터를 다 올리는건 솔직히 무리. 비즈니스적으로 풀어서 페이징으로 처리하는게 답일 듯(?)

**복제 MySQL을 위한 ReplicationDriver**

ReplicationDriver는 데이터를 변경하는 쓰기 쿼리는 마스터 MySQL에서 실행하고, 읽기 쿼리는 슬레이브 MySQL에서 실행하게 함

`com.mysql.jdbc.Driver` 대신 `com.mysql.jdbc.ReplicationDriver` 사용

쿼리 실행하기 직전에 Connection.setReadOnly(false)를 실행하면 마스터 서버로, setReadOnly(true)로 설정하면 슬레이브 서버로 쿼리를 전송

**ReplicationDriver 문제점**

- ReplicationDriver를 사용하면 ReplicationConnection 객체가 생성되는데, 이는 항상 마스터에 대한 커넥션 1개와 슬레이브에 대한 커넥션 1개를 동시에 가짐
- 어느 한 시점을 기준으로 둘중 하나만 사용할 수 있기 때문에 항상 하나의 커넥션은 불필요하게 낭비됨
- 커넥션 풀을 사용할 때는 커넥션의 타임아웃 방지나 유효성 체크를 위해 SELECT 1 과 같은 ValidationQuery 를 설정할 수 있는데, 마스터/슬레이브중 사용중인 한쪽만 유효성 체크를 수행하기 때문에 나머지 한쪽 커넥션은 타임아웃으로 인해 연결해지될 수 있음
