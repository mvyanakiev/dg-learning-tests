[Integration tests for MVC - Source](https://www.baeldung.com/integration-testing-in-spring)

# Base setup

## Dependencies

Това са необходимите базови депендънсита:

```XML
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter-engine</artifactId>
    <version>5.8.1</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter-api</artifactId>
    <version>5.8.1</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-test</artifactId>
    <version>5.3.3</version>
    <scope>test</scope>
</dependency>
```


За да assert-ваме:

```XML
<dependency>
    <groupId>org.hamcrest</groupId>
    <artifactId>hamcrest-library</artifactId>
    <version>2.2</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>com.jayway.jsonpath</groupId>
    <artifactId>json-path</artifactId>
    <version>2.5.0</version>
    <scope>test</scope>
</dependency>
```

## Anotations

За да вдигне Спринг:  
```@ExtendWith(SpringExtension.class)```


За да конфигурира контекст:  
`@ContextConfiguration(classes = { ApplicationConfig.class })`

В `ApplicationConfig.class` са дефинирани бийновете.  
Може да се ползва и с XML конфигуриране:  
`@ContextConfiguration(locations={""})`

За да зареди web application context.  
WebApplicationContext Object зареждеа всички бийнове и контролери в контекста.
Дефолтния път e `src/main/webapp`. Можеш да го презапишеш с:  
`@WebAppConfiguration(value = "")`

MockMvc енкапсулира всички web application бийнове и ги прави достъпни за тестване. Зареждаш го еднокартно за целия клас с:  
```JAVA
private MockMvc mockMvc;
@BeforeEach
public void setup() throws Exception {
    this.mockMvc = MockMvcBuilders
		.webAppContextSetup(this.webApplicationContext)
		.build();
}
```
---

# Tests

## Проверяваме името на view-то

Имаме енд пойнт `http://localhost:8080/spring-mvc-test/homePage` и там трябва да има view "index":  

```JAVA
@Test
public void givenHomePageURI_whenMockMVC_thenReturnsIndexJSPViewName() {
    this.mockMvc
		.perform(get("/homePage"))
		.andDo(print())
		.andExpect(view().name("index"));
}
```

`perform()` - извиква GET метод, който връща `ResultActions`, където можем да сравним с очкваното съдържание (body), хедър или HTTP статус (200, 404...);  
`andDo(print())` - принти request-a и respons-a за да проследиш какво се случва;  
`andExpect` - сравнява полученото с очакваното;  


## Проверяваме съдържанието на respons-a (body)

Имаме енд пойнт `http://localhost:8080/spring-mvc-test/greet` и там трябва да получим отговор:

```JSON
{
    "id": 1,
    "message": "Hello World!!!"
}
```

```JAVA
@Test
public void givenGreetURI_whenMockMVC_thenVerifyResponse() {
    MvcResult mvcResult = 
		this.mockMvc
		.perform(get("/greet"))
		.andDo(print()).andExpect(status().isOk())
		.andExpect(jsonPath("$.message").value("Hello World!!!"))
		.andReturn();
    
    Assert.assertEquals("application/json;charset=UTF-8", 
      mvcResult.getResponse().getContentType());
}
```

`perform(get("/greet"))` - извиква GET метод;  
`andDo(print()).andExpect(status().isOk())` - потвърждава, че контролерът връща статус 200 (ОК);  
`andExpect(jsonPath("$.message").value("Hello World!!!"))` - потвърждава, че съдържанието на respons-a e "Hello World!!!";  
`andReturn()` - връща MvcResult обект, който се използва когато проверяваме нещо, което не е достъпно директно. В конкретния случай присвоява стойност на mvcResult променливата от тип MvcResult за да можем чрез `assertEquals` да верифицираме Content Type;  

## Проверяваме съдържанието на respons-a (body) с Path Variable

Имаме енд пойнт `http://localhost:8080/spring-mvc-test/greetWithPathVariable/John` и там трябва да получим отговор:  
```JSON
{
    "id": 1,
    "message": "Hello World John!!!"
}
```

```JAVA
@Test
public void givenGreetURIWithPathVariable_whenMockMVC_thenResponseOK() {
    this.mockMvc
      .perform(get("/greetWithPathVariable/{name}", "John"))
      .andDo(print()).andExpect(status().isOk())
      
      .andExpect(content().contentType("application/json;charset=UTF-8"))
      .andExpect(jsonPath("$.message").value("Hello World John!!!"));
}
```

`perform(get("/greetWithPathVariable/{name}", "John"))` - извиква GET метод с параметър;  








  






