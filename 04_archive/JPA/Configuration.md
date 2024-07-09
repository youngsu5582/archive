---
tags:
  - JPA
참고_링크: https://docs.spring.io/spring-data/jpa/reference/repositories/create-instances.html
추가_정보: Spring JPA 공식 문서
---
```java
@Configuration
@EnableJpaRepositories
@EnableTransactionManagement
class ApplicationConfig {

  @Bean
  public DataSource dataSource() {

    EmbeddedDatabaseBuilder builder = new EmbeddedDatabaseBuilder();
    return builder.setType(EmbeddedDatabaseType.HSQL).build();
  }

  @Bean
  public LocalContainerEntityManagerFactoryBean entityManagerFactory() {

    HibernateJpaVendorAdapter vendorAdapter = new HibernateJpaVendorAdapter();
    vendorAdapter.setGenerateDdl(true);

    LocalContainerEntityManagerFactoryBean factory = new LocalContainerEntityManagerFactoryBean();
    factory.setJpaVendorAdapter(vendorAdapter);
    factory.setPackagesToScan("com.acme.domain");
    factory.setDataSource(dataSource());
    return factory;
  }

  @Bean
  public PlatformTransactionManager transactionManager(EntityManagerFactory entityManagerFactory) {

    JpaTransactionManager txManager = new JpaTransactionManager();
    txManager.setEntityManagerFactory(entityManagerFactory);
    return txManager;
  }
}
```

이렇게 3가지로 구성
#### DataSource

설정들을 통해 DB 접근해 Connection 얻음
```java
DriverManagerDataSource dataSource = new DriverManagerDataSource(); dataSource.setDriverClassName("com.mysql.cj.jdbc.Driver"); dataSource.setUrl("jdbc:mysql://localhost:3306/your_database_name"); dataSource.setUsername("your_username"); dataSource.setPassword("your_password");
```
InMemory 아닐시에는 이렇게 접근 가능
#### EntityManagerFactoryBean

EntityManager 관리하는 Factory 가 담겨있는 Bean

```java
final AbstractJpaVendorAdapter vendorAdapter = new HibernateJpaVendorAdapter();  
vendorAdapter.setGenerateDdl(true);
```

실제 사용을 하는 vendor 주입
-> 각종 설정 가능!
```java
final LocalContainerEntityManagerFactoryBean factory = new LocalContainerEntityManagerFactoryBean();  
factory.setJpaVendorAdapter(vendorAdapter);  
factory.setPackagesToScan("com.roomescape.dao");  
factory.setDataSource(dataSource());
```

그후 factory 설정
-> vendor 지정
-> scan 할 package 지정
-> 실제 연결하는 DB(DataSource)
#### TransactionManager

트랜잭션을 담당하는 매니저
```java
@Bean  
public PlatformTransactionManager transactionManager(final EntityManagerFactory entityManagerFactory) {  
    final JpaTransactionManager txManager = new JpaTransactionManager();  
    txManager.setEntityManagerFactory(entityManagerFactory);  
    return txManager;  
}
```

EntityManagerFactory 주입