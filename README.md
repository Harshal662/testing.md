Sure! Below are the steps and files you'll need to set up your project in IntelliJ IDEA.

### Step 1: Create a New Project in IntelliJ IDEA

1. Open IntelliJ IDEA.
2. Select "New Project".
3. Choose "Maven" from the left sidebar.
4. Enter your project's GroupId and ArtifactId. For example:
   - GroupId: `com.yourcompany`
   - ArtifactId: `MultiApiRequestProject`
5. Click "Finish".

### Step 2: Set Up Project Structure

IntelliJ will create a basic Maven project structure for you. You'll need to create the necessary directories and files as follows:

```
MultiApiRequestProject
├── src
│   ├── main
│   │   ├── java
│   │   │   └── com
│   │   │       └── yourpackage
│   │   │           └── MultiApiRequest.java
│   │   └── resources
│   │       └── config.properties
│   └── test
│       └── java
│           └── com
│               └── yourpackage
│                   └── MultiApiRequestTest.java
├── target
│   └── ...
├── pom.xml
└── README.md
```

### Step 3: Create the `config.properties` File

Create the `config.properties` file in the `src/main/resources` directory:

#### `config.properties`

```properties
# Proxy settings
proxyHost=your_proxy_host
proxyPort=your_proxy_port

# IDCS Token API
idcsTokenApiUrl=https://example.com/idcs/token

# Other APIs
api1Url=https://example.com/api1
api1Headers=header1:value1,header2:value2
api1Payload={"key1": "value1", "key2": "value2"}

api2Url=https://example.com/api2
api2Headers=header1:value1,header2:value2
api2Payload={"key1": "value1", "key2": "value2"}

# File paths to store responses
responseFile1=path_to_store_response1.json
responseFile2=path_to_store_response2.json
```

### Step 4: Create the `MultiApiRequest.java` File

Create the `MultiApiRequest.java` file in the `src/main/java/com/yourpackage` directory:

#### `MultiApiRequest.java`

```java
package com.yourpackage;

import java.io.*;
import java.net.HttpURLConnection;
import java.net.URL;
import java.nio.file.Files;
import java.nio.file.Paths;
import java.util.Properties;

public class MultiApiRequest {

    private static Properties properties = new Properties();

    public static void main(String[] args) {
        try {
            // Load properties
            loadProperties();

            // Get IDCS token
            String token = getIdcsToken();

            // Make API requests
            makeApiRequest(properties.getProperty("api1Url"), properties.getProperty("api1Headers"), properties.getProperty("api1Payload"), properties.getProperty("responseFile1"), token);
            makeApiRequest(properties.getProperty("api2Url"), properties.getProperty("api2Headers"), properties.getProperty("api2Payload"), properties.getProperty("responseFile2"), token);

        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    private static void loadProperties() throws IOException {
        try (InputStream input = new FileInputStream("src/main/resources/config.properties")) {
            properties.load(input);
        }
    }

    private static String getIdcsToken() throws IOException {
        URL url = new URL(properties.getProperty("idcsTokenApiUrl"));
        HttpURLConnection conn = (HttpURLConnection) url.openConnection();

        // Set proxy if needed
        System.setProperty("http.proxyHost", properties.getProperty("proxyHost"));
        System.setProperty("http.proxyPort", properties.getProperty("proxyPort"));

        conn.setRequestMethod("POST");  // Assuming it's a POST request
        conn.setDoOutput(true);

        if (conn.getResponseCode() == 200) {
            try (BufferedReader in = new BufferedReader(new InputStreamReader(conn.getInputStream()))) {
                String inputLine;
                StringBuilder content = new StringBuilder();
                while ((inputLine = in.readLine()) != null) {
                    content.append(inputLine);
                }
                return parseTokenFromResponse(content.toString());
            }
        } else {
            throw new IOException("Failed to get IDCS token: HTTP error code : " + conn.getResponseCode());
        }
    }

    private static String parseTokenFromResponse(String response) {
        // Implement JSON parsing to extract the token
        // Assuming the response JSON is like {"token": "YOUR_TOKEN"}
        // Use your preferred JSON library, e.g., org.json, Gson, Jackson, etc.
        // For simplicity, let's assume the token is directly the response for now
        return response;
    }

    private static void makeApiRequest(String apiUrl, String headers, String payload, String responseFilePath, String token) throws IOException {
        URL url = new URL(apiUrl);
        HttpURLConnection conn = (HttpURLConnection) url.openConnection();

        // Disable proxy for API request
        System.setProperty("http.proxyHost", "");
        System.setProperty("http.proxyPort", "");

        conn.setRequestMethod("POST");  // Assuming it's a POST request
        conn.setRequestProperty("Authorization", "Bearer " + token);
        conn.setDoOutput(true);

        // Set additional headers
        if (headers != null && !headers.isEmpty()) {
            String[] headerPairs = headers.split(",");
            for (String headerPair : headerPairs) {
                String[] header = headerPair.split(":");
                conn.setRequestProperty(header[0].trim(), header[1].trim());
            }
        }

        // Set JSON payload
        if (payload != null && !payload.isEmpty()) {
            try (OutputStream os = conn.getOutputStream()) {
                byte[] input = payload.getBytes("utf-8");
                os.write(input, 0, input.length);
            }
        }

        if (conn.getResponseCode() == 200) {
            try (BufferedReader in = new BufferedReader(new InputStreamReader(conn.getInputStream()))) {
                String inputLine;
                StringBuilder content = new StringBuilder();
                while ((inputLine = in.readLine()) != null) {
                    content.append(inputLine);
                }
                saveResponseToFile(responseFilePath, content.toString());
            }
        } else {
            throw new IOException("Failed to get API response: HTTP error code : " + conn.getResponseCode());
        }
    }

    private static void saveResponseToFile(String filePath, String content) throws IOException {
        Files.write(Paths.get(filePath), content.getBytes());
    }
}
```

### Step 5: Create the `pom.xml` File

Ensure your `pom.xml` includes the necessary dependencies:

#### `pom.xml`

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.yourcompany</groupId>
    <artifactId>MultiApiRequestProject</artifactId>
    <version>1.0-SNAPSHOT</version>

    <dependencies>
        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-databind</artifactId>
            <version>2.13.2</version>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.8.1</version>
                <configuration>
                    <source>11</source>
                    <target>11</target>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

### Step 6: Optional - Create Test Class

(Optional) Create the `MultiApiRequestTest.java` file in the `src/test/java/com/yourpackage` directory for unit tests:

#### `MultiApiRequestTest.java`

```java
package com.yourpackage;

import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.*;

class MultiApiRequestTest {

    @Test
    void testGetIdcsToken() {
        // Implement test for getIdcsToken method
    }

    @Test
    void testMakeApiRequest() {
        // Implement test for makeApiRequest method
    }
}
```

### Step 7: Run Your Application

1. Open the `MultiApiRequest.java` file.
2. Right-click and select "Run 'MultiApiRequest.main()'".
3. IntelliJ will build and run your project, executing the logic to make API requests and save responses to files.

This setup and structure will help you manage your project effectively within IntelliJ IDEA.
