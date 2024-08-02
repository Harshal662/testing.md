Sure, I'll guide you through creating a Spring Boot project from scratch. Here’s a step-by-step guide:

### Step 1: Set Up Your Development Environment

1. **Install Java Development Kit (JDK)**
   - Download and install the latest JDK (Java Development Kit) from [Oracle](https://www.oracle.com/java/technologies/javase-jdk11-downloads.html) or [OpenJDK](https://adoptium.net/).

2. **Install an Integrated Development Environment (IDE)**
   - Recommended IDEs for Java and Spring Boot are [IntelliJ IDEA](https://www.jetbrains.com/idea/) and [Eclipse](https://www.eclipse.org/).

### Step 2: Create a New Spring Boot Project

1. **Using Spring Initializr**

   Spring Initializr is a web-based tool that generates a Spring Boot project structure for you.

   - Go to [Spring Initializr](https://start.spring.io/).
   - Fill out the form with the following settings:
     - **Project**: Maven Project (or Gradle Project if preferred)
     - **Language**: Java
     - **Spring Boot Version**: Choose the latest stable version
     - **Group**: com.example
     - **Artifact**: myapiapp
     - **Name**: MyApiApp
     - **Package Name**: com.example.myapiapp
     - **Packaging**: Jar
     - **Java**: 11 or 17 (whichever version is installed)
   - **Dependencies**: Add the following dependencies:
     - Spring Web
     - Spring Boot DevTools (for easier development)
     - Spring Configuration Processor (for configuration properties)
   - Click **Generate** to download a zip file containing your project.

2. **Extract and Import the Project**

   - Extract the zip file to your desired location.
   - Open your IDE and import the project as a Maven or Gradle project.

### Step 3: Add Required Dependencies

If you're using Maven, ensure your `pom.xml` file includes the necessary dependencies. Here's a basic example:

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-devtools</artifactId>
        <optional>true</optional>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-configuration-processor</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-webflux</artifactId>
    </dependency>
</dependencies>
```

### Step 4: Create Application Code

1. **Define Application Properties**

   Create an `application.properties` file in `src/main/resources` to store your configuration.

   ```properties
   # API Configuration
   idcs.token.url=https://idcs.example.com/token
   api1.url=https://api.example.com/endpoint1
   api2.url=https://api.example.com/endpoint2
   ```

2. **Create Token Service**

   Create a new class `TokenService` in `src/main/java/com/example/myapiapp/service/TokenService.java`.

   ```java
   package com.example.myapiapp.service;

   import org.springframework.beans.factory.annotation.Value;
   import org.springframework.stereotype.Service;
   import org.springframework.web.reactive.function.client.WebClient;
   import reactor.core.publisher.Mono;

   @Service
   public class TokenService {

       @Value("${idcs.token.url}")
       private String tokenUrl;

       private final WebClient webClient;

       public TokenService(WebClient.Builder webClientBuilder) {
           this.webClient = webClientBuilder.baseUrl(tokenUrl).build();
       }

       public Mono<String> getToken() {
           return webClient.post()
                           .retrieve()
                           .bodyToMono(String.class) // Adjust response handling as needed
                           .map(response -> extractToken(response)); // Implement extractToken method
       }

       private String extractToken(String response) {
           // Extract the token from the response
           return response; // Modify according to response format
       }
   }
   ```

3. **Create API Service**

   Create a new class `ApiService` in `src/main/java/com/example/myapiapp/service/ApiService.java`.

   ```java
   package com.example.myapiapp.service;

   import org.springframework.beans.factory.annotation.Value;
   import org.springframework.stereotype.Service;
   import org.springframework.web.reactive.function.client.WebClient;
   import reactor.core.publisher.Mono;

   import java.nio.file.Files;
   import java.nio.file.Path;
   import java.nio.file.StandardOpenOption;

   @Service
   public class ApiService {

       @Value("${api1.url}")
       private String api1Url;

       @Value("${api2.url}")
       private String api2Url;

       private final TokenService tokenService;
       private final WebClient webClient;

       public ApiService(TokenService tokenService, WebClient.Builder webClientBuilder) {
           this.tokenService = tokenService;
           this.webClient = webClientBuilder.build();
       }

       public void fetchData() {
           tokenService.getToken().flatMap(token -> {
               return Mono.zip(
                   callApi(api1Url, token, "api1_response.json"),
                   callApi(api2Url, token, "api2_response.json")
               );
           }).block(); // Use block() to make it synchronous for simplicity
       }

       private Mono<Void> callApi(String apiUrl, String token, String fileName) {
           return webClient.get()
                           .uri(apiUrl)
                           .header("Authorization", "Bearer " + token)
                           .retrieve()
                           .bodyToMono(String.class)
                           .flatMap(response -> saveToFile(response, fileName));
       }

       private Mono<Void> saveToFile(String response, String fileName) {
           return Mono.fromRunnable(() -> {
               try {
                   Path path = Path.of(fileName);
                   Files.write(path, response.getBytes(), StandardOpenOption.CREATE, StandardOpenOption.TRUNCATE_EXISTING);
               } catch (Exception e) {
                   e.printStackTrace(); // Handle the exception appropriately
               }
           });
       }
   }
   ```

4. **Create the Main Application Class**

   Update the `Application.java` file in `src/main/java/com/example/myapiapp/Application.java`.

   ```java
   package com.example.myapiapp;

   import com.example.myapiapp.service.ApiService;
   import org.springframework.boot.CommandLineRunner;
   import org.springframework.boot.SpringApplication;
   import org.springframework.boot.autoconfigure.SpringBootApplication;
   import org.springframework.context.annotation.Bean;

   @SpringBootApplication
   public class Application {

       public static void main(String[] args) {
           SpringApplication.run(Application.class, args);
       }

       @Bean
       public CommandLineRunner run(ApiService apiService) {
           return args -> {
               apiService.fetchData();
           };
       }
   }
   ```

### Step 5: Run the Application

1. **Build and Run**

   - Open a terminal in the root directory of your project.
   - Run the application with Maven:

     ```bash
     ./mvnw spring-boot:run
     ```

   - Or build and run the JAR file:

     ```bash
     ./mvnw clean package
     java -jar target/myapiapp-0.0.1-SNAPSHOT.jar
     ```

2. **Verify Output**

   - Check the project directory for files named `api1_response.json` and `api2_response.json` to verify that the responses are being saved correctly.

### Additional Tips

- **Exception Handling**: Improve error handling in the production environment.
- **Testing**: Write unit tests to ensure your services work as expected.
- **Configuration**: You can use profiles to manage different configurations for development, testing, and production environments.

This guide should help you set up and run your Spring Boot application. Let me know if you have any questions or run into any issues!
