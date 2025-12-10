---
title: "Data Persistence in Flutter with SharedPreferences"
date: 2025-12-10T11:18:38-03:00
draft: false
tags: ["Flutter", "SharedPreferences", "Persistence", "Local Storage", "Dart"]
categories: ["Flutter"]
---

When developing applications, a common need is to preserve simple data between user sessions. SharedPreferences is the ideal solution for storing small amounts of data in key-value format, such as user preferences, application settings, or simple states that need to persist.

In this tutorial, we'll implement data persistence in Flutter's default counter application, making the counter value persist even after closing and reopening the app.

### What is SharedPreferences?

SharedPreferences is a cross-platform API that allows you to store primitive data types persistently on the device. It works like a key-value dictionary, where you can save and retrieve data of basic types such as strings, integers, booleans, and string lists.

**Platform Implementation:**

* **Android:** Uses native `SharedPreferences`, storing data in XML files in the application's private directory.
* **iOS/macOS:** Uses `NSUserDefaults`, storing data in plist files.
* **Web:** Uses browser's `localStorage`. **Important:** data is tied to the specific domain and browser. If the user clears browser cache/cookies, the data will be lost. If accessing the same site in another browser or in incognito mode, they won't have access to previously saved data. For web applications that need more reliable persistence, consider using backend with authentication.
* **Windows:** Stores data in JSON files in the user's configuration directory (typically `C:\Users\{username}\AppData\Roaming\{company_name}\{app_name}`).
* **Linux:** Stores data in JSON files in the user's configuration directory (typically `~/.local/share/{app_name}` or `~/.config/{app_name}`).

**When to Use SharedPreferences:**

SharedPreferences is appropriate for:
* Configuration preferences (theme, language, notifications)
* Simple application states
* Onboarding data
* Small amounts of non-sensitive data

**When NOT to Use:**

* Sensitive data (passwords, tokens) - for this use `flutter_secure_storage` which is more secure
* Large data volumes - use databases like SQLite
* Complex structured data - consider Hive or ObjectBox
* Data requiring complex queries

### Step 1: Add the Dependency

First, add the `shared_preferences` package to your project. In the terminal, at the project root, run:

```bash
flutter pub add shared_preferences
```

This adds the necessary line to your `pubspec.yaml` and downloads the package.

### Step 2: Code Structure

We'll modify Flutter's default counter application (the template created when you start a new project) to include persistence, meaning if you stop the program and start it again, the counter value will persist. The logic is simple:

1. When the app starts, load the saved value
2. Whenever the counter is incremented, save the new value
3. The value will remain available even after restarting the application

### Step 3: Complete Implementation

Replace the contents of the `lib/main.dart` file with the code below:

```dart
import 'package:flutter/material.dart';
import 'package:shared_preferences/shared_preferences.dart'; // NEW: package import

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Flutter Demo',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.deepPurple),
        useMaterial3: true,
      ),
      home: const MyHomePage(title: 'Flutter Demo Home Page'),
    );
  }
}

class MyHomePage extends StatefulWidget {
  const MyHomePage({super.key, required this.title});

  final String title;

  @override
  State<MyHomePage> createState() => _MyHomePageState();
}

class _MyHomePageState extends State<MyHomePage> {
  int _counter = 0;

  // NEW: Key to save/retrieve the value from SharedPreferences
  static const String _counterKey = 'counter';

  // NEW: initState method to load the saved value when the app starts
  @override
  void initState() {
    super.initState();
    _loadCounter();
  }

  // NEW: Loads the counter value from SharedPreferences
  Future<void> _loadCounter() async {
    final prefs = await SharedPreferences.getInstance();
    setState(() {
      _counter = prefs.getInt(_counterKey) ?? 0;
    });
  }

  // NEW: Saves the counter value to SharedPreferences
  Future<void> _saveCounter() async {
    final prefs = await SharedPreferences.getInstance();
    await prefs.setInt(_counterKey, _counter);
  }

  void _incrementCounter() {
    setState(() {
      _counter++;
    });
    _saveCounter(); // NEW: saves the value after incrementing
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        backgroundColor: Theme.of(context).colorScheme.inversePrimary,
        title: Text(widget.title),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: <Widget>[
            const Text(
              'You have pushed the button this many times:',
            ),
            Text(
              '$_counter',
              style: Theme.of(context).textTheme.headlineMedium,
            ),
          ],
        ),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: _incrementCounter,
        tooltip: 'Increment',
        child: const Icon(Icons.add),
      ),
    );
  }
}
```

### Understanding the Code

Let's analyze only the parts that were added to Flutter's standard code:

**1. Import and Constant:**
```dart
import 'package:shared_preferences/shared_preferences.dart'; // NEW
static const String _counterKey = 'counter'; // NEW
```
We import the package and define a constant for the key. The **key** works as a unique identifier (similar to a variable name) that will be used to store and retrieve the counter value. For example, when we save `prefs.setInt('counter', 5)`, we're saying "save the value 5 with the identifier 'counter'". Later, when we want to retrieve this value, we use the same identifier: `prefs.getInt('counter')`. 

**Why use a constant?** Although it's possible to write `'counter'` directly everywhere, using a constant (`_counterKey`) brings important benefits: if you need to change the key name, just change it in one place; avoids typos (if you accidentally write `'conter'` somewhere, the code won't work); and facilitates maintenance in larger projects where you might have dozens of different keys.

**2. initState Method:**
```dart
@override
void initState() {
  super.initState();
  _loadCounter(); // NEW
}
```
The `initState()` is called automatically when the widget is created. In this case, the widget is the `MyHomePage` page. Here we load the saved value as soon as the page is initialized.

