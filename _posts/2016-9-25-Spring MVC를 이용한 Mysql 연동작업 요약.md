---
layout: post
title: Spring MVC를 이용한 Mysql 연동작업 요약
---
<hr>
### 1. Mysql 설치 및 DB정보 설정, 권한 주기

### 2. schema 및 user 테이블 작성
```mysql
create schema nhndnt;
create table user (
	userid varchar(100) not null,
	userpw varchar(50) not null,
	username varchar(50) not null,
	primary key(userid)
);
```

### 3. user 10명에 대한 정보 삽입
```mysql
insert into user values('id001', '001', '홍길동');
insert into user values('id002', '002', '김길동');
insert into user values('id003', '003', '이길동');
insert into user values('id004', '004', '남궁길동');
insert into user values('id005', '005', '조길동');
insert into user values('id006', '006', '한길동');
insert into user values('id007', '007', '박길동');
insert into user values('id008', '008', '최길동');
insert into user values('id009', '009', '제갈길동');
insert into user values('id010', '010', '오길동');
```

### 4. Maven 설정 (http://mavenrepository.com/ 사이트를 이용하면 쉽게 설정 가능)
* pom.xml 파일의 Dependencies탭에 Dependency를 등록하면 자동적으로 해당 jar파일을 다운받아서 Project의 환경을 쉽게 관리할 수 있음
* Overview 탭에서 java, spring framework, aspectj, slf4j의 버전을 변경해줄 수 있음
* 아래는 Spring MVC Project 기준으로 DB연동을 위한 Maven설정 시 필요한 Dependency들을 artifactId 기준으로 나열하였음
- mysql-connector-java : Mysql의 JDBC연결을 위함
- junit : 단위 테스트 프레임워크인 junit을 사용하기 위함, Spring MVC Project를 만들면 기본적으로 Maven 설정에 적용되나 최신버전을 이용하기 위해서는 버전을 바꿔줘야 함
- mybatis : mybatis(java객체와 DB의 Column을 대응시켜 Mapping해줌)를 사용하기 위함
- mybatis-spring : mybatis와 spring을 연결해주는 접착제 역할을 담당함
- spring-jdbc : spring과 jdbc를 연결해주는 접착제 역할을 담당함
- spring-test : spring과 mybatis가 제대로 연동되는지 확인하기 위한 역할을 담당함
- spring-jdbc와 spring-test의 버전은 ${org.springframework-version}를 입력해서 spring framework의 버전과 맞춰야 한다.
* pom.xml 파일을 저장해야 설정이 적용된다.

### 5. root-context.xml 파일의 설정 : spring framework에서 사용할 모듈에 관련된 작업
* src/main/webapp/WEB-INF/spring/root-context.xml 파일 찾기
* Namespaces 탭으로 이동
* aop, beans, context, jdbc, mybatis-spring 체크 후 저장
* Source 탭으로 이동하면 xml namespace가 추가된 것을 확인할 수 있음

### 6. JDBC 연결 설정하기
* DataSource 객체 설정
```xml
<bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
	<property name="driverClassName" value="com.mysql.jdbc.Driver"></property>
	<property name="url" value="jdbc:mysql://127.0.0.1:3306/nhndnt"></property>
	<property name="username" value="root"></property>
	<property name="password" value="1234"></property>
</bean>
```

* DataSource 테스트
```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = { "file:src/main/webapp/WEB-INF/spring/**/*.xml" })
public class DataSourceTest {

	@Autowired
	private DataSource ds;

	@Test
	public void dataSourceTest() throws SQLException {
		assertNotNull(ds);
		try (Connection con = ds.getConnection()) {
			assertNotNull(con);
		} catch (Exception e) {
			e.printStackTrace();
		}
	}
}
```

### 7. mybatis 연결 설정하기 (http://www.mybatis.org/mybatis-3/ko/ 사이트 참조)
* SqlSessionFactory 객체 설정
```xml
<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
	<property name="dataSource" ref="dataSource"></property>
</bean>
```

* SqlSessionFactory, SqlSession 테스트
```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = { "file:src/main/webapp/WEB-INF/spring/**/*.xml" })
public class MyBatisTest {
	
	@Autowired
	SqlSessionFactory sqlSessionFactory;

	@Test
	public void sqlSessionFactoryTest() {
		assertNotNull(sqlSessionFactory);
	}

	@Test
	public void sqlSessionTest() {
		try (SqlSession sqlSession = sqlSessionFactory.openSession()) {
			assertNotNull(sqlSession);
		} catch (Exception e) {
			e.printStackTrace();
		}
	}
}
```
