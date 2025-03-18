# Bean vs component
@Component is used at class level whereas @Bean is used at method level. 
@Bean can only be used with class which has @Configuration. 
@Bean explicitly declares a bean rather than letting Spring handling automatically.
Creating database connection in app . Then we will use @Bean
1. Define a datasource
2. then define entitymanager

```java
@Configuration
@EnableTransactionManagement
@EnableJpaRepositories(
        basePackages = "com.example.repository.mysql",
        entityManagerFactoryRef = "mysqlEntityManagerFactory",
        transactionManagerRef = "mysqlTransactionManager"
)
public class MySQLDatabaseConfig {

    @Primary //is used if we have multiple databases
    @Bean(name = "mysqlDataSource")
    @ConfigurationProperties(prefix = "spring.datasource.mysql")
    public DataSource dataSource() {
        return DataSourceBuilder.create().build();
    }

    @Primary
    @Bean(name = "mysqlEntityManagerFactory")
    public LocalContainerEntityManagerFactoryBean entityManagerFactory(
            EntityManagerFactoryBuilder builder,
            @Qualifier("mysqlDataSource") DataSource dataSource) {
        return builder
                .dataSource(dataSource)
                .packages("com.example.entity") // Entities package
                .persistenceUnit("mysqlPU")
                .build();
    }

    @Primary
    @Bean(name = "mysqlTransactionManager") //used if we have multiple transaction then @Transactional("mysqlTransactionManager")
    public PlatformTransactionManager transactionManager(
            @Qualifier("mysqlEntityManagerFactory") EntityManagerFactory entityManagerFactory) {
        return new JpaTransactionManager(entityManagerFactory);
    }
}

```

# Creating bean without dependency injection
```java
@SpringBootApplication
class MyApp{
  public static void main(String[] args) {
    ConfigurableApplicationContext context = SpringApplication.run(MyApp.class, args);

    MyBean myBean1 = new MyBean();//not managed by SPring
    MyBean mybean2 = context.getBean(MyBean.class); //part of DI and managed by Spring
    
  }
}

```
