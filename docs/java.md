# Handling a Request

This guide explains how to implement a custom URL for handling HTTP POST requests on your web server effectively. Creating a callback URL is a crucial step in integrating your application with our services.

## Key Considerations for Callback Handling

When setting up a custom callback, it is essential to keep the following points in mind:

- **Session Management**: Each request we send to you will include a unique session ID, which lasts for 3 minutes (180 seconds). Sessions are automatically terminated after this period.

- **Session Completion**: Inform us whether a session is ongoing or complete. If a session is ongoing, begin your response with "CON." If it is the final response for that session, start your response with "END."

- **Error Handling**: In the case of an HTTP error response (status code 4XX) from your script or a malformed response that does not start with "CON" or "END," we will gracefully terminate the session.

- **Menu Content**: Ensure that your menu does not contain special characters, as some telcos may have difficulty rendering such content, which could disrupt access to your USSD services.

### API Parameters

The API makes an HTTP POST request to your server with the following parameters, which you can retrieve from the form fields of the request. The content type is `application/x-www-form-urlencoded`. This request is triggered when a user interacts with your WhatsApp bot.

## Request Parameters

- **sessionID**: A unique value generated when a session starts, and it is sent with each mobile subscriber response.

- **phoneNumber**: The phone number of the mobile subscriber interacting with your SMS/WhatsApp bot.

- **networkCode**: The telco network code associated with the phone number interacting with your bot.

- **serviceCode**: A unique code assigned to your application.

- **text**: This field represents user input. It is an empty string in the first notification of a session. Subsequently, it concatenates all user input within the session with an asterisk (*) until the session ends (e.g., 1*colls*24).

## Sample Java Script

Here is a sample java script to handle incoming requests:
```java
package com.example.ussd;

import static spark.Spark.*;
import java.util.HashMap;
import java.util.Map;

public class App {

    public static void main(String[] args) {

        post("/bot", (req, res) -> {
            // Read the variables sent via POST from our API
            Map<String, String> body = new HashMap<>();
            String[] formData = req.body().split("&");
            for (String entry : formData) {
                String[] keyValue = entry.split("=");
                body.put(keyValue[0], keyValue.length == 2 ? keyValue[1] : "");
            }

            String sessionId = body.get("sessionId");
            String serviceCode = body.get("serviceCode");
            String phoneNumber = body.get("phoneNumber");
            String text = body.get("text");

            StringBuilder response = new StringBuilder("");

            if (text.isEmpty()) {
                // This is the first request. Note how we start the response with CON
                response.append("CON Welcome to animal offsprings, select a category to continue\n");
                response.append("1. Domestic Animals\n");
                response.append("2. Wild Animals");
            } else if (text.contentEquals("1")) {
                // Business logic for the first-level response
                response.append("CON Choose an animal to know its offspring\n");
                response.append("1. Cow\n");
                response.append("2. Chicken\n");
                response.append("3. Dog\n");
            } else if (text.contentEquals("2")) {
                // Business logic for the first-level response
                response.append("CON Choose an animal to know its offspring\n");
                response.append("1. Cow\n");
                response.append("2. Chicken\n");
                response.append("3. Dog\n");
            } else if (text.contentEquals("1*1")) {
                // This is a second-level response where the user selected 1 in the first instance
                // This is a terminal request. Note how we start the response with END
                response.append("END The young one of a Cow is a calf");
            }

            return response.toString();
        });
    }
}

```