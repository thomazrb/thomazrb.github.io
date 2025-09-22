---
title: "Practical Guide: Sending Data from ESP32 to Firebase Firestore via REST API"
date: 2025-09-22T14:37:23-03:00
draft: false
tags: ["ESP32", "Firebase", "Firestore", "IoT", "API REST", "ArduinoJson"]
categories: ["IoT", "ESP32"]
---

The need to store data in the cloud is a fundamental pillar of Internet of Things (IoT) projects. Whether for logging sensor readings, monitoring a device's state, or creating event logs, having an accessible and scalable database is essential. The ESP32, in its various versions, combined with Google's Firebase Firestore, forms a powerful and accessible duo for developers and hobbyists alike.

In this technical tutorial, we will explore the most lightweight and universal method for sending data from an ESP32 to Firestore: using the **native REST API**. Instead of relying on heavy Firebase libraries, we will build HTTP requests from scratch. This approach not only saves precious memory and resources on the microcontroller but also deepens the understanding of how web APIs work.

The goal is clear: read a value (simulated in this case) on the ESP32, connect it to Wi-Fi, and create a new record (document) in a Firestore collection.

### The Solution's Architecture: Talking to the Cloud via HTTP

Before diving into the code, it's crucial to understand the communication flow. Firestore, like many modern cloud services, exposes a REST (Representational State Transfer) API. In simple terms, this means we can interact with our database by sending HTTP requests to specific URLs, much like a browser accesses a website.

Our workflow will be as follows:

1.  **Connection**: The ESP32 connects to a Wi-Fi network to gain internet access.
2.  **Data Formatting**: The data to be sent (e.g., sensor ID and value) is structured in a specific JSON format that the Firestore API requires.
3.  **HTTP POST Request**: The ESP32 sends the JSON data to a unique URL that represents our collection in Firestore. The request is of the `POST` type, which is the standard for creating new resources.
4.  **Document Creation**: If the request is successful, Firestore creates a new document in the specified collection with the data we sent.

### Part 1: Firebase Project Configuration

The first step is to prepare the ground on the Firebase side.

#### 1.1. Create a Firebase Project

