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

### Chapter 5 - The Forklifted Application

This chapter is about moving an existing application to the cloud.  It is not about building applications that are native to the cloud (that's the focus of all the other chapters).

Cloud Foundry doesn't care what kind of application it's running.  It cares about Linux containers, which are operating system processes.  

> Buildpacks tell Cloud Foundry what to do given a Java `.jar`, a Rails application, a Java `.war`, a Node.js application, etc., and how to turn it into a container that the platform can treat like a process.  A buildpack is a set of *callbacks* -- shell scripts that respond to well-known calls -- that the runtime will use to ultimately create a Linux container to be run. This process is called *staging*. 

You can use a custom build pack  with an argument to `cf push`:

```
cf push -b https://github.com/a/custom-buildpack.git#my-branch custom-app
```

> Alternatively, you can specify the buildpack in the `manifest.yml` file that accompanies your application.

IBM maintains a Websphere Liberty buildpack for applications historically deployed using WebSphere.

Cloud Foundry supports runing containerized (Docker) applications.  However, it is not recommended due to security, extra work for DevOps, and they are harder to patch/update.

> Cloud Foundry applications consume backing services by looking for their locators and credentials in an environment variable called `VCAP_SERVICES`. The simplicity of this approach is a feature: any language can pluck the environment variable out of the environment and parse the embedded JSON to extract things like service hosts, ports, and credentials.

Cloud Foundry is HTTP-first.  If you need to do RPC with RMI/EJB, you will need to tunnel it through HTTP.

Spring Session can be used for `HttpSession` and makes it easy to write session data to Redis, Hazelcast, etc.
 
Cloud Foundry doesn't provide a durable filesystem, but you can use FUSE-based filesystems like SSHFS on Cloud Foundry.  This would let you mount a remote filesystem using SSH.  If you don't care about using `java.io.File`, you can use an alternative like MongoDB GridFS.

> Cloud Foundry terminates HTTPS requests at the highly available proxy that guards all applications.  Any call that you route to your application will respond to HTTPS as well.

The Spring Boot Actuator provides a `/health` endpoint to give basic application health information.

### Chapter 6 - Rest APIs

You can handle the `OPTIONS` HTTP verb like this:

```
@RequestMapping(method = RequestMethod.OPTIONS)
ResponseEntity<?> options() {
	return ResponseEntity
	.ok()
	.allow(HttpMethod.GET, HttpMethod.POST,
	HttpMethod.HEAD, HttpMethod.OPTIONS,
	HttpMethod.PUT, HttpMethod.DELETE)
	.build();
}
```

Use `@GetMapping`, `@PostMapping`, etc., as an alternative to `@RequestMapping`.

Building and returning a URI from a `@PostMapping`:

```
URI uri = MvcUriComponentsBuilder.fromController(getClass()).path("/{id}")
.buildAndExpand(customer.getId()).toUri();
return ResponseEntity.created(uri).body(customer);
```

> When Spring MVC sees a return value, it uses a strategy object, `HttpMessageConverter`, to convert the object into a representation suitable for the client. The client specifies what kind of representation it expects using content-negotiation. By default, Spring MVC looks at the incoming request’s Accept header for a media type, like application/json, application/xml, etc., that a
configured `HttpMessageConverter` is capable of creating, given the object and the desired media type. This same process happens in reverse for data sent from the client to the service and passed to the handler methods as `@RequestBody` parameters

Use `@ControllerAdvice` and `@ExceptionHandler` for handling errors thrown out of your controllers.

HAL, or Hypertext Application Language, is a specialization of JSON with a content type of `application/hal+json` that is an encoding standard for describing link elements in JSON.  Spring Boot supports a HAL client called the HAL Browser, which can be added to a Spring Boot app through Spring Initializr (plus the actuator).

> Postel’s law, also known as the *The Robustness Principle*, says that service implementations should be conservative in what they produce, but liberal in what they accept from others. If a service only needs a subset of a given message payload to do its work, it should not protest if there is extra information in the message for which it may not immediately have any use.

The author recommends a `MAJOR.MINOR.PATCH versioning scheme, where:

> The MAJOR version should change only when an API has breaking changes from its previous version. The MINOR version should change when an API has evolved, but existing clients should continue to work with it. A PATCH version change signals bug fixes to existing functionality.





