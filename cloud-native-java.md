### Chapter 1

> The greater the number of shared resources, whether application servers or databases, the less control teams had when delivering features into production. (25)

> The key is to understand that agile is a value, not a destination (33)

Microservices are small, independent services built around specific business capabilities.

> One of the main ideas behind building microservices is the ability to have feature teams organize themselves and applications around specific business capabilities. (38)

### Chapter 2

Spring Initializr can be used to build up a new Spring Boot application.

> The Spring framework `Assert` class supports design-by-contract behavior, not unit testing!

### Chapter 3 - Twelve-Factor Application Style Configuration

Configuration in Spring describes how you want to wire beans together.  For XML, use `ClassPathXmlApplicationContext`.  For annotations, use `AnnotationConfigApplicationContext`.

In a Twelve-Factor application, configuration refers to values that may change from one environment to another.

You can configure a class to load values from a properties file by annotating it with `@PropertySource`.  For example, `@PropertySource("classpath:some.properties")` will set `@Value` values from the `some.properties` file.  You will also need to have an instance of `PropertySourcesPlaceholderConfigurer` created.  For example:

```
@Configuration
@PropertySource("classpath:some.properties")
public class AppConfigMongoDB {
	
	@Value("${my.value}")
	private String myValue;
	
	@Bean
	static PropertySourcesPlaceholderConfigurer pspc() {
		return new PropertySourcesPlaceholderConfigurer();
	}
}
```

Environment properties can be extracted from the `Environment` class.  

Spring Boot will automatically read configuration files based on the profile name, like `src/main/resources/application-foo.properties` where `foo` is the current profile.

Spring Boot will also automatically load YAML files if SnakeYAML is on the classpath.

Spring Boot makes both `-D` arguments to the `java` process and environment variables available as properties.

You can map configuration values straight to POJOs by using `@EnableConfigurationProperties` on your Spring application along with `@ConfigurationProperties` on your POJO.

You can use a Spring Cloud Config Server to manage your application configuration externally from your app through a REST API.  With Cloud Foundry, there is a Config Server service in the service catalog.

You can combine this config server with the `@RefreshScope` annotation to make the bean refresh whenever a refresh event is triggered (`RefreshScopeRefreshedEvent`).  

> You can trigger the refresh by sending an empty `POST` request to `http://127.0.0.1:8080/refresh`, which is a Spring Boot Actuator endpoint that is exposed automatically.

This can be done via curl: `curl -d{} http://127.0.0.1:8080/refresh`.

### Chapter 4

`@SpringBootTest` indicates that a class is a Spring Boot test and scans for a `ContextConfiguration` to load the `ApplicationContext`.

`@RunWith(SpringRunner.class` is an alternative to the old `@RunWith(SpringJUnit4ClassRunner`.

> Integration tests require a Spring context to test the integration between different components (142)

> When writing integration tests in Spring Boot, it becomes possible to mock only selected components in the `ApplicationContext` of a test class. This helpful feature allows developers to test integrations between collaborating components while still being able to mock objects at the boundary of the application -- the difference between a unit test and an integration test still being whether or not a Spring context is used at all. This distinction is important to remember and will aid communication between team members who are writing both unit tests and integration tests. (144)

`@MockBean` mocks a bean in the application context and mutes the original bean.  Internally, it uses Mockito.

There is a `BDDMockito` class that supports BDD style mock objects.

The `webEnvironment` attribute of `@SpringBootTest` describes how Spring Boot should configure the embedded servlet container that your application uses at runtime (mock servlet environment, real servlet environment, or no servlet environment).

Spring Boot lets you test different slices of your application using specific annotations:

* `@JsonTest` to test JSON serialization and deserialization.  If you want to compare an object to a JSON equivalent stored in a file, you can use a `JacksonTester<MyClass>` and read a JSON file from `src/test/resources` for the expected value.
* `@WebMvcTest` in conjunction with `@MockMvc` to test controllers by request mappings instead of directly invoking the methods on the controller.
* `@DataJpaTest` to test Spring Data JPA repositories.
* `@RestClientTest` with `@MockRestServiceServer` to test classes that use `RestTemplate`.




 
