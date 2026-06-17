---
title: "How to Lock Screen Orientation in Flutter"
date: 2026-06-17T13:30:00-03:00
draft: false
tags: ["Flutter", "Dart", "SystemChrome", "Orientation"]
categories: ["Flutter"]
---

Many developers who want to lock screen orientation in Flutter go straight to `AndroidManifest.xml` and add `android:screenOrientation="portrait"`. That works, but there's a better way.

### The Dart approach: `SystemChrome`

Add the import and configure the orientations at the beginning of `main()`, before calling `runApp`:

```dart
import 'package:flutter/services.dart'; // ← required import

void main() {
  WidgetsFlutterBinding.ensureInitialized(); // needed before any system call
  SystemChrome.setPreferredOrientations([
    DeviceOrientation.portraitUp,
    DeviceOrientation.portraitDown,
  ]);
  runApp(const MyApp());
}
```

`WidgetsFlutterBinding.ensureInitialized()` is required to make sure Flutter is ready before any system call. Without it, `setPreferredOrientations` may not work correctly.

### Variants

**Portrait only (no upside-down):**
```dart
SystemChrome.setPreferredOrientations([
  DeviceOrientation.portraitUp,
]);
```

**Landscape only:**
```dart
SystemChrome.setPreferredOrientations([
  DeviceOrientation.landscapeLeft,
  DeviceOrientation.landscapeRight,
]);
```

**Allow all orientations:**
```dart
SystemChrome.setPreferredOrientations(DeviceOrientation.values);
```

### Why not use AndroidManifest?

| | AndroidManifest | SystemChrome (Dart) |
|---|---|---|
| When it acts | Compile-time | Runtime |
| Platforms | Android only | Android **and** iOS |
| Flexibility | App-wide, fixed | Can change per screen |

The real advantage of `SystemChrome` shows when you need different behavior per screen, for example: lock portrait app-wide but unlock landscape for a specific video player or minigame. Just call `setPreferredOrientations` again on that screen.

For locking the whole app to portrait, both approaches work. But the Dart way handles it cleanly and covers iOS for free.
