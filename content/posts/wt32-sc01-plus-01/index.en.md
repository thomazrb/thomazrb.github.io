---
title: "Complete Guide to the WT32-SC01 Plus (Part 1 of 6): Introduction and First Steps"
date: 2025-08-25T16:17:01-03:00
draft: false
tags: ["ESP32-S3", "WT32-SC01 Plus", "Display", "LVGL", "IoT", "SquareLine Studio"]
categories: ["Hardware", "ESP32"]
---

For those who develop projects with graphical interfaces, the complexity of integrating displays, touch controllers, and microcontrollers can be a major challenge. The **WT32-SC01 Plus** board emerges as an interesting alternative to simplify this process.

This is a development board that integrates the ESP32-S3 processor with a 3.5-inch capacitive touch screen. This "all-in-one" approach is ideal for prototyping and developing user interfaces (UI), such as automation control panels, small interactive consoles, and other devices that require visual interaction with the user.

![WT32-SC01 Plus Board](sc01plus.png)

In this 6-part series, we will explore the full potential of this board, from the initial setup to creating complex and functional interfaces. This first article is our starting point: we will get to know the hardware, understand its features, and prepare the ground for our future projects.

### An Integrated Approach

The main feature of the WT32-SC01 Plus is its integrated nature. Unlike the traditional assembly, which requires purchasing and connecting a separate ESP32, display, and touch controller, this board already combines all the components. This speeds up development and significantly reduces the chances of connection errors or hardware compatibility issues.

### Key Technical Specifications

Let's take a look at the board's specifications:

* **Processor:** **ESP32-S3** Dual-Core Xtensa LX7. A modern and powerful chip with Wi-Fi, Bluetooth 5 (LE), AI acceleration, and low power consumption.
* **Display:** A **3.5-inch** screen with a resolution of **480x320 pixels**. Communication is handled via an **8-bit parallel interface**, which is significantly faster than the common SPI communication, resulting in smoother animations.
* **Touch:** **Capacitive FT6336U** touch controller. It's the same type of technology found in smartphones, offering a much more precise and smooth touch response than older resistive panels.
* **Memory:** It comes equipped with 16MB of Flash and 8MB of PSRAM, essential for storing images, fonts, and for the proper functioning of graphics libraries like LVGL.
* **Connectivity:** In addition to Wi-Fi and Bluetooth, the board has a USB-C connector for programming and power, a MicroSD card slot for expandable storage, and connectors for GPIO expansion.
* **Audio:** It includes an audio amplifier and a speaker connector, allowing for the playback of sounds and audio alerts directly on the board.

### Points to Consider

1.  **Accelerated Development:** Being an integrated solution, the time between taking the board out of the box and starting to program the interface is minimal. The hardware complexity has already been solved.
2.  **Cost-Effective:** For many projects, acquiring the integrated board can be more economical than purchasing high-quality components separately.
3.  **Optimized for Graphical Interfaces:** The board's architecture, with its fast processor, parallel display interface, and generous PSRAM, is especially well-suited for graphics libraries like **LVGL**, enabling the creation of fluid and responsive interfaces, particularly when combined with tools like **SquareLine Studio**.

### Where to Find It?

The board is available from various online stores. For reference, here is the AliExpress seller from whom I purchased mine (be careful not to buy the debug tool or the old version; the correct board is the WT32-SC01 PLUS):

* [**Purchase Link - WT32-SC01 Plus on AliExpress**](https://s.click.aliexpress.com/e/_oCKjXXt)

### Next Steps

Now that we are familiar with the hardware, we are ready to get our hands dirty. In the next post in this series, we will set up our development environment, install the necessary libraries, and write our first piece of code: the classic "Hello World!" will appear on our screen.

See you then!