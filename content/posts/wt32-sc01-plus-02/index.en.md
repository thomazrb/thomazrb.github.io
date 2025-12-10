---
title: "Complete Guide to the WT32-SC01 Plus (Part 2 of 6): The First Code (Hello World!)"
date: 2025-08-25T16:40:00-03:00
lastmod: 2024-12-10T10:16:00-03:00
draft: false
tags: ["ESP32-S3", "WT32-SC01 Plus", "LovyanGFX", "Arduino", "Hello World"]
categories: ["Hardware", "ESP32"]
---

In the first post of this series, we got acquainted with the **WT32-SC01 Plus** board and its main features. Now, it's time to get our hands dirty and do what we love most: write code and see something happen on the screen!

In this tutorial, we will set up the development environment in the Arduino IDE and create our first program: the classic "Hello World!". The goal here is to ensure that communication with the display is working perfectly, without the complexity of UI libraries like LVGL. For this, we will use the **LovyanGFX** library.

While other excellent libraries for display control exist, such as the popular `TFT_eSPI` or `Arduino_GFX`, we chose `LovyanGFX` for this tutorial for one main reason: it offers an integrated solution, controlling both the display and the touch functionalities in a single library. This simplifies the setup and the code, as we will see in the upcoming posts.

### Step 1: Setting Up the Environment in Arduino IDE

Before writing any code, we need to ensure the Arduino IDE is prepared to communicate with our board and the graphics library.

#### 1.1 Install the ESP32 Board Package

Compatibility between the libraries and the ESP32 core is crucial. Newer versions in the 3.x.x series caused compilation errors with the LovyanGFX library, so, as we discovered in our tests, the most stable version for this project is **2.0.17**.

1.  In the Arduino IDE, go to **File > Preferences**.
2.  In the "Additional Board Manager URLs" field, add the following URL:
    ```
    https://raw.githubusercontent.com/espressif/arduino-esp32/gh-pages/package_esp32_index.json
    ```
3.  Go to **Tools > Board > Boards Manager...**.
4.  Search for `esp32`, select "esp32 by Espressif Systems", choose version **2.0.17**, and click "Install".

#### 1.2 Install the LovyanGFX Library

As mentioned, this library will be responsible for controlling the display.

1.  Go to **Tools > Manage Libraries...**.
2.  Search for `LovyanGFX` and install the latest version.

### Step 2: The "Hello World!" Code

With the environment ready, create a new sketch in the Arduino IDE and paste the complete code below. This code already contains all the necessary hardware configuration for our board.

```cpp
/*
 * EXAMPLE 1: HELLO WORLD WITH LOVYANGFX
 * * This sketch demonstrates the basic use of the LovyanGFX library to
 * initialize the screen of the WT32-SC01 Plus board and write text.
 */

#define LGFX_USE_V1 // Define that we are using Version 1 of LovyanGFX

#include <LovyanGFX.hpp>

// ======================================================================================
// === 1. HARDWARE CONFIGURATION (LOVYANGFX) ==========================================
// ======================================================================================
// This class describes the screen hardware for the LovyanGFX library.
class LGFX : public lgfx::LGFX_Device
{
  // Driver instances for each screen hardware component
  lgfx::Panel_ST7796  _panel_instance;
  lgfx::Bus_Parallel8 _bus_instance;
  lgfx::Light_PWM     _light_instance;

public:
  LGFX(void)
  {
    // --- Screen Bus Configuration (BUS) ---
    { 
      auto cfg = _bus_instance.config();
      cfg.port = 0;
      cfg.freq_write = 20000000;
      cfg.pin_wr = 47;
      cfg.pin_rd = -1;
      cfg.pin_rs = 0;
      cfg.pin_d0 = 9; cfg.pin_d1 = 46; cfg.pin_d2 = 3; cfg.pin_d3 = 8;
      cfg.pin_d4 = 18; cfg.pin_d5 = 17; cfg.pin_d6 = 16; cfg.pin_d7 = 15;
      _bus_instance.config(cfg);
      _panel_instance.setBus(&_bus_instance);
    }

    // --- Screen Panel Configuration (PANEL) ---
    { 
      auto cfg = _panel_instance.config();
      cfg.pin_cs = -1;
      cfg.pin_rst = 4;
      cfg.pin_busy = -1;
      cfg.memory_width = 320;
      cfg.memory_height = 480;
      cfg.panel_width = 320;
      cfg.panel_height = 480;
      cfg.offset_x = 0;
      cfg.offset_y = 0;
      // Combination that corrects the colors for this screen model
      cfg.invert = true;
      cfg.rgb_order = false;
      _panel_instance.config(cfg);
    }

    // --- Backlight Configuration (BACKLIGHT) ---
    { 
      auto cfg = _light_instance.config();
      cfg.pin_bl = 45;
      cfg.invert = false;
      cfg.freq = 44100;
      cfg.pwm_channel = 7;
      _light_instance.config(cfg);
      _panel_instance.setLight(&_light_instance);
    }
    
    setPanel(&_panel_instance);
  }
};

// Main instance of the LovyanGFX driver that will be used in the code
LGFX gfx; 

void setup()
{
    Serial.begin(115200);
    Serial.println("Starting Example 1: Hello World with LovyanGFX");

    // --- 1. Initialize Hardware ---
    gfx.begin();
    gfx.setRotation(1); // 1 = Landscape (USB cable to the right)
    gfx.setBrightness(255); // Maximum brightness

    // --- 2. Draw on the Screen ---
    gfx.fillScreen(TFT_BLACK); // Fills the background with black

    gfx.setTextSize(4); // Sets the font size (4x scale)
    gfx.setTextColor(TFT_WHITE); // Sets the text color to white
    
    // Center the text on the screen
    const char* text = "Hello World!";
    int16_t textWidth = gfx.textWidth(text);
    int16_t textHeight = gfx.fontHeight();
    int16_t cursorX = (gfx.width() - textWidth) / 2;
    int16_t cursorY = (gfx.height() - textHeight) / 2;
    
    gfx.setCursor(cursorX, cursorY); // Positions the cursor for the text
    gfx.println(text); // Prints the text on the screen

    Serial.println("Text 'Hello World!' written to the screen.");
}

void loop()
{
  // The loop is empty, as the action happens only once in setup.
  delay(1000);
}
```
### Step 3: Understanding the Code

* **`LGFX` Class**: This initial block is the most important part. It acts as an "instruction manual" for the library, describing exactly how the WT32-SC01 Plus hardware is assembled: which pins are used for screen communication, the resolution, how to correct the colors, etc.
* **`setup()`**: This is where the magic happens.
    * `gfx.begin()`: Initializes the screen with the settings we defined.
    * `gfx.setRotation(1)`: Rotates the screen to landscape mode.
    * `gfx.fillScreen(TFT_BLACK)`: Fills the entire screen with black.
    * `gfx.setTextSize(4)` and `gfx.setTextColor(TFT_WHITE)`: Configure the appearance of our text.
    * **Centering**: The next block calculates the exact position to make the "Hello World!" text perfectly centered.
    * `gfx.println(text)`: Finally, it draws the text on the screen.
* **`loop()`**: Since our goal is just to write the text once, the main loop remains empty.

### Conclusion

Congratulations! If everything went well, you should now have the phrase "Hello World!" shining on the screen of your WT32-SC01 Plus. This confirms that your environment is set up correctly and that communication with the display is 100% functional.

In the next post, we will take it a step further: we will introduce the **LVGL** library and the **SquareLine Studio** tool to create our first graphical interface, still without touch, but already showing the power of these tools.

See you then!