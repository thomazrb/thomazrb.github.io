---
title: "Tutorial: Flutter App Internationalization with 'flutter_localizations' and 'intl'"
date: 2025-06-30T01:36:52-03:00
draft: false
tags: ["Flutter", "Localization"]
categories: ["Flutter"]
---

This tutorial will guide you through the process of adding multi-language support (internationalization and localization) to your Flutter application, using the `flutter_localizations` and `intl` packages for Material Design widgets.

## 1. Introduction

**Internationalization (i18n)** is the process of designing and developing an application so that it can be adapted to different languages and regions without engineering changes. **Localization (l10n)** is the process of adapting an application for a specific locale or market by adding locale-specific components and translating text.

In Flutter, the **`flutter_localizations`** package provides localizations for standard Material Design widgets, while the **`intl`** package is the Dart library that handles message formatting, date and number formatting, and bidirectional text.

## 2. Prerequisites

* Flutter SDK installed and configured.

* An existing Flutter project.

* Basic knowledge of Flutter and Dart development.

## 3. Step-by-Step

### Step 1: Install Dependencies

Instead of manually editing `pubspec.yaml`, we will use the `flutter pub add` command to add the packages. Open your terminal at the root of your Flutter project and run the following commands:

```bash
flutter pub add flutter_localizations --sdk=flutter
flutter pub add intl
```

* The `flutter pub add flutter_localizations --sdk=flutter` command adds `flutter_localizations`, specifying that it is part of the Flutter SDK; it is not a third-party library.

* The `flutter pub add intl` command adds the `intl` package as a normal dependency.

After running these commands, your `pubspec.yaml` file will be automatically updated, and the packages will be downloaded. You should see something like:

```yaml
dependencies:
  flutter:
    sdk: flutter
  flutter_localizations:
    sdk: flutter
  intl: ^0.20.2 # The version may vary
```

### Step 2: Enable Localization Code Generation

Still in `pubspec.yaml`, add the `generate: true` flag in the `flutter` section. This instructs Flutter to automatically generate localization files from your ARB (Application Resource Bundle) files.

```yaml
flutter:
  uses-material-design: true
  generate: true # Add this line to enable localization code generation
  # ... other Flutter configurations
```

Run `flutter pub get` in your terminal after this change to ensure the configurations are applied.

### Step 3: Create the `l10n.yaml` Configuration File

Create a new file named **`l10n.yaml`** in the root of your project (at the same level as `pubspec.yaml`). This file configures how Flutter should generate the localization files.

```yaml
arb-dir: lib/l10n
template-arb-file: app_en.arb
output-localization-file: app_localizations.dart
```

* `arb-dir`: The directory where your `.arb` files (with translations) will be stored. We will create it in `lib/l10n`.

* `template-arb-file`: The name of your base ARB file (usually the default language, like English).

* `output-localization-file`: The name of the Dart file that will be generated with the localization classes.

### Step 4: Create the ARB (Application Resource Bundle) Files

Create the directory specified in `arb-dir`, which is **`lib/l10n`**. Inside it, create your ARB files for each language you want to support.

**`lib/l10n/app_en.arb` (English - base file):**

```json
{
  "@@locale": "en",
  "helloWorld": "Hello World!",
  "@helloWorld": {
    "description": "A conventional greeting"
  },
  "welcomeMessage": "Welcome, {name}!",
  "@welcomeMessage": {
    "description": "A welcome message with a name parameter",
    "placeholders": {
      "name": {
        "type": "String"
      }
    }
  }
}
```

**`lib/l10n/app_pt.arb` (Portuguese):**

```json
{
  "@@locale": "pt",
  "helloWorld": "Olá Mundo!",
  "welcomeMessage": "Bem-vindo(a), {name}!"
}
```

**Explanation of ARB files:**

* `"@@locale": "en"`: Defines the locale for this file.

* `"helloWorld": "Hello World!"`: The key for your string and its translation.

* `"@helloWorld"`: Optional metadata for the key, such as a description for translators.

* `"welcomeMessage": "Welcome, {name}!"`: Example of a string with a placeholder (`{name}`). The `@welcomeMessage` defines the type of the placeholder (these are mandatory when the string contains parameters so that the Flutter code generator knows how to create the corresponding Dart function).

### Step 5: Generate Localization Files

After creating or modifying your ARB files, you need to generate the corresponding Dart code. Run the following command in the terminal at the root of your project:

```bash
flutter gen-l10n
```

This command will read your `.arb` files and `l10n.yaml` and generate the `app_localizations.dart` file (and other necessary files) in the configured location. If the file already exists, it will be updated.

### Step 6: Configure `MaterialApp`

In your `main.dart` file (or where you define your `MaterialApp`), import the generated localization files and configure the `localizationsDelegates` and `supportedLocales` properties.

