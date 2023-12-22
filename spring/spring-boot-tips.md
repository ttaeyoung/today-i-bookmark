# Spring boot Tips!!

## DataSource 설정하기

기존에 1개의 DataSource는 spring.datasource 에 url, username, password 등을 이용하여 설정했다
```yml
spring:
  datasource:
    url: jdbc:mariadb://localhost:3306/{database-name}
    username: root
    password: {password}
    driver-class-name: org.mariadb.jdbc.Driver
```
별도의 Java Config 로 설정하지 않아도 Datasource가 정상적으로 적용된 것을 확인할 수 있다

DB를 2개 이상 사용해야 할 경우, Java Config 를 이용해서 Datasource를 직접 만들어야 한다
```java
@Primary
    @Bean(name = "dataSource")
    @ConfigurationProperties(prefix = "spring.datasource")
    public DataSource dataSource() {
        return DataSourceBuilder.create()
                .type(HikariDataSource.class)
                .build();
    }
```
이렇게 설정하고 실행하는 경우, 아래와 같은 오류가 발생한다
```
Caused by: java.lang.IllegalArgumentException: jdbcUrl is required with driverClassName.
	at com.zaxxer.hikari.HikariConfig.validate(HikariConfig.java:1022) ~[HikariCP-5.0.1.jar:na]
	at com.zaxxer.hikari.HikariDataSource.getConnection(HikariDataSource.java:109) ~[HikariCP-5.0.1.jar:na]
	at org.springframework.jdbc.datasource.DataSourceUtils.fetchConnection(DataSourceUtils.java:160) ~[spring-jdbc-6.1.2.jar:6.1.2]
```

이유는 간단하다 HikariCP의 Database URL 설정은 url 이 아니라 jdbcUrl을 사용하기 때문이다 (com.zaxxer.hikari.HikariConfig 참고)
> 자동으로 생성할 경우 문제가 되지 않지만 Java Config 를 이용한 수동 설정인 경우만 발생한다. 

url 을 jdbc-url 로 변경하면 해결가능하다.

### 추가사항
SpringBoot 에서는 datasource.url 을 자동설정/수동설정에 따라 변경하는 건 불편하고 오해가 생길 수 있으므로 HikariCP 의 설정을 추가했다
```yml
spring:
  profiles: version1
  datasource:
    hikari:
        jdbc-url: jdbc:mariadb://localhost:3306/{database-name}
        username: root
        password: {password}
        driver-class-name: org.mariadb.jdbc.Driver
```

### Reference
* [Spring Boot & HikariCP Datasource 연동하기](https://jojoldu.tistory.com/296)