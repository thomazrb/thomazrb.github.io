---
title: "Complete Guide to the WT32-SC01 Plus (Part 6 of 6): Navigating Between Screens"
date: 2025-09-04T10:03:10-03:00
draft: false
tags: ["ESP32-S3", "WT32-SC01 Plus", "LovyanGFX", "LVGL", "SquareLine Studio", "Screens", "UI"]
categories: ["Hardware", "ESP32"]
---

We've reached the end of our practical journey with the **WT32-SC01 Plus**! Throughout the last few posts, we've learned how to set up the environment, draw on the screen, enable touch, and manipulate widgets. Now, we'll combine all this knowledge to build the structure of a real application, which almost always involves more than one screen.

In this final practical tutorial, we will learn how to create and manage multiple screens in **SquareLine Studio** and navigate between them using button events. We will create a simple two-screen application, where a button on each screen takes us to the other, demonstrating the basic navigation flow essential for any complex project, such as settings menus, information pages, etc.

### Step 1: Creating the Screens in SquareLine Studio

First, let's design our interface with two distinct screens.

1. Open SquareLine Studio.
2. In the "Screens" panel in the upper left corner, you'll see `Screen1`. Click the small `+` icon to add a new screen. It will automatically be named `Screen2`.
3. **On `Screen1`:**
   * Add a **Label** with the text "Main Screen".
   * Add a **Button** with the text "Go to Screen 2".
   * Select the button and, in the **Events** section, create an event with `Trigger: CLICKED` and `Function: goToScreen2`.
4. **On `Screen2`:**
   * Select `Screen2` in the "Screens" panel to edit it.
   * Add a **Label** with the text "Secondary Screen".
   * Add a **Button** with the text "Back to Screen 1".
   * Select this button and create an event with `Trigger: CLICKED` and `Function: backToScreen1`.
5. Export the UI files and place them in your Arduino project folder.

### Step 2: The Navigation Logic in `ui_events.cpp`

The magic of screen switching happens in the `ui_events.cpp` file, where we will implement the two functions we just defined in SquareLine.

Remember to **rename `ui_events.c` to `ui_events.cpp`**.

```cpp
/*
 * ui_events.cpp
 * Logic for screen navigation.
 */
#include <Arduino.h>
#include "ui.h"

// Function called by the button on Screen 1
void goToScreen2(lv_event_t * e)
{
    Serial.println("Navigating to Screen 2...");
    // LVGL function to load a new screen with an animation.
    // Parameters: (target_screen, animation_type, duration, delay, delete_old)
    lv_scr_load_anim(ui_Screen2, LV_SCR_LOAD_ANIM_MOVE_LEFT, 500, 0, false);
}

// Function called by the button on Screen 2
void backToScreen1(lv_event_t * e)
{
    Serial.println("Going back to Screen 1...");
    lv_scr_load_anim(ui_Screen1, LV_SCR_LOAD_ANIM_MOVE_RIGHT, 500, 0, false);
}
```
**Understanding the Code:**

* **`lv_scr_load_anim(...)`**: This is the main LVGL function for navigation.
    * The first parameter (`ui_Screen2` or `ui_Screen1`) is the pointer to the screen object we want to load. Just like widgets, SquareLine Studio creates a global variable for each screen.
    * `LV_SCR_LOAD_ANIM_MOVE_LEFT`: This is the animation type. LVGL offers several, such as `FADE_ON`, `OVER_TOP`, etc. `MOVE_RIGHT` provides the reverse animation.
    * `500`: The duration of the animation in milliseconds.
    * `0`: A delay before the animation starts.
    * `false`: Tells LVGL **not** to delete the previous screen from memory. This is useful for quick back-and-forth navigation. In projects with many screens and limited RAM, you might set this to `true` to free up memory.

### Step 3: The Main Code (Unchanged)

Once again, our `.ino` file requires no changes. It serves as the solid foundation that runs our application, regardless of how many screens or what event logic we have.

Here is the complete code to ensure your project works perfectly.

```cpp
/*
 * EXAMPLE 5: NAVIGATING BETWEEN SCREENS
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

// --- Bridge Functions for LVGL ---
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
    Serial.println("Starting Example 5: Navigating Between Screens");
    
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
### Series Conclusion

Congratulations! You have reached the end of this tutorial series for the WT32-SC01 Plus. Throughout these posts, you have learned to:

* Set up the development environment from scratch.
* Control the display directly with `LovyanGFX`.
* Integrate the powerful `LVGL` library with the `SquareLine Studio` visual tool.
* Enable touch and respond to button events.
* Manipulate widgets to create dynamic interfaces.
* Manage and navigate between multiple screens.

With this solid foundation, you are more than prepared to create your own complex projects with professional interfaces. The WT32-SC01 Plus is an incredibly capable platform, and you now have the tools to explore its full potential.

I hope this series has been helpful. Happy creating!