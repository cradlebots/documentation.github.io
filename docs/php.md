# 🌐 Handling a Request: Implementing Custom Callbacks

In this guide, we'll walk you through the process of setting up a custom URL to handle HTTP POST requests effectively on your web server. Creating a callback URL is a vital step in seamlessly integrating your application with our services.

## 🔑 Key Considerations for Callback Handling

When configuring a custom callback, it's crucial to keep the following key points in mind:

- **Session Management**: Each request we send to you will include a unique session ID, which remains active for 3 minutes (180 seconds). Sessions are automatically terminated after this duration.

- **Session Completion**: You must inform us whether a session is ongoing or complete. For ongoing sessions, initiate your response with "CON." When it's the final response for that session, start your reply with "END."

- **Error Handling**: In the case of an HTTP error response (status code 4XX) from your script or a malformed response that doesn't begin with "CON" or "END," we will gracefully terminate the session.

- **Menu Content**: Ensure that your menu content doesn't contain special characters, as some telcos may have difficulty rendering such content. This could potentially disrupt access to your USSD services.

### 📟 API Parameters

The API initiates an HTTP POST request to your server with the following parameters, which you can extract from the form fields of the request. The content type is `application/x-www-form-urlencoded`. This request is triggered when a user interacts with your WhatsApp bot.

## 🚀 Request Parameters

- **sessionID**: A unique value generated at the beginning of a session, and it is sent with each mobile subscriber's response.

- **phoneNumber**: The phone number of the mobile subscriber interacting with your SMS/WhatsApp bot.

- **networkCode**: The telco network code associated with the phone number interacting with your bot.

- **serviceCode**: A unique code assigned to your application.

- **text**: This field represents user input. It is an empty string in the initial notification of a session. Subsequently, it concatenates all user input within the session with an asterisk (*) until the session concludes (e.g., 1*colls*24).

## 📜 Sample php Script


Here is a sample PHP script to handle incoming requests:
*php*
```php
<?php
// Read the variables sent via POST from our API
$sessionId = $_POST["sessionId"];
$serviceCode = $_POST["serviceCode"];
$phoneNumber = $_POST["phoneNumber"];
$text = $_POST["text"];

if ($text == "") {
    // This is the first request. Note how we start the response with CON
    $response = "CON Welcome to animal offsprings, select category to continue\n";
    $response .= "1. Domestic Animals\n";
    $response .= "2. Wild Animals";
} else if ($text == "1") {
    // Business logic for the first-level response
    $response = "CON Choose an animal to know its offspring\n";
    $response .= "1. Cow\n";
    $response .= "2. Chicken\n";
    $response .= "3. Dog\n";

} else if ($text == "2") {
    // Business logic for the first-level response
    $response = "CON Choose an animal to know its offspring\n";
    $response .= "1. Cow\n";
    $response .= "2. Chicken\n";
    $response .= "3. Dog\n";
} else if ($text == "1*1") {
    // This is a second-level response where the user selected 1 in the first instance
    // This is a terminal request. Note how we start the response with END
    $response = "END The Young one of a Cow  is a calf";
}

// Echo the response back to the API
header('Content-type: text/plain');
echo $response;

?>
```