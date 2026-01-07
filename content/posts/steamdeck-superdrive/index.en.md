---
title: "How to Use Apple's SuperDrive on Steam Deck"
date: 2026-01-06T19:01:37-03:00
draft: false
tags: ["Steam Deck", "SuperDrive", "Linux", "Hardware"]
categories: ["Steam Deck"]
---

The Steam Deck is a versatile machine that goes far beyond digital gaming. If you own an Apple USB SuperDrive and want to use it on your Steam Deck to read DVDs, this guide will show you how to set everything up quickly and easily.

### Prerequisites

Before you begin, you'll need:

1. **A Steam Deck with a USB dock or hub:** The SuperDrive has a USB-A connection (the traditional USB, not USB-C) and uses the USB 2.0 standard, so you'll need a dock or hub with USB ports. The drive works well when connected directly to USB ports on the dock, whether they're USB 2.0 or USB 3.0 (the blue ones).

2. **Distrobox Ubuntu:** This tutorial uses a distrobox with Ubuntu to facilitate the process. The advantage of using distrobox is that you can install packages and make configurations without needing to break the Steam Deck's read-only system, keeping everything organized and secure. If you don't have one configured yet, you can easily install it through the **BoxBuddy** app available in the Steam Deck's Discover store. (In a future tutorial, I might show the detailed installation process.)

### Step 1: Installing the Required Tools

First, we need to ensure that the `sg3_utils` package is installed in your Ubuntu distrobox. This package contains the necessary tools to send SCSI commands directly to the drive.

Open your Ubuntu distrobox terminal and run:

```bash
sudo apt update
sudo apt install sg3-utils
```

### Step 2: Identifying the Drive

Now we need to discover which device the SuperDrive was mounted on. In most cases, it will appear as `/dev/sr0`. To check, run:

```bash
ls /dev/sr*
```

If you see `/dev/sr0` listed, you're all set! If the drive was mounted with a different number (like `/dev/sr1`, `/dev/sr2`, etc.), note the exact device name that appeared, as you'll need to use it in the next steps.

### Step 3: Setting Permissions

By default, the storage device requires special access permissions. We need to grant these permissions for the drive.

In the Steam Deck's Konsole (outside the distrobox), run:

```bash
sudo chmod 666 /dev/sr0
```

**Important:** This command needs to be executed on the Steam Deck's main system (SteamOS), not inside the distrobox.

### Step 4: Initializing the SuperDrive

Now comes the magic! Apple's SuperDrive needs to receive a special initialization command to start working. Enter your Ubuntu distrobox again and run:

```bash
sudo sg_raw /dev/sr0 EA 00 00 00 00 00 01
```

If everything went well, you'll see the following message:

```
NVMe Result=0x0
```

### Step 5: Setting Up VLC for DVD Playback

Now that the SuperDrive is working, let's configure VLC to play your DVDs. VLC can be easily installed through the Steam Deck's Discover store.

#### Installing VLC

1. Open **Discover** (the Steam Deck app store in desktop mode)
2. Search for **VLC Media Player**
3. Click **Install**

#### Support for Commercial DVDs

To play protected commercial DVDs, you'll need to install an additional component that provides the `libdvdcss` library:

1. Still in the **Discover** store, search for **"Commercial DVD support for HandBrake"**
2. Install this package

This package adds the necessary support to decode commercial DVDs with CSS (Content Scramble System) protection, allowing VLC to play virtually any DVD.

### Ready to Watch!

With everything configured, just insert a DVD into the SuperDrive and open VLC. The player will automatically detect the disc and you'll be able to watch your movies directly on the Steam Deck!

### Important Notes

- **USB Ports:** The SuperDrive uses standard USB-A (USB 2.0) and works on any USB port of a dock or hub when connected directly.
- **Temporary Permissions:** The permissions granted with `chmod` are temporary and will be reset after restarting the system. You'll need to run the permission commands again if you restart the Steam Deck.
- **Reinitialization Required:** Whenever you disconnect and reconnect the SuperDrive, you'll need to run the `sg_raw` command from Step 4 again to initialize it (in addition to the permissions from Step 3, if the system has been restarted).
- **Compatibility:** This method works specifically with Apple's SuperDrive. Other USB drives may work differently or may not require these special commands.

### Conclusion

With just a few commands, we've transformed the Steam Deck into a functional DVD player using Apple's SuperDrive. It's a great demonstration of the flexibility of the Linux system running underneath SteamOS and the possibilities that distrobox offers to expand the portable console's functionality.