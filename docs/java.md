# 🌐 Handling a Request: Setting Up Custom Callbacks

This comprehensive guide will walk you through the process of implementing a custom URL for handling HTTP POST requests on your web server effectively. The creation of a callback URL is a pivotal step in seamlessly integrating your application with our services.

## 🔑 Key Considerations for Callback Handling

When configuring a custom callback, it is imperative to keep the following essential considerations in mind:

- **Session Management**: Each request sent to you will include a unique session ID, which remains active for 3 minutes (180 seconds). After this duration, sessions are automatically terminated.

- **Session Completion**: You must inform us whether a session is ongoing or complete. For ongoing sessions, initiate your response with "CON." When it's the final response for that session, start your reply with "END."

- **Error Handling**: In case of an HTTP error response (status code 4XX) from your script or a malformed response that doesn't begin with "CON" or "END," we will gracefully terminate the session.

- **Menu Content**: Ensure that your menu content does not contain special characters, as some telcos may have difficulty rendering such content. This could potentially disrupt access to your USSD services.

### 📟 API Parameters

The API initiates an HTTP POST request to your server with the following parameters, which you can extract from the form fields of the request. The content type is `application/x-www-form-urlencoded`. This request is triggered when a user interacts with your WhatsApp bot.

## 🚀 Request Parameters

- **sessionID**: A unique value generated at the beginning of a session, and it is sent with each mobile subscriber's response.

- **phoneNumber**: The phone number of the mobile subscriber interacting with your SMS/WhatsApp bot.

- **networkCode**: The telco network code associated with the phone number interacting with your bot.

- **serviceCode**: A unique code assigned to your application.

- **text**: This field represents user input. It is an empty string in the initial notification of a session. Subsequently, it concatenates all user input within the session with an asterisk (*) until the session concludes (e.g., 1*colls*24).

## 📜 Sample Java code

Here's a sample Java code for handling incoming requests:

```java
package com.example.bot;

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
