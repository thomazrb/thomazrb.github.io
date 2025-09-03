---
title: "Complete Guide to the WT32-SC01 Plus (Part 4 of 6): Enabling Touch and Events"
date: 2025-09-03T09:56:12-03:00
draft: false
tags: ["ESP32-S3", "WT32-SC01 Plus", "LovyanGFX", "LVGL", "SquareLine Studio", "Touchscreen", "Events"]
categories: ["Hardware", "ESP32"]
---

In the previous posts, we prepared our environment, got the **WT32-SC01 Plus** screen working, and even displayed our first graphical interface created with **SquareLine Studio**. However, our screens were still static. It's time to bring our project to life by enabling the board's most important feature: the touch screen.

In this tutorial, we will transform our static interface into an interactive application. Our goal is to add a button to the screen and, when it's pressed, execute an action: printing a message to the Serial Monitor. This is the fundamental step for creating any complex application, from calculators to control panels.

### Step 1: Enabling Touch in the Main Code

So far, our `.ino` code was only concerned with drawing on the screen. For it to read touch input, we need to make two important additions: configure the touch hardware in our `LGFX` class and create the "bridge" for LVGL to understand the touch data.

1.  **Touch Hardware Configuration:** Open your sketch and add the touch driver configuration to your `LGFX` class. `LovyanGFX` already has a native driver for our board's `FT6336U` chip.

    ```cpp
    // Add this line with the other driver instances
    lgfx::Touch_FT5x06  _touch_instance;

    // Add this configuration block inside the LGFX(void) function
    { 
      auto cfg = _touch_instance.config();
      cfg.x_min = 0; cfg.x_max = 319;
      cfg.y_min = 0; cfg.y_max = 479;
      cfg.pin_int = 7;
      cfg.bus_shared = true;
      cfg.offset_rotation = 0;
      cfg.i2c_port = 0;
      cfg.i2c_addr = 0x38;
      cfg.pin_sda = 6;
      cfg.pin_scl = 5;
      cfg.freq = 400000;
      _touch_instance.config(cfg);
      _panel_instance.setTouch(&_touch_instance);
    }
    ```

2.  **Create the Touch Read Function:** Just as `my_disp_flush` is for drawing, `my_touch_read` is for reading touch input. It gets the coordinates from `LovyanGFX` and "translates" them into a format LVGL understands. Add this function to your code:

    ```cpp
    void my_touch_read(lv_indev_drv_t *indev_drv, lv_indev_data_t *data) {
        uint16_t touchX, touchY;
        bool touched = gfx.getTouch(&touchX, &touchY);
        if (touched) {
            data->point.x = touchX;
            data->point.y = touchY;
            data->state = LV_INDEV_STATE_PR; // Pressed
        } else {
            data->state = LV_INDEV_STATE_REL; // Released
        }
    }
    ```

3.  **Register the Touch Driver in `setup()`:** Finally, we need to tell LVGL to use our `my_touch_read` function as an input source. Add this block to your `setup()`:

    ```cpp
    static lv_indev_drv_t indev_drv;
    lv_indev_drv_init(&indev_drv);
    indev_drv.type = LV_INDEV_TYPE_POINTER;
    indev_drv.read_cb = my_touch_read;
    lv_indev_drv_register(&indev_drv);
    ```

### Step 2: Creating the Interactive Interface in SquareLine Studio

1.  Create a new project in SquareLine Studio (or use the previous one).
2.  Drag a **Button** widget onto the screen.
3.  Inside the button, add a **Label** and change its text to something like "Click Me".
4.  Select the **Button** object. In the "Inspector" panel on the right, scroll down to the **Events** section.
5.  Click **"Add Event"**.
6.  Configure the event as follows:
    * **Trigger:** `CLICKED` (or `PRESSED`).
    * **Action:** `Call function`.
    * **Function:** Type the name of the function we want to call when the button is clicked, for example, `buttonPressed`.

7.  Export the UI files as we did in the previous post.

### Step 3: The Event Logic in `ui_events.cpp`

When you export, SquareLine Studio creates two files to handle events: `ui_events.h` and `ui_events.c`. This is where we will write what happens when the button is pressed.

By default, the `ui_events.c` file is a pure C file. However, to use Arduino functions like `Serial.println()`, we need it to be compiled as C++. The solution is simple:

**Rename the `ui_events.c` file to `ui_events.cpp`.**

Now, open your new `ui_events.cpp` and add the logic for our `buttonPressed` function:

```cpp
/*
 * ui_events.cpp
 * Logic for the SquareLine Studio interface events.
 */
#include <Arduino.h> // Essential for using Arduino functions
#include "ui.h"

void buttonPressed(lv_event_t * e)
{
    // This function will be called whenever the button is clicked
    Serial.println("The button was pressed!");
    
    // Any other logic can be added here.
}
```

### Step 4: The Final Code and Result

Putting it all together, our final `.ino` file looks like this. Note that it's very similar to the previous example, but now with the complete touch configuration.