**3. Loading Data:**
```dart
Future<void> _loadCounter() async {
  final prefs = await SharedPreferences.getInstance();
  setState(() {
    _counter = prefs.getInt(_counterKey) ?? 0;
  });
}
```
Here we use `Future` and `async` because accessing SharedPreferences involves disk/storage reading, which is an **asynchronous** operation (meaning it takes time to complete). The `async/await` allows Flutter to perform other tasks (like keeping the interface responsive) while waiting for the operation to finish. When we use `await`, we're saying: "pause this method here and continue when `getInstance()` finishes, but don't freeze the rest of the application". We get the SharedPreferences instance and load the saved value. The `??` operator provides `0` as default value if the key doesn't exist.

**4. Saving Data:**
```dart
Future<void> _saveCounter() async {
  final prefs = await SharedPreferences.getInstance();
  await prefs.setInt(_counterKey, _counter);
}
```
Just like loading, we use `Future` and `async` because writing data to storage is an asynchronous operation. The `async/await` allows the application to continue working normally while data is saved in the background. We save the current counter value. This method is called whenever we increment the counter.

**5. Call in Increment:**
```dart
void _incrementCounter() {
  setState(() {
    _counter++;
  });
  _saveCounter(); // NEW: only this line was added
}
```
After incrementing, we automatically save the new value.

**Available Methods in SharedPreferences:**

SharedPreferences offers methods for different data types:
* `setInt()` / `getInt()` - integers
* `setDouble()` / `getDouble()` - decimal numbers
* `setString()` / `getString()` - text
* `setBool()` / `getBool()` - booleans
* `setStringList()` / `getStringList()` - string lists

All `set` methods return `Future<bool>`, and all `get` methods can return `null` if the key doesn't exist.

### Important Considerations

**Performance:**
* SharedPreferences keeps data in memory after first load, making subsequent reads extremely fast.
* Write operations are asynchronous and don't block the UI.

**Size Limitations:**
* There's no official limit, but it's recommended to store only a few kilobytes.
* For larger data, consider alternatives like SQLite or Hive.

**Security:**
* Data is NOT encrypted.
* Don't store sensitive information like passwords or authentication tokens.
* For sensitive data, use the `flutter_secure_storage` package.

**Data Types:**
* Only primitive types are supported.
* For complex objects, you'll need to serialize them (usually to JSON) before saving as a string.

### Testing the Application

1. **Run the application:**
   ```bash
   flutter run
   ```

2. Increment the counter several times.

3. Completely close the application (not just minimize).

4. Reopen the application.

5. The counter should display the last saved value, not zero.

### Best Practices

**1. Use constants for keys:**
```dart
class PrefsKeys {
  static const String counter = 'counter';
  static const String userName = 'user_name';
  static const String isDarkMode = 'is_dark_mode';
}
```
Creating a dedicated class to store all your application's keys centralizes configuration and makes maintenance much easier. This way, if you need to change a key name, just change it in one place. Additionally, you have a clear view of all data being stored in SharedPreferences. To use these constants, simply access them through the class: `prefs.setInt(PrefsKeys.counter, 5)` or `prefs.getString(PrefsKeys.userName) ?? ''`.

**2. Create a utility class:**
```dart
class PrefsService {
  static SharedPreferences? _prefs;
  
  static Future<void> init() async {
    _prefs ??= await SharedPreferences.getInstance();
  }
  
  static int getCounter() => _prefs?.getInt('counter') ?? 0;
  static Future<bool> setCounter(int value) async => 
      await _prefs!.setInt('counter', value);
}
```
A service class encapsulates all SharedPreferences access logic in one place. This makes code more organized, as you don't need to keep getting the SharedPreferences instance in multiple different places. Just call `PrefsService.getCounter()` or `PrefsService.setCounter(value)` from anywhere in your code.

**3. Handle errors properly:**
```dart
Future<void> _saveCounter() async {
  try {
    _prefs ??= await SharedPreferences.getInstance();
    final success = await _prefs!.setInt(_counterKey, _counter);
    if (!success) {
      // Handle the failure
      print('Error saving counter');
    }
  } catch (e) {
    print('Exception while saving: $e');
  }
}
```
Disk operations can fail for various reasons (insufficient space, permissions, etc.). Using `try-catch` allows your application to handle these situations gracefully, showing a message to the user or trying again, instead of simply crashing.

**4. Initialize in main() when appropriate:**
```dart
void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await PrefsService.init();
  runApp(const MyApp());
}
```
If you created a utility class like in example 2, initializing SharedPreferences in `main()` ensures it's ready before any screen is loaded. This avoids delays the first time you need to access the data. The `WidgetsFlutterBinding.ensureInitialized()` is necessary to use plugins before calling `runApp()`.

### Alternatives to SharedPreferences

Depending on your project needs, consider:

* **flutter_secure_storage:** For sensitive data requiring encryption
* **Hive:** Fast and efficient NoSQL database, great for complex objects
* **sqflite:** Complete SQL database for complex queries
* **get_storage:** Faster alternative to SharedPreferences
* **ObjectBox:** High-performance database for object models

### Conclusion

SharedPreferences is a fundamental tool in Flutter development, offering a simple and efficient way to persist basic data. While it has limitations, it's perfect for use cases like user preferences, application settings, and simple states.

We implemented persistence in Flutter's default counter, demonstrating fundamental concepts: asynchronous initialization, loading existing data, and saving after state changes. This foundation can be expanded to more complex scenarios in your applications.

Always remember to choose the right tool for each use case: SharedPreferences for simple data, secure storage solutions for sensitive information, and complete databases for complex structured data.