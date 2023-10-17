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

## Sample Node Script

Here is a sample node script to handle incoming requests:
```js
const express = require('express');
const bodyParser = require('body-parser');

const app = express();
app.use(bodyParser.json());
app.use(bodyParser.urlencoded({ extended: false }));

app.post('/ussd', (req, res) => {
    // Read the variables sent via POST from our API
    const {
        sessionId,
        serviceCode,
        phoneNumber,
        text,
    } = req.body;

    let response = '';

    if (text === '') {
        // This is the first request. Note how we start the response with CON
        response = 'CON Welcome to animal offsprings, select a category to continue\n1. Domestic Animals\n2. Wild Animals';
    } else if (text === '1') {
        // Business logic for first-level response
        response = 'CON Choose an animal to know its offspring\n1. Cow\n2. Chicken\n3. Dog';
    } else if (text === '2') {
        // Business logic for first-level response
        response = `CON Choose an animal to know its offspring\n1. Cow\n2. Chicken\n3. Dog`;
    } else if (text === '1*1') {
        // This is a second-level response where the user selected 1 in the first instance
        // This is a terminal request. Note how we start the response with END
        response = 'END The young one of a Cow is a calf';
    }

    // Send the response back to the API
    res.set('Content-Type', 'text/plain');
    res.send(response);
});

app.listen(3000, () => {
    console.log('Server is running on port 3000');
});


```