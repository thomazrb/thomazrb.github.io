---
title: "Using an RC Radio as a Joystick in MSFS 2024 through GeForce NOW"
date: 2026-08-03T07:30:00-03:00
draft: false
tags: ["GeForce NOW", "MSFS 2024", "EdgeTX", "Joystick", "Flight Simulator", "macOS", "Windows"]
categories: ["Gaming"]
---

An RC radio (the transmitter used for model airplanes) is, without exaggeration, one of the best home-made flight controllers there is: the throttle has no spring and stays wherever you leave it, while the sticks self-center. It is the ergonomics of a real cockpit. The challenge is getting it to work in **Microsoft Flight Simulator 2024 running through GeForce NOW**, and that is where most people get stuck.

**Tested with:** a Flysky ProArt PA-01 (running EdgeTX) on a Mac with an M4 chip (Apple Silicon), MSFS 2024 via GeForce NOW. But the method works with any EdgeTX or OpenTX radio, and on native Windows too.

### Why It Doesn't Work Out of the Box

GeForce NOW only forwards controllers that use the **XInput** standard (the one Xbox gamepads use). An RC radio connected over USB shows up to the system as a **generic joystick** (DirectInput, with no standard mapping). As a result, the browser or the app can see the radio, but GeForce NOW ignores it, and inside MSFS the device list shows only keyboard and mouse.

The solution is to make the radio **pretend to be an Xbox controller** (standard XInput). On **Windows** this is straightforward. On **Mac** there is one extra step (a Windows virtual machine), because macOS does not let you create a virtual controller.

### Prerequisites

1. **A radio running EdgeTX or OpenTX** with USB Joystick mode (nearly all of them have it).
2. **A USB cable.**
3. **A GeForce NOW account** and **MSFS 2024** in your library.
4. **ViGEmBus and XOutput** (both free).
5. **Mac only:** a **Windows 11 virtual machine** (Parallels, UTM, or VMware Fusion).

### Step 1: Preparing the Radio (Windows and Mac)

Create a **clean model** on the radio (in EdgeTX: New Model, then Blank Model).

**Important:** if you reuse a model with mixes (flying wing, tank mix, and so on), the axes arrive mixed and at half travel. A blank model sends each stick straight to its own channel, unmixed: CH1 = aileron, CH2 = elevator, CH3 = throttle, CH4 = rudder.

Then set the radio to **USB Joystick** mode (not "USB Storage") and test it at **hardwaretester.com/gamepad**: the radio should appear with its axes responding to movement.

### Step 2: The Virtual Machine (Mac only)

If you are on **native Windows, skip to Step 3.**

On **Mac** (Apple Silicon), macOS does not let you create a virtual controller, so we run the solution inside a Windows virtual machine:

1. Create a **Windows 11 VM (ARM)**.
2. Set up **USB passthrough** of the radio into the VM (it needs to "see" the radio in the gamepad tester).
3. **From here on, do everything inside the VM.**

Yes, this is streaming running inside a virtual machine. In practice, the latency is imperceptible, and you can fly comfortably.

### Step 3: Turning the Radio into a Virtual Xbox Controller

1. Install **ViGEmBus**, the driver that creates the virtual controller.

   **Note:** on Windows ARM (the Mac VM), download the installer that includes the **arm64** build (`ViGEmBus_..._x64_x86_arm64.exe`).

2. Install and open **XOutput**. It will list the radio under DirectInput.

3. Click **Add controller**, then **Edit**, and map the radio axes to the Xbox axes:
   - Left Stick X ← aileron
   - Left Stick Y ← elevator
   - Right Stick Y ← throttle
   - Right Stick X ← rudder

4. Click **Start**.

5. Check **hardwaretester.com/gamepad** again: an **"Xbox 360 Controller"** now appears with **standard** mapping. That is the one GeForce NOW accepts.

### Step 4: Mapping in MSFS 2024

Open GeForce NOW, launch MSFS, and go to **Settings, then Controls** with the **"Xbox 360 Controller"** selected. When you start configuring, MSFS creates a profile for you. Just accept it.

**Aileron and elevator are already mapped** to the sticks by default, so you don't need to touch them. You only need to configure the **throttle** and the **rudder**:

- Throttle → **THROTTLE AXIS**
- Rudder → **RUDDER AXIS** (this is what steers the plane on the ground)

**Clear the conflicts before setting the final mapping.** Here is the detail that saves hours: the right stick also controls the camera, so the throttle and the rudder conflict with it. When you select the **full axis** (the stick icon with the up-and-down arrow), MSFS shows a few conflicts. But there are other **silent** ones that only appear when you point each **separate direction**: the up arrow and the down arrow individually.

So run the axis through all **three forms** (the full axis, the up direction, and the down direction), clearing the conflicts each one reveals. Only after removing all three do you assign the correct mapping. Do the same for the rudder (on the horizontal stick: the full axis, the left direction, and the right direction). If you clear only the full axis, a hidden conflict remains and the camera keeps moving along with your input.

Finally:

- Lower the throttle **dead zone** to zero (the default is around 0.14 and creates a "hole" in the middle of the travel).
- If the **throttle is inverted** (pushing forward cuts the power), enable the **invert axis** option.

### Conclusion

With these steps, the RC radio turns into a Cessna cockpit: an analog throttle that stays at the exact power you set, sticks that self-center, and a rudder that steers the plane on the runway. On Windows it is three steps; on Mac, the same three inside a virtual machine. Happy flying!
