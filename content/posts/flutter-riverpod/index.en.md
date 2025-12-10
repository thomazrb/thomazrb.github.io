---
title: "Riverpod: Modern and Simple State Management in Flutter"
date: 2025-12-10T15:30:00-03:00
draft: false
tags: ["Flutter", "Riverpod", "State Management", "Dart"]
categories: ["Flutter"]
---

When developing Flutter applications, we frequently need to handle information that changes over time: the number displayed on a counter, whether the user is logged in or not, items in a shopping cart, the current app theme. This information that can change and needs to be shared across different parts of the application is what we call **state**. Managing state means controlling this information in an organized way: where it's stored, how it's modified, and how widgets are notified when it changes to update the interface.

Managing state in a Flutter application can seem complicated at first, but with **Riverpod**, this task becomes much simpler and more powerful. Riverpod is an evolution of Provider, created by the same developer (Remi Rousselet), bringing significant improvements like not depending on `BuildContext` and better testability.

In this guide, we'll build a simple counter application to demonstrate how Riverpod works in practice. You'll learn to create providers, consume their values, and react to state changes in a clean and efficient way.

**Versions used in this tutorial:**
- Flutter: 3.38.3
- flutter_riverpod: 3.0.3

### Why Use Riverpod?

* **No Context:** You can access providers from anywhere, without needing `BuildContext` as was necessary in Provider.
* **Compile-Time Safety:** Type errors are detected during compilation, not at runtime.
* **Testable:** Much easier to test than other state management solutions.
* **Less Boilerplate:** Cleaner and more direct code than other solutions like BLoC (Business Logic Component), another very popular state management pattern in Flutter. It's great for large and complex apps, but can be "too much" for simple projects.
* **Evolution of Provider:** Created by the same developer, fixing design problems from the original Provider.

### Step 1: Creating the Project and Adding Dependencies

Let's start by creating a new Flutter project and adding Riverpod.

1. **Create the project:**
   ```bash
   flutter create riverpod_example
   ```

2. **Navigate to the project folder:**
   ```bash
   cd riverpod_example
   ```

3. **Add Riverpod:** We'll use `flutter_riverpod`, which is the Flutter-specific version.
   ```bash
   flutter pub add flutter_riverpod
   ```

### Step 2: Understanding the Basic Concepts

Before diving into the code, let's understand Riverpod's main concepts in a clear way:

* **Notifier:** It's a special class that works like a "radio station". When the value changes, it broadcasts updates to all widgets that are "tuned in" to it. Think of it as a radio station that transmits updates to all tuned devices.

* **NotifierProvider:** It's the widget that makes the `Notifier` available to the entire application. It's like installing the radio station's antenna - without it, nobody can tune into the signal. We usually declare it in a separate providers file.

* **ProviderScope:** A widget that must wrap your entire application in `main.dart`. It's the "general manager" that takes care of all providers. Without it, nothing works - it's like the main transmission tower that coordinates all antennas.

* **ConsumerWidget:** A special widget that "listens" to provider broadcasts. Every time the `Notifier` sends an update, the `ConsumerWidget` automatically rebuilds. It's like a radio that automatically updates the information on the screen when it receives a new signal. **Important:** It's inside the `ConsumerWidget` that we have access to `ref`, which allows us to use `ref.watch` and `ref.read`. If you use a normal `StatelessWidget`, you won't have access to `ref`.

* **ref.watch():** A method that tunes into the provider's "frequency" and keeps listening to updates. When something changes, the widget is automatically rebuilt. Use inside the `build()` method when you want the screen to update automatically. **Only works inside a `ConsumerWidget`!**

* **ref.read():** This method just "takes a quick look" at the current value without listening to the broadcast. It's like quickly checking which song is playing without turning on the radio. Use inside callbacks (like `onPressed` of a button or `onChanged` of a TextField) when you just want to execute a one-time action. **Also only works inside a `ConsumerWidget`!**

### Step 3: Project Structure

Let's create a simple and organized structure. Our app will have two pages to demonstrate how Riverpod shares state between different screens:

```
lib/
  ├── main.dart
  ├── providers/
  │   └── counter_provider.dart
  └── pages/
      ├── counter_page.dart      # Page with the text field
      └── display_page.dart      # Page that only displays the value
```

### Step 4: Creating the Counter Provider

#### `lib/providers/counter_provider.dart`