```cpp
/*
 * EXAMPLE 3: INTERACTIVE BUTTON WITH SQUARELINE AND LOVYANGFX
 * * This sketch enables touch and responds to a click event
 * on a button created in SquareLine Studio.
 */

#define LGFX_USE_V1
#include <LovyanGFX.hpp>
#include <lvgl.h>
#include "ui.h"

// Complete LGFX class with TOUCH configuration included
class LGFX : public lgfx::LGFX_Device {
  lgfx::Panel_ST7796  _panel_instance;
  lgfx::Bus_Parallel8 _bus_instance;
  lgfx::Light_PWM     _light_instance;
  lgfx::Touch_FT5x06  _touch_instance; // Touch driver
public:
  LGFX(void) {
    { // BUS
      auto cfg = _bus_instance.config();
      cfg.port = 0; cfg.freq_write = 20000000;
      cfg.pin_wr = 47; cfg.pin_rd = -1; cfg.pin_rs = 0;
      cfg.pin_d0 = 9; cfg.pin_d1 = 46; cfg.pin_d2 = 3; cfg.pin_d3 = 8;
      cfg.pin_d4 = 18; cfg.pin_d5 = 17; cfg.pin_d6 = 16; cfg.pin_d7 = 15;
      _bus_instance.config(cfg);
      _panel_instance.setBus(&_bus_instance);
    }
    { // PANEL
      auto cfg = _panel_instance.config();
      cfg.pin_cs = -1; cfg.pin_rst = 4; cfg.pin_busy = -1;
      cfg.memory_width = 320; cfg.memory_height = 480;
      cfg.panel_width = 320; cfg.panel_height = 480;
      cfg.offset_x = 0; cfg.offset_y = 0;
      cfg.invert = true; cfg.rgb_order = false;
      _panel_instance.config(cfg);
    }
    { // BACKLIGHT
      auto cfg = _light_instance.config();
      cfg.pin_bl = 45; cfg.invert = false; cfg.freq = 44100; cfg.pwm_channel = 7;
      _light_instance.config(cfg);
      _panel_instance.setLight(&_light_instance);
    }
    { // TOUCH
      auto cfg = _touch_instance.config();
      cfg.x_min = 0; cfg.x_max = 319;
      cfg.y_min = 0; cfg.y_max = 479;
      cfg.pin_int = 7;
      cfg.bus_shared = true;
      cfg.offset_rotation = 0;
      cfg.i2c_port = 0; cfg.i2c_addr = 0x38;
      cfg.pin_sda = 6; cfg.pin_scl = 5;
      cfg.freq = 400000;
      _touch_instance.config(cfg);
      _panel_instance.setTouch(&_touch_instance);
    }
    setPanel(&_panel_instance);
  }
};

LGFX gfx; 
static const uint16_t screenWidth  = 480;
static const uint16_t screenHeight = 320;
static lv_disp_draw_buf_t draw_buf;
static lv_color_t buf1[screenWidth * screenHeight / 10];

void my_disp_flush(lv_disp_drv_t *disp_drv, const lv_area_t *area, lv_color_t *color_p) {
    uint32_t w = (area->x2 - area->x1 + 1);
    uint32_t h = (area->y2 - area->y1 + 1);
    gfx.startWrite();
    gfx.setAddrWindow(area->x1, area->y1, w, h);
    gfx.pushPixels((uint16_t *)color_p, w * h, true);
    gfx.endWrite();
    lv_disp_flush_ready(disp_drv);
}

void my_touch_read(lv_indev_drv_t *indev_drv, lv_indev_data_t *data) {
    uint16_t touchX, touchY;
    bool touched = gfx.getTouch(&touchX, &touchY);
    if (touched) {
        data->point.x = touchX;
        data->point.y = touchY;
        data->state = LV_INDEV_STATE_PR;
    } else {
        data->state = LV_INDEV_STATE_REL;
    }
}

void setup() {
    Serial.begin(115200);
    Serial.println("Starting Example 3: Interactive Button");
    gfx.begin();
    gfx.setRotation(1);
    gfx.setBrightness(255);
    lv_init();
    lv_disp_draw_buf_init(&draw_buf, buf1, NULL, screenWidth * screenHeight / 10);
    
    static lv_disp_drv_t disp_drv;
    lv_disp_drv_init(&disp_drv);
    disp_drv.hor_res = screenWidth;
    disp_drv.ver_res = screenHeight;
    disp_drv.flush_cb = my_disp_flush;
    disp_drv.draw_buf = &draw_buf;
    lv_disp_drv_register(&disp_drv);

    static lv_indev_drv_t indev_drv;
    lv_indev_drv_init(&indev_drv);
    indev_drv.type = LV_INDEV_TYPE_POINTER;
    indev_drv.read_cb = my_touch_read;
    lv_indev_drv_register(&indev_drv);

    ui_init(); 
    Serial.println("Setup finished. The interface and touch should now work.");
}

void loop() {
    lv_timer_handler();
    lv_tick_inc(5);
    delay(5);
}
```

Compile and upload the code. Now, when you touch the button on the screen, you will see the message "The button was pressed!" appear in your Serial Monitor!

### Conclusion

Congratulations! You have just created your first fully interactive application on the WT32-SC01 Plus. We learned how to enable touch, configure events in SquareLine Studio, and respond to those events in our code.

With this foundation, the possibilities are endless. In the next post, we will use what we've learned to create something even more practical: a counter that increments and decrements a value on the screen.

See you then!