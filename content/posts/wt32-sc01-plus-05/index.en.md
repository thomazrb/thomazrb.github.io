---
title: "Complete Guide to the WT32-SC01 Plus (Part 5 of 6): Manipulating Widgets (Interactive Counter)"
date: 2025-09-03T19:33:57-03:00
draft: false
tags: ["ESP32-S3", "WT32-SC01 Plus", "LovyanGFX", "LVGL", "SquareLine Studio", "Widgets", "Events"]
categories: ["Hardware", "ESP32"]
---

In the previous post, we brought our interface to life by enabling touch and responding to a button event. Now that we know how to capture user interactions, the next logical step is to make those interactions modify the interface itself.

In this tutorial, we will build a slightly more complex and much more practical application: a **digital counter**. We will create a screen with a number and two buttons, one to increment and one to decrement that number. This example is fundamental because it teaches us how to read from and write to screen widgets, an essential skill for any UI project, whether it's displaying sensor data, adjusting settings, or any other dynamic application.

### Step 1: Designing the Counter Interface in SquareLine Studio

Let's start by creating the visual layout of our application.

1.  Open SquareLine Studio and create a new project (or continue an existing one).
2.  On the main screen (`Screen1`), add the following widgets:
    * A **Label** in the center. In the "Inspector" panel, change its initial text to `0`. Give it an easy-to-remember name, like `labelCounter`.
    * A **Button** to the left of the label. Add a **Label** inside it with the text `-`.
    * A **Button** to the right of the label. Add a **Label** inside it with the text `+`.
3.  **Configuring Events:** Now, let's link our buttons to functions we will write later.
    * Select the `-` button. In the **Events** section, add an event with `Trigger: CLICKED` and `Function: decrementCounter`.
    * Select the `+` button. In the **Events** section, add an event with `Trigger: CLICKED` and `Function: incrementCounter`.
4.  Export the UI files. Remember to place all generated files in the same folder as your `.ino` sketch.

### Step 2: The Counter Logic in `ui_events.cpp`

All the "intelligence" of our application will reside in the `ui_events.cpp` file. This is where we will declare our counter, create the functions that the buttons call, and, most importantly, update the label's text on the screen.

Don't forget to **rename `ui_events.c` to `ui_events.cpp`** to be able to use Arduino functions.

```cpp
/*
 * ui_events.cpp
 * Logic for the counter application.
 */
#include <Arduino.h>
#include "ui.h"

// Variable to store the value of our counter
static int32_t counter = 0;

// Function to update the label's text on the screen
void updateLabel() {
    // Buffer to format the number as text
    char buffer[12]; 
    sprintf(buffer, "%d", counter); // Converts the int to a string

    // The magic happens here!
    // SquareLine Studio creates a global variable for each widget.
    // We use the lv_label_set_text function to change our label's text.
    lv_label_set_text(ui_labelCounter, buffer);
}

// Function called by the increment button
void incrementCounter(lv_event_t * e)
{
    counter++;
    updateLabel();
    Serial.print("Counter incremented to: ");
    Serial.println(counter);
}

// Function called by the decrement button
void decrementCounter(lv_event_t * e)
{
    counter--;
    updateLabel();
    Serial.print("Counter decremented to: ");
    Serial.println(counter);
}
```
**Understanding the Code:**

* **`static int32_t counter = 0;`**: We create a static variable to hold the counter's value. It needs to be static (or global) so that its value is preserved between function calls.
* **`lv_label_set_text(ui_labelCounter, buffer);`**: This is the most important line. SquareLine Studio automatically declares pointers to all your widgets in the `ui.h` file. `ui_labelCounter` is our label. The `lv_label_set_text` function is a native LVGL function that allows us to change the text of a label widget at any time.

### Step 3: The Main Code (No Changes!)

The best part is that our `.ino` file **needs no changes**. It is already set up to initialize the hardware, LVGL, and touch. All the new logic is contained within the UI files. For completeness, here is the base code we are using:

```cpp
/*
 * EXAMPLE 4: INTERACTIVE COUNTER
 * The main code does not change. All logic is in the UI files.
 */

#define LGFX_USE_V1
#include <LovyanGFX.hpp>
#include <lvgl.h>
#include "ui.h"

// --- Hardware Configuration for LovyanGFX ---
class LGFX : public lgfx::LGFX_Device {
  lgfx::Panel_ST7796  _panel_instance;
  lgfx::Bus_Parallel8 _bus_instance;
  lgfx::Light_PWM     _light_instance;
  lgfx::Touch_FT5x06  _touch_instance;
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

// --- Driver and Buffer Setup ---
LGFX gfx; 
static const uint16_t screenWidth  = 480;
static const uint16_t screenHeight = 320;
static lv_disp_draw_buf_t draw_buf;
static lv_color_t buf1[screenWidth * screenHeight / 10];

// --- LVGL Bridge Functions ---
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

// --- Main Application ---
void setup() {
    Serial.begin(115200);
    Serial.println("Starting Example 4: Interactive Counter");
    
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
    Serial.println("Setup finished.");
}

void loop() {
    lv_timer_handler();
    lv_tick_inc(5);
    delay(5);
}
```
### Conclusion

Congratulations! You now have a fully dynamic application. When you touch the buttons, the value on the screen updates in real-time, and you can see the confirmation in the Serial Monitor.

You've learned the most powerful concept of UI programming with LVGL: how to **access and manipulate widgets dynamically** from your event code. With this technique, you can create anything from digital thermometers that display sensor data to complex control panels.

In our next and final practical post, we will explore how to manage multiple screens and navigate between them.

See you then!