Let's create a very simple `Notifier`. Starting from Riverpod 3.0+, we can no longer access `state` directly from outside the Notifier, so we create a `setValue` method:

```dart
import 'package:flutter_riverpod/flutter_riverpod.dart';

// Class that manages the counter state
class CounterNotifier extends Notifier<int> {
  // Returns the initial value
  @override
  int build() => 0;
  
  // Method to modify the value
  void setValue(int value) {
    state = value;
  }
}

// Provider that makes CounterNotifier available
final counterProvider = NotifierProvider<CounterNotifier, int>(() {
  return CounterNotifier();
});
```

**How it works:**
- **To read the value:** `ref.watch(counterProvider)` - reads and keeps observing changes (rebuilds the widget when it changes)
- **To modify the value:** `ref.read(counterProvider.notifier).setValue(newValue)` - accesses the notifier and calls the method to change the state

**Why `.notifier`?** When we do `ref.read(counterProvider)`, we get only the **value** (the int, the number). But the value alone has no methods. To modify, we need to access the **manager** (the `CounterNotifier`), which is what has the methods like `setValue()`. That's why we use `ref.read(counterProvider.notifier)` - we're getting the manager, not the value.

Riverpod takes care of notifying all widgets that are using `ref.watch` automatically when you call `setValue`.

### Step 5: Creating the Pages

Now let's create two pages to demonstrate Riverpod's power: one page with a text field to type the counter value, and another that only displays the value. The interesting thing is that **we won't pass the value as a parameter** between pages - Riverpod handles that!

#### `lib/pages/counter_page.dart`

This is the main page with the text field to modify the counter:

```dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import '../providers/counter_provider.dart';
import 'display_page.dart';

class CounterPage extends ConsumerWidget {
  const CounterPage({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // ref.watch observes the provider and rebuilds when it changes
    final counter = ref.watch(counterProvider);

    return Scaffold(
      appBar: AppBar(
        title: const Text('Enter the Value'),
        backgroundColor: Colors.blue,
        foregroundColor: Colors.white,
      ),
      body: Center(
        child: Padding(
          padding: const EdgeInsets.all(32.0),
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              const Text(
                'Current value:',
                style: TextStyle(fontSize: 20),
              ),
              const SizedBox(height: 16),
              Text(
                '$counter',
                style: const TextStyle(
                  fontSize: 72,
                  fontWeight: FontWeight.bold,
                  color: Colors.blue,
                ),
              ),
              const SizedBox(height: 32),
              TextField(
                decoration: const InputDecoration(
                  labelText: 'Type a number',
                  border: OutlineInputBorder(),
                  prefixIcon: Icon(Icons.edit),
                ),
                keyboardType: TextInputType.number,
                onChanged: (value) {
                  // Try to convert text to number
                  final number = int.tryParse(value);
                  if (number != null) {
                    // ref.read doesn't rebuild, just modifies the state
                    ref.read(counterProvider.notifier).setValue(number);
                  }
                },
              ),
              const SizedBox(height: 48),
              ElevatedButton.icon(
                onPressed: () {
                  Navigator.push(
                    context,
                    MaterialPageRoute(builder: (context) => const DisplayPage()),
                  );
                },
                icon: const Icon(Icons.visibility),
                label: const Text('View on Another Page'),
                style: ElevatedButton.styleFrom(
                  padding: const EdgeInsets.symmetric(horizontal: 24, vertical: 16),
                ),
              ),
            ],
          ),
        ),
      ),
    );
  }
}
```

#### `lib/pages/display_page.dart`

This page only displays the counter value. Note that **we don't receive any parameter** - Riverpod provides the value automatically. Here we'll use `ref.read` instead of `ref.watch` to demonstrate the difference:

```dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import '../providers/counter_provider.dart';

class DisplayPage extends ConsumerWidget {
  const DisplayPage({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // Here we use ref.read - gets the value ONCE when the page is created
    final counter = ref.read(counterProvider);

    return Scaffold(
      appBar: AppBar(
        title: const Text('Display'),
        backgroundColor: Colors.green,
        foregroundColor: Colors.white,
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            const Icon(
              Icons.remove_red_eye,
              size: 64,
              color: Colors.green,
            ),
            const SizedBox(height: 24),
            const Text(
              'The counter is at:',
              style: TextStyle(fontSize: 24),
            ),
            const SizedBox(height: 16),
            Text(
              '$counter',
              style: const TextStyle(
                fontSize: 96,
                fontWeight: FontWeight.bold,
                color: Colors.green,
              ),
            ),
            const SizedBox(height: 48),
            Card(
              margin: const EdgeInsets.symmetric(horizontal: 32),
              child: Padding(
                padding: const EdgeInsets.all(16.0),
                child: Column(
                  children: const [
                    Icon(Icons.info_outline, size: 48, color: Colors.green),
                    SizedBox(height: 16),
                    Text(
                      'This page uses ref.read',
                      style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold),
                      textAlign: TextAlign.center,
                    ),
                    SizedBox(height: 8),
                    Text(
                      'It gets the current value when created. Go back, change the counter, and come back here - you\'ll see the new value!',
                      textAlign: TextAlign.center,
                    ),
                  ],
                ),
              ),
            ),
          ],
        ),
      ),
    );
  }
}
```