**`lib/main.dart` (example):**

```dart
import 'package:flutter/material.dart';
import 'package:flutter_localizations/flutter_localizations.dart';
// ATTENTION: The import below assumes 'app_localizations.dart' is generated directly in 'lib/l10n/'.
// Although the default Flutter approach (the one previously used) is
// 'package:flutter_gen/gen_l10n/app_localizations.dart;',
// if your environment generates the file in this location, use this line.
import 'l10n/app_localizations.dart';


void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Flutter Localization Demo',
      theme: ThemeData(
        primarySwatch: Colors.blue,
      ),
      // Localization properties
      localizationsDelegates: const [
        AppLocalizations.delegate, // Delegate generated from your ARB files
        GlobalMaterialLocalizations.delegate, // Localizations for Material widgets
        GlobalWidgetsLocalizations.delegate,  // Localizations for basic widgets
      ],
      supportedLocales: const [
        Locale('en'), // English
        Locale('pt'), // Portuguese
        // Add other languages you support
      ],
      home: const MyHomePage(),
    );
  }
}

class MyHomePage extends StatefulWidget {
  const MyHomePage({super.key});

  @override
  State<MyHomePage> createState() => _MyHomePageState();
}

class _MyHomePageState extends State<MyHomePage> {
  @override
  Widget build(BuildContext context) {
    // Accessing localized strings
    final appLocalizations = AppLocalizations.of(context)!;

    return Scaffold(
      appBar: AppBar(
        title: Text(appLocalizations.helloWorld),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: <Widget>[
            Text(
              appLocalizations.welcomeMessage('User'), // Using the string with a placeholder
              style: Theme.of(context).textTheme.headlineMedium,
            ),
            const SizedBox(height: 20),
            Text(
              'Current Language: ${Localizations.localeOf(context).languageCode}',
              style: Theme.of(context).textTheme.bodyLarge,
            ),
          ],
        ),
      ),
    );
  }
}
```

**Explanation:**

* `import 'l10n/app_localizations.dart';`: This is the import path you requested, assuming `app_localizations.dart` is generated directly in the `lib/l10n/` folder. Note that the default Flutter approach (especially in newer versions) for generated files is `package:flutter_gen/gen_l10n/app_localizations.dart;`. Choose the path that works for your environment.

* `AppLocalizations.delegate`: This is the delegate specific to your application that knows how to load translations from your ARB files.

In Flutter, a `LocalizationsDelegate` is a class that knows how to load specific resources for a `Locale` (language and region) for your application. It's like a "manager" that knows where to find the translations and how to make them available to your widgets.

* `GlobalMaterialLocalizations.delegate`, `GlobalWidgetsLocalizations.delegate`: These are the delegates provided by `flutter_localizations` (i.e., native to the Flutter SDK itself) that provide translations for standard Material Design widgets (such as `DatePicker`, `TimePicker`, navigation buttons, etc.), ensuring that month names, "OK" and "CANCEL" buttons, "back" buttons, etc., are translated.

* `supportedLocales`: A list of all `Locale`s that your application supports. Flutter will attempt to match the user's device language with one of the languages in this list.

### Step 7: Use Localized Strings in Your Code

To access your translated strings, you will use the `AppLocalizations` class (the class name is derived from `output-localization-file` in `l10n.yaml`).

```dart
// Inside a widget that has access to a BuildContext
final appLocalizations = AppLocalizations.of(context)!;

// For a simple string:
Text(appLocalizations.helloWorld)

// For a string with a placeholder:
Text(appLocalizations.welcomeMessage('UserName'))
```

When you run your app, Flutter will detect the device language and load the corresponding translations. If the device language is not supported, it will use the default language (defined in `template-arb-file` in `l10n.yaml`, which is `en` in this example).

### Step 8: Run the Application

Save all files and run your Flutter application:

```bash
flutter run
```

Your app should now display "Hello World!" or "Olá Mundo!" and the welcome message according to your device's language (if it's Portuguese or English).

### Additional Tips:

* **Change Device Language:** To test localization, you can change the language of your emulator or physical device in the system settings.

* **Correct Import:** Always verify the exact path of the `app_localizations.dart` file that Flutter generates in your project. The path can be `import 'l10n/app_localizations.dart';` (if generated directly in `lib/l10n/`) or `import 'package:flutter_gen/gen_l10n/app_localizations.dart';` (the more common default). Make sure to use the path your Flutter environment is using.

* **Add New Languages:** To add a new language, simply create a new `.arb` file (e.g., `app_es.arb` for Spanish) in the `lib/l10n` directory and add the `Locale('es')` to the `supportedLocales` list in your `MaterialApp`. **Remember to run `flutter gen-l10n` again** so that the new language and its translations are included in the generated code.
