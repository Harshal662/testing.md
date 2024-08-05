Got it! Let's update the `config.properties` and `MultiApiRequest.java` files to include form parameters and headers for the IDCS token request.

### Updated `config.properties`

Add entries for the IDCS token form parameters and headers:

```properties
# Proxy settings
proxyHost=ig-proxies.gslb.global
proxyPort=8080
proxyUsername=G0134343
proxyPassword=Sarjerao%1154135

# IDCS Token API
idcsTokenApiUrl=https://example.com/idcs/token
idcsTokenFormParameters=param1=value1,param2=value2
idcsTokenHeaders=header1:value1,header2:value2

# Other APIs
api1Url=https://example.com/api1
api1Headers=header1:value1,header2:value2
api1Payload={"ABC": {"XYZ": "24379847"}}

api2Url=https://example.com/api2
api2Headers=header1:value1,header2:value2
api2Payload={"ABC": {"XYZ": "24379847"}}

# File paths to store responses
responseFile1=path_to_store_response1.json
responseFile2=path_to_store_response2.json
```

### Updated `MultiApiRequest.java`

Update the `MultiApiRequest.java` to handle form parameters and headers for the IDCS token request:

```java
package com.yourpackage;

import java.io.*;
import java.net.Authenticator;
import java.net.HttpURLConnection;
import java.net.InetSocketAddress;
import java.net.PasswordAuthentication;
import java.net.Proxy;
import java.net.URL;
import java.nio.file.Files;
import java.nio.file.Paths;
import java.util.Properties;

import com.fasterxml.jackson.databind.JsonNode;
import com.fasterxml.jackson.databind.ObjectMapper;

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
        String proxyHost = properties.getProperty("proxyHost");
        int proxyPort = Integer.parseInt(properties.getProperty("proxyPort"));
        String proxyUsername = properties.getProperty("proxyUsername");
        String proxyPassword = properties.getProperty("proxyPassword");

        Proxy proxy = new Proxy(Proxy.Type.HTTP, new InetSocketAddress(proxyHost, proxyPort));

        // Set up proxy authentication
        Authenticator authenticator = new Authenticator() {
            public PasswordAuthentication getPasswordAuthentication() {
                return new PasswordAuthentication(proxyUsername, proxyPassword.toCharArray());
            }
        };
        Authenticator.setDefault(authenticator);

        HttpURLConnection conn = (HttpURLConnection) url.openConnection(proxy);
        conn.setRequestMethod("POST");  // Assuming it's a POST request
        conn.setDoOutput(true);

        // Set IDCS token headers
        String idcsTokenHeaders = properties.getProperty("idcsTokenHeaders");
        if (idcsTokenHeaders != null && !idcsTokenHeaders.isEmpty()) {
            String[] headerPairs = idcsTokenHeaders.split(",");
            for (String headerPair : headerPairs) {
                String[] header = headerPair.split(":");
                conn.setRequestProperty(header[0].trim(), header[1].trim());
            }
        }

        // Set IDCS token form parameters
        String idcsTokenFormParameters = properties.getProperty("idcsTokenFormParameters");
        if (idcsTokenFormParameters != null && !idcsTokenFormParameters.isEmpty()) {
            StringBuilder formParams = new StringBuilder();
            String[] paramPairs = idcsTokenFormParameters.split(",");
            for (String paramPair : paramPairs) {
                if (formParams.length() > 0) {
                    formParams.append("&");
                }
                formParams.append(paramPair);
            }
            byte[] formDataBytes = formParams.toString().getBytes("UTF-8");
            conn.getOutputStream().write(formDataBytes);
        }

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

    private static String parseTokenFromResponse(String response) throws IOException {
        // Implement JSON parsing to extract the token
        ObjectMapper objectMapper = new ObjectMapper();
        JsonNode jsonNode = objectMapper.readTree(response);
        return jsonNode.get("token").asText();
    }

    private static void makeApiRequest(String apiUrl, String headers, String payload, String responseFilePath, String token) throws IOException {
        URL url = new URL(apiUrl);
        HttpURLConnection conn = (HttpURLConnection) url.openConnection();

        // Disable proxy for API request
        System.clearProperty("http.proxyHost");
        System.clearProperty("http.proxyPort");

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
            conn.setRequestProperty("Content-Type", "application/json; utf-8");
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

### Summary of Changes
1. **Properties File**: Added `idcsTokenFormParameters` and `idcsTokenHeaders` entries in the `config.properties` file.
2. **Form Parameters and Headers**: Updated `getIdcsToken` to handle form parameters and headers for the IDCS token request.

This updated code and configuration should now correctly handle the form parameters and headers required for the IDCS token request, while also using the proxy with authentication details.