**ref.watch vs ref.read in practice:**

- **CounterPage (first)** uses `ref.watch`: The number updates automatically as you type in the TextField because it's "observing" changes.

- **DisplayPage (second)** uses `ref.read`: Gets the value only when the page is created. Since Flutter **recreates** the page every time you navigate to it, you'll always see the current value. However, if something changed the counter while you're looking at this page, it **wouldn't update** automatically.

**When to use each:**
- Use `ref.watch` when you need the screen to update automatically when detecting changes
- Use `ref.read` when you want to get the value just once, without observing (saves resources)

### Step 6: Configuring Main

#### `lib/main.dart`

```dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'pages/counter_page.dart';

void main() {
  // ProviderScope must wrap the entire app
  // It's the "general manager" of all providers
  runApp(
    const ProviderScope(
      child: MyApp(),
    ),
  );
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Riverpod Example',
      debugShowCheckedModeBanner: false,
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.blue),
      ),
      home: const CounterPage(),
    );
  }
}
```

### Step 7: Understanding the Data Flow

1. **Provider Creation:** We create a `CounterNotifier` class that extends `Notifier<int>` and defines the initial value in the `build()` method. We also create a `setValue()` method to modify the state.

2. **ProviderScope:** We wrap the app with `ProviderScope` in `main.dart` so all providers are available throughout the application.

3. **Sharing Between Pages:** Both pages (`CounterPage` and `DisplayPage`) access the **same provider** through `ref`. We don't need to pass parameters in `Navigator.push()` - Riverpod handles this!

4. **Reading with `ref.watch` and `ref.read`:** 
   - `CounterPage` uses `ref.watch(counterProvider)` to observe changes - that's why the number updates automatically as you type in the TextField
   - `DisplayPage` uses `ref.read(counterProvider)` to get the value only when the page is created - since Flutter recreates the page every time you navigate to it, it always shows the current value

5. **Modification with `ref.read`:** To modify the state, we use `ref.read(counterProvider.notifier).setValue(newValue)`. Simple and direct!

### `ref.watch` vs `ref.read`

It's important to understand the different ways to access the provider:

* **`ref.watch(counterProvider)`**: Reads the value AND subscribes to changes. The widget will be automatically rebuilt when the state changes. Use in the `build()` method (the method that builds/draws your widget on screen) when you need the screen to update automatically when detecting changes. When you use `ref.watch` inside a widget's `build()`, Riverpod "registers" that that widget wants to be notified of changes - so every time the state changes, Riverpod calls the `build()` method again, redrawing the widget with the new value.

* **`ref.read(counterProvider)`**: Reads the value once, WITHOUT subscribing to changes. The widget won't be rebuilt if the state changes. Use when you just want to get the current value without observing (saves resources). In our example, we use this in `DisplayPage`.

* **`ref.read(counterProvider.notifier).setValue()`**: Accesses the notifier (the manager) to call methods that modify the state. Use inside callbacks (like `onPressed` of a button or `onChanged` of a TextField) when you just want to execute a modification action, without rebuilding the widget. Never use `ref.watch` inside callbacks - this would cause problems by trying to rebuild the widget in the middle of an action.

### Practical Example

```dart
@override
Widget build(BuildContext context, WidgetRef ref) {  // <-- Widget's build method
  // watch: rebuilds when counter changes (observes changes)
  // When the state changes, this build() method runs again
  final counter = ref.watch(counterProvider);
  
  // read: gets the value once, doesn't observe changes
  // final counter = ref.read(counterProvider);
  
  return Column(
    children: [
      Text('$counter'), // Updates automatically if using watch
      
      TextField(
        onChanged: (value) {  // <-- Callback, not the build()
          final number = int.tryParse(value);
          if (number != null) {
            // read + notifier: accesses the manager to modify
            // Here we use read because we're inside a callback
            ref.read(counterProvider.notifier).setValue(number);
          }
        },
      ),
    ],
  );
}
```

