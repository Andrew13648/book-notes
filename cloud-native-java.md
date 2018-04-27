### Chapter 1

> The greater the number of shared resources, whether application servers or databases, the less control teams had when delivering features into production. (25)

> The key is to understand that agile is a value, not a destination (33)

Microservies are small, independent services built around specific business capabilities.

Spring Initializr can be used to build up a new Spring Boot application.

### Chapter 4

`@SpringBootTest` indicates that a class is a Spring Boot test and scans for a `ContextConfiguration` to load the `ApplicationContext`.

`@RunWith(SpringRunner.class` is an alternative to the old `@RunWith(SpringJUnit4ClassRunner`.

> Integration tests require a Spring context to test the integration between different components (142)

> When writing integration tests in Spring Boot, it becomes possible to mock only selected components in the `ApplicationContext` of a test class. This helpful feature allows developers to test integrations between collaborating components while still being able to mock objects at the boundary of the application -- the difference between a unit test and an integration test still being whether or not a Spring context is used at all. This distinction is important to remember and will aid communication between team members who are writing both unit tests and integration tests. (144)

`@MockBean` mocks a bean in the application context and mutes the original bean.  Internally, it uses Mockito.

There is a `BDDMockito` class that supports BDD style mock objects.

The `webEnvironment` attribute of `@SpringBootTest` describes how Spring Boot should configure the embedded servlet container that your application uses at runtime (mock servlet environment, real servlet environment, or no servlet environment).

If you want to compare an object to a JSON equivalent stored in a file, you can use a `JacksonTester<MyClass>` and read a JSON file from `src/test/resources` for the expected value.

Use `@WebMvcTest` and `@MockMvc` to test controllers by request mappings instead of directly invoking the methods on the controller.

 