Go to the [Firebase Console](https://console.firebase.google.com/), click "Add project," and follow the instructions to create a new project.

#### 1.2. Enable Firestore Database

In your project's left-side menu, navigate to **Build > Firestore Database** and click "Create database."

* Start in **production mode**.
* Choose the server location closest to you (e.g., `southamerica-east1` for São Paulo).

#### 1.3. Configure Security Rules

For this tutorial, we will open up access to the database. **Warning: these rules are not secure for a production environment**, as they allow anyone to read and write to your database.

* In the Firestore "Rules" tab, replace the content with the following lines and click "Publish":

```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /{document=**} {
      // Allows read and write access without authentication.
      // For development purposes only.
      allow read, write: if true;
    }
  }
}
```

#### 1.4. Get Credentials (Project ID and API Key)

We need two pieces of information for our ESP32 to authenticate with the API.

1.  **Project ID**: In the menu, click the gear icon ⚙️ > **Project settings**. In the "General" tab, you will find the **Project ID**. Copy this value.
2.  **Web API Key**: On the same page, scroll down to the "Your apps" section. If there are no apps, click the Web icon (`</>`) to register a new one.
    * Give it a nickname (e.g., "ESP32 App") and click "Register app."
    * On the next screen, Firebase will show a `firebaseConfig` configuration object. Copy the value of the `apiKey`. This is your **Web API Key**.

With the ID and Key in hand, Firebase is ready.

### Part 2: Preparing the Arduino Environment

Now, let's move to our code. We need a library to facilitate JSON manipulation.

1.  **ESP32 Core**: Ensure that the ESP32 board support is installed in your Arduino IDE.
2.  **Install the `ArduinoJson` Library**: This library is essential for efficiently building the body (payload) of our HTTP request.
    * Go to `Tools > Manage Libraries...`.
    * Search for `ArduinoJson` by Benoit Blanchon and install the latest version.

The `WiFi.h` and `HTTPClient.h` libraries, which we will use for the connection and the request, are already included with the ESP32 core.

### Part 3: The ESP32 Code

This is the complete code. Copy it into a new sketch in your Arduino IDE and fill in the constants with your information.

```cpp
#include <WiFi.h>
#include <HTTPClient.h>
#include <ArduinoJson.h>

// --- FILL IN WITH YOUR INFORMATION ---
const char* WIFI_SSID = "YOUR_WIFI_SSID";
const char* WIFI_PASSWORD = "YOUR_WIFI_PASSWORD";

const char* FIREBASE_PROJECT_ID = "YOUR_FIREBASE_PROJECT_ID";
const char* FIREBASE_API_KEY = "YOUR_WEB_API_KEY";
// -----------------------------------------

// The name of the collection where data will be saved in Firestore
const char* COLLECTION_NAME = "sensor_readings";

void setup() {
  Serial.begin(115200);
  Serial.println("Starting...");

  // Connect to Wi-Fi
  WiFi.begin(WIFI_SSID, WIFI_PASSWORD);
  Serial.print("Connecting to Wi-Fi");
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println("\nSuccessfully connected!");
  Serial.print("IP Address: ");
  Serial.println(WiFi.localIP());
}

void loop() {
  // Generate a random sensor value for the example
  int sensorValue = random(0, 1024);
  
  Serial.printf("Sending data to Firestore: sensorId = 1, value = %d\n", sensorValue);
  
  // Call the function that sends the data
  sendDataToFirestore(1, sensorValue);

  // Wait 30 seconds before sending the next data point
  delay(30000); 
}

void sendDataToFirestore(int sensorId, int value) {
  if (WiFi.status() != WL_CONNECTED) {
    Serial.println("Error: Not connected to Wi-Fi.");
    return;
  }

  // Assemble the Firestore REST API URL.
  // By POSTing to the collection name, Firestore creates a random document ID.
  String url = "https://firestore.googleapis.com/v1/projects/" + String(FIREBASE_PROJECT_ID) +
               "/databases/(default)/documents/" + String(COLLECTION_NAME) +
               "?key=" + String(FIREBASE_API_KEY);

  // Create the request payload (body) in the JSON format required by Firestore.
  // This structure is the most critical part and a common source of errors.
  StaticJsonDocument<256> jsonDoc;
  
  JsonObject fields = jsonDoc.createNestedObject("fields");

  JsonObject sensorIdField = fields.createNestedObject("sensorId");
  sensorIdField["integerValue"] = sensorId;

  JsonObject valueField = fields.createNestedObject("value");
  valueField["integerValue"] = value;

  String jsonPayload;
  serializeJson(jsonDoc, jsonPayload);

  // Start the HTTP request
  HTTPClient http;
  http.begin(url);
  http.addHeader("Content-Type", "application/json");

  Serial.println("--- STARTING HTTP REQUEST ---");
  Serial.println("URL: " + url);
  Serial.println("Payload: " + jsonPayload);

  // Send the POST request with the JSON payload
  int httpResponseCode = http.POST(jsonPayload);

  // Check the server's response
  if (httpResponseCode > 0) {
    String responsePayload = http.getString();
    Serial.print("HTTP Response code: ");
    Serial.println(httpResponseCode);
    Serial.print("Response: ");
    Serial.println(responsePayload);
  } else {
    Serial.print("Error on POST request. Error code: ");
    Serial.println(httpResponseCode);
  }

  Serial.println("--- END OF HTTP REQUEST --- \n");

  // Free resources
  http.end();
}
```

#### Dissecting the Code

* **`sendDataToFirestore()`**: This function encapsulates all communication logic.
* **The API URL**: The `String url` is built dynamically with your project ID and API key. Note the endpoint: `/v1/projects/.../documents/{COLLECTION_NAME}`. By sending a `POST` to this address, we instruct Firestore to create a new document within the `sensor_readings` collection.
* **The JSON Payload**: This is the most important point. The Firestore REST API does not accept a simple JSON like `{"sensorId": 1, "value": 123}`. Instead, it requires a more detailed structure where each field is an object specifying its data type (`integerValue`, `stringValue`, `booleanValue`, etc.). The `ArduinoJson` library makes creating this complex structure a simple and safe task.
* **The `HTTPClient` Request**: We create an instance of `HTTPClient`, set the target URL (`http.begin(url)`), add the `Content-Type: application/json` header to inform the server that we are sending JSON, and finally execute the request with `http.POST(jsonPayload)`.
* **Checking the Response**: A response code of `200` means the document was created successfully. Any other code (especially in the `4xx` range) indicates an error that can be debugged by observing the server's response in the Serial Monitor.

### Testing the Integration

1.  Upload the code to your ESP32.
2.  Open the Serial Monitor (baud rate 115200).
3.  Observe the output. You should see the ESP32 connect to Wi-Fi and, every 30 seconds, print the details of the HTTP request and the server's response.
4.  Go to the Firebase Console and navigate to your Firestore database. You will see the `sensor_readings` collection and, within it, documents being created with random IDs. Clicking on a document will show the `sensorId` and `value` fields with the data you sent.

### Conclusion and Next Steps

We did it! Using only the native ESP32 libraries and `ArduinoJson`, we established a robust bridge between the physical world (our microcontroller) and the cloud. This REST API approach is extremely powerful because it is universal, lightweight, and compatible with virtually any device capable of making an HTTP request.

From here, the possibilities are immense:

* **Security**: Replace the open rules with an authentication system, like Firebase Authentication, to protect your data.
* **Data Structures**: Send more complex data, such as strings (`stringValue`), geographic coordinates (`geoPointValue`), or timestamps (`timestampValue`).
* **Reading Data**: Explore `GET` requests from the REST API to read data from Firestore and display it on your device, creating a two-way communication system.

You now have the fundamental foundation to build much more complex and connected IoT projects. Happy coding!