### Working with Multiple Values

A common question: what if I need to store more than one value in my provider? For example, three integers instead of one? With Riverpod, you can create a class to group related values:

```dart
import 'package:flutter_riverpod/flutter_riverpod.dart';

// Class to group related values
class CountersState {
  final int counter1;
  final int counter2;
  final int counter3;

  CountersState({
    required this.counter1,
    required this.counter2,
    required this.counter3,
  });
}

// Notifier that manages the composite state
class CountersNotifier extends Notifier<CountersState> {
  @override
  CountersState build() {
    return CountersState(counter1: 0, counter2: 0, counter3: 0);
  }
  
  // Methods to modify individual values
  void setCounter1(int value) {
    state = CountersState(
      counter1: value,
      counter2: state.counter2,
      counter3: state.counter3,
    );
  }
  
  void setCounter2(int value) {
    state = CountersState(
      counter1: state.counter1,
      counter2: value,
      counter3: state.counter3,
    );
  }
  
  void setCounter3(int value) {
    state = CountersState(
      counter1: state.counter1,
      counter2: state.counter2,
      counter3: value,
    );
  }
}

// Provider
final countersProvider = NotifierProvider<CountersNotifier, CountersState>(() {
  return CountersNotifier();
});
```

**How to use:**

```dart
// Read values
final state = ref.watch(countersProvider);
Text('Counter 1: ${state.counter1}');

// Modify values
ref.read(countersProvider.notifier).setCounter1(10);
```

**Tip:** To make it easier to modify just one value, you can add a `copyWith` method to the `CountersState` class, as we show in the best practices section below.

### Best Practices with Riverpod

1. **Keep Providers Organized:** Create a `providers` folder and separate by functionality.

2. **Avoid Logic in Build:** If you need complex logic, create methods in the `Notifier` instead of manipulating the state directly.

3. **Use `ConsumerWidget` only when necessary:** If a widget doesn't need to access providers, it can be a normal `StatelessWidget`.

4. **Use `autoDispose` when appropriate:** By default, providers stay "alive" in memory all the time, even if no widget is using them. `autoDispose` makes the provider automatically destroyed when the last widget using it is removed from the screen. This saves memory!
   ```dart
   // WITHOUT autoDispose: stays in memory forever
   final counterProvider = NotifierProvider<CounterNotifier, int>(() {
     return CounterNotifier();
   });
   
   // WITH autoDispose: is cleaned when no widget is using it anymore
   final tempProvider = NotifierProvider.autoDispose<CounterNotifier, int>(() {
     return CounterNotifier();
   });
   ```
   **When to use autoDispose:**
   - ✅ Temporary data (product details, visited user profile, etc)
   - ✅ Screen-specific data that doesn't need to stay in memory
   - ❌ Global data you want to keep (logged user, app theme, shopping cart)

5. **Create helper methods for complex states:** If your state has many fields, create a `copyWith` method:
   ```dart
   class CountersState {
     final int counter1;
     final int counter2;
     
     CountersState({required this.counter1, required this.counter2});
     
     CountersState copyWith({int? counter1, int? counter2}) {
       return CountersState(
         counter1: counter1 ?? this.counter1,
         counter2: counter2 ?? this.counter2,
       );
     }
   }
   
   // Usage: makes it easy to update just one field
   state = state.copyWith(counter1: 10);
   ```

### Conclusion

Riverpod transforms state management in Flutter into something simple and powerful. With its architecture without `BuildContext`, type safety, and ease of testing, it has become the preferred choice of many modern Flutter developers.

The example we built demonstrates one of the most important powers of state management: **sharing state between different pages without passing parameters**. Both pages access the same provider through `ref` and automatically react to changes. This is state management in practice!

From here, you have a solid foundation to build scalable and well-structured Flutter applications. Try adding more features to the app, like saving the counter to SharedPreferences or creating multiple independent counters. The code is ready to evolve!

**Additional Resources:**
* [Official Riverpod Documentation](https://riverpod.dev)
* [Riverpod on Pub.dev](https://pub.dev/packages/flutter_riverpod)
* [Official Examples](https://github.com/rrousselGit/riverpod/tree/master/examples)