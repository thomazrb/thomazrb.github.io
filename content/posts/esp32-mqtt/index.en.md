---
title: "ESP32: Sending Data via MQTT to the Cloud"
date: 2026-01-19T11:11:11-03:00
draft: false
tags: ["ESP32", "MQTT", "IoT", "Arduino IDE"]
categories: ["IoT", "ESP32"]
---

The Internet of Things (IoT) has revolutionized the way devices communicate, and the MQTT (Message Queuing Telemetry Transport) protocol is one of the most popular and efficient ways to make this communication happen. In this tutorial, we'll connect an ESP32 to the internet via WiFi and make it periodically send data to a public MQTT broker, allowing you to view this information in real-time both in the browser and in an Android app.

The ESP32 is a powerful microcontroller with integrated WiFi and Bluetooth, perfect for IoT projects. We'll create a simple example that sends an incrementing counter every 30 seconds, demonstrating the fundamental concepts of MQTT communication that you can apply to more complex projects.

**Versions used in this tutorial:**
- Arduino IDE: 2.3.7
- ESP32 Board Package: 3.3.5
- PubSubClient (MQTT library): 2.8.0

### What is MQTT?

MQTT is a lightweight messaging protocol, designed specifically for devices with limited resources and networks with low bandwidth. It works on the publish/subscribe model:

* **Broker:** The central server that receives all messages and distributes them to interested parties (in our case, HiveMQ's public broker).
* **Publisher:** The device that publishes messages to a topic (our ESP32).
* **Subscriber:** The device or application that subscribes to topics to receive messages (our browser or Android app).
* **Topic:** It's like a "channel" where messages are published. For example: `home/living_room/temperature`.

### Step 1: Preparing the Arduino IDE

Before we start programming, we need to configure the Arduino IDE to work with the ESP32.

#### 1.1: Install ESP32 Support

If you don't have ESP32 support installed in the Arduino IDE yet:

1. Open the Arduino IDE
2. Go to **File → Preferences** (or **Arduino IDE → Settings** on macOS)
3. In the **"Additional Boards Manager URLs"** field, add:
   ```
   https://raw.githubusercontent.com/espressif/arduino-esp32/gh-pages/package_esp32_index.json
   ```
4. Click **OK**
5. Go to **Tools → Board → Boards Manager**
6. Search for "esp32" and install the **"esp32 by Espressif Systems"** package
7. Wait for the installation to complete

#### 1.2: Install the MQTT Library

We need the PubSubClient library for MQTT communication:

1. Go to **Sketch → Include Library → Manage Libraries** (or **Tools → Manage Libraries**)
2. Search for "PubSubClient"
3. Install the **"PubSubClient" by Nick O'Leary** library

#### 1.3: Select the ESP32 Board

1. Connect your ESP32 to the computer via USB
2. Go to **Tools → Board** and select your ESP32 model (usually **"ESP32 Dev Module"** works for most models)
3. In **Tools → Port**, select the COM port where your ESP32 is connected

### Step 2: Understanding the Code

Our program will do the following:

1. Connect the ESP32 to the WiFi network
2. Connect to the public MQTT broker
3. Every 30 seconds, send an incrementing counter to an MQTT topic

### Step 3: Complete Code

Create a new sketch in the Arduino IDE and paste the code below. **Important:** replace `YOUR_WIFI_HERE` and `YOUR_PASSWORD_HERE` with your WiFi network credentials.

```cpp
#include <WiFi.h>
#include <PubSubClient.h>

// WiFi settings - REPLACE with your credentials
const char* ssid = "YOUR_WIFI_HERE";
const char* password = "YOUR_PASSWORD_HERE";

// MQTT broker settings
const char* mqtt_server = "mqtt-dashboard.com";
const int mqtt_port = 1883;

// MQTT topic where data will be published
// Use a unique identifier to avoid conflicts with other users
const char* mqtt_topic = "esp32/thomazrb/counter";

// WiFi and MQTT clients
WiFiClient espClient;
PubSubClient client(espClient);

// Control variables
unsigned long lastMsg = 0;
int counter = 0;

// Function to connect to WiFi
void setup_wifi() {
  delay(10);
  Serial.println();
  Serial.print("Connecting to WiFi: ");
  Serial.println(ssid);

  WiFi.begin(ssid, password);

  // Wait until connected
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }

  Serial.println("");
  Serial.println("WiFi connected!");
  Serial.print("IP address: ");
  Serial.println(WiFi.localIP());
}

// Callback function - called when a message is received
// In this simple example, we won't subscribe to any topic, but the function is mandatory
void callback(char* topic, byte* payload, unsigned int length) {
  Serial.print("Message received on topic: ");
  Serial.println(topic);
  
  Serial.print("Content: ");
  for (int i = 0; i < length; i++) {
    Serial.print((char)payload[i]);
  }
  Serial.println();
}

void setup() {
  // Initialize serial communication for debugging
  Serial.begin(115200);
  
  // Connect to WiFi
  setup_wifi();
  
  // Configure MQTT server and callback function
  client.setServer(mqtt_server, mqtt_port);
  client.setCallback(callback);
  
  // Connect to MQTT broker
  Serial.print("Connecting to MQTT broker...");
  
  // Generate a unique client ID
  String clientId = "ESP32Client-";
  clientId += String(random(0xffff), HEX);
  
  if (client.connect(clientId.c_str())) {
    Serial.println("connected!");
    // Publish a confirmation message
    client.publish(mqtt_topic, "ESP32 Online");
  } else {
    Serial.print("failed, rc=");
    Serial.println(client.state());
  }
}

void loop() {
  // Keep MQTT connection alive
  client.loop();

  // Send message every 30 seconds
  unsigned long now = millis();
  if (now - lastMsg > 30000) { // 30000 ms = 30 seconds
    lastMsg = now;
    
    // Increment counter
    counter++;
    
    // Convert counter to string
    char msg[50];
    snprintf(msg, 50, "Counter: %d", counter);
    
    // Publish to MQTT topic
    Serial.print("Publishing message: ");
    Serial.println(msg);
    client.publish(mqtt_topic, msg);
  }
}
```

### Step 4: Upload and Test

1. Click the **Upload** button (right arrow) in the Arduino IDE
2. Wait for the upload to complete
3. Open the **Serial Monitor** (magnifying glass icon in the upper right corner or **Tools → Serial Monitor**)
4. Set the speed to **115200 baud**
5. You should see messages indicating connection to WiFi and the MQTT broker

### Step 5: Viewing Data with MQTT Websocket Client

You can use HiveMQ's free web client to test MQTT connections:

1. Access: [https://www.hivemq.com/demos/websocket-client/](https://www.hivemq.com/demos/websocket-client/)
2. Click **"Connect"** (the default settings are already correct)
3. After connecting, go to the **"Subscriptions"** section
4. In **"Topic"**, type exactly the same topic you used in the code: `esp32/thomazrb/counter`
5. **QoS:** 0
6. Click **"Subscribe"**
7. You'll start seeing messages arriving every 30 seconds!

### Step 6: Viewing on Android with IoT MQTT Panel

To track the data on your phone:

1. Install the **"IoT MQTT Panel"** app from the Play Store
2. Open the app and tap the red **"SETUP A CONNECTION"** button
3. Fill in the fields:
   - **Connection name:** MQTT Dashboard (or any name you prefer)
   - **Client ID:** leave empty (will be generated automatically)
   - **Broker address:** `mqtt-dashboard.com`
   - **Port:** `8883`
   - **Network protocol:** TCP-SSL (important to change from TCP to TCP-SSL!)
4. Tap **"Add Dashboard"** and give the dashboard a name (e.g., "My Dashboard")
5. Tap **"CREATE"**
6. The app will connect to the broker and open the empty dashboard
7. Tap the blue **"ADD A PANEL"** button in the center of the screen (after adding the first panel, the next ones are added via the **"+"** button in the lower right corner)
8. Select **"Text Output"**
9. Configure only the required fields:
   - **Panel name:** ESP32 Counter
   - **Topic:** `esp32/thomazrb/counter` (use the same topic as in the code!)
   - **QoS:** 0 (already default)
10. Tap **"CREATE"**
11. The ESP32 messages will appear automatically every 30 seconds!

### Understanding the Code: Important Details

#### WiFi and Credentials

```cpp
const char* ssid = "YOUR_WIFI_HERE";
const char* password = "YOUR_PASSWORD_HERE";
```

These constants store your WiFi network credentials. The ESP32 needs this information to connect to the internet.

#### MQTT Broker Configuration

```cpp
const char* mqtt_server = "mqtt-dashboard.com";
const int mqtt_port = 1883;
const char* mqtt_topic = "esp32/thomazrb/counter";
```

- **mqtt_server:** Public MQTT broker address
- **mqtt_port:** Standard port for MQTT without encryption (use 8883 for TLS/SSL)
- **mqtt_topic:** The "channel" where we'll publish messages. **Important:** change `thomazrb` to your own unique identifier to avoid conflicts with other users of the public broker!

#### Non-blocking Timing

```cpp
unsigned long now = millis();
if (now - lastMsg > 30000) {
    // code executed every 30 seconds
}
```

We use `millis()` instead of `delay()` because `delay()` blocks all program execution. With `millis()`, the ESP32 can continue processing other tasks (like maintaining the MQTT connection) while waiting for the 30 seconds.

#### Unique Client ID

```cpp
String clientId = "ESP32Client-";
clientId += String(random(0xffff), HEX);
```

Each MQTT client needs to have a unique ID. We generate a random one to avoid conflicts in case you connect multiple ESP32s to the same broker.

### What is QoS (Quality of Service)?

QoS (Quality of Service) is an important concept in MQTT that defines the level of message delivery guarantee. There are three levels:

* **QoS 0 (At most once):** The message is sent once, without confirmation. It's the fastest, but messages can be lost if there are network problems. Ideal for data that can be occasionally lost (like temperature readings that repeat every few seconds).

* **QoS 1 (At least once):** The message is delivered at least once, with confirmation. There may be duplicates. Good for important data where duplicates are not a problem.

* **QoS 2 (Exactly once):** The message is delivered exactly once, with a more complex confirmation protocol. Slower, but guarantees single delivery. Ideal for critical commands or financial transactions.

**In our example we use QoS 0** because:
- It's a simple counter that repeats every 30 seconds
- If a message is lost, the next one will come soon
- It keeps the code and communication simpler and faster

For projects where each message is critical (like alarms, actuator commands, or data that doesn't repeat), consider using QoS 1 or 2.

### Security and Best Practices

**Important about Public Brokers:**

- The MQTT broker is **public and without authentication**
- Anyone can read your messages if they know the topic
- **Never send sensitive data** (passwords, personal information, etc.)
- For professional projects, use a private broker with authentication

**Security Tips:**

1. Use topics with unique identifiers (e.g., `esp32/user_12345/sensor`)
2. For production, consider using private brokers like:
   - **CloudMQTT** (now part of CloudAMQP)
   - **AWS IoT Core**
   - **Azure IoT Hub**
   - **Your own Mosquitto server** ([https://mosquitto.org/](https://mosquitto.org/))
3. Implement TLS/SSL to encrypt communication
4. Use authentication (username/password) whenever possible

### Next Steps and Improvements

Now that you have an ESP32 sending data via MQTT, here are some ideas to expand the project:

1. **Add Real Sensors:** Replace the counter with sensor readings (temperature, humidity, light)
2. **Receive Commands:** Make the ESP32 subscribe to a topic to receive commands (turn LEDs on/off, for example)
3. **JSON Format:** Send structured data in JSON to facilitate parsing
4. **Deep Sleep:** Use ESP32's low power mode between sends to save battery

### Conclusion

Congratulations! You've created your first IoT application with ESP32 and MQTT. This simple example demonstrates the fundamental concepts that are the foundation of complex IoT systems. The MQTT protocol is extremely versatile and scalable, being used from home projects to industrial applications.

From here, you have a solid foundation to create your own IoT projects, connecting sensors, actuators, and building custom dashboards. The ESP32 is a powerful and affordable platform, and MQTT is the perfect protocol to make your devices talk!

**Additional Resources:**
* [ESP32 Arduino Core Documentation](https://docs.espressif.com/projects/arduino-esp32/en/latest/)
* [PubSubClient Library Documentation](http://pubsubclient.knolleary.net/)
* [HiveMQ MQTT Essentials](https://www.hivemq.com/mqtt-essentials/)
* [MQTT.org - Official MQTT Site](https://mqtt.org/)