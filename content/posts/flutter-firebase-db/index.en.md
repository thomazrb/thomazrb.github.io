---
title: "Creating an App with Flutter and Firebase"
date: 2025-08-19T11:03:03-03:00
draft: false
tags: ["Flutter", "Firebase", "Firestore", "Database"]
categories: ["Flutter"]
---

The power of Flutter lies in its ability to create applications for multiple platforms from a single codebase. When combined with Firebase, this power extends to creating cloud-connected apps quickly and efficiently.

In this guide, we'll demonstrate this versatility by building a user registration app (name and CPF). Although our example focuses on compiling for **Windows**, the same principles apply to web, mobile, and other desktop platforms with very few changes.

### Prerequisites

Before you begin, ensure you have your environment set up:

1.  **The Flutter SDK** installed and added to your PATH.
2.  **Support for Windows development enabled in Flutter.** You can check if everything is correct with the `flutter doctor` command. If anything is missing in the "Windows" section, follow the instructions provided by the command itself.
3.  **A Google account** to access Firebase.
4.  **Node.js and npm** installed. They are required for the Firebase CLI. Download the **LTS** version from the official [nodejs.org](https://nodejs.org/) website.

### Step 1: Preparing the Firebase Environment

This step prepares your computer to communicate with Firebase. If you've done this before, you can skip to the next step.

1.  **Install the CLI globally** via npm. This adds the Firebase commands to your system:
    ```bash
    npm install -g firebase-tools
    ```

2.  **Log in** to your Google account to authenticate your machine:
    ```bash
    firebase login
    ```
    This command will open a window in your browser for authentication.

### Step 2: Configure the Project in the Firebase Console

1.  Go to the [Firebase Console](https://console.firebase.google.com/).
2.  Click **"Add project"** and give it a name (e.g., `flutter-firebase-db-example`).
3.  Follow the setup steps. You don't need to enable Google Analytics for this tutorial.
4.  In the left menu, go to **Build > Firestore Database** and click **"Create database"**.
5.  Select **"Start in test mode"**.
    * **Warning:** Test mode allows anyone to read and write to your database. It's great for development, but for a production app, you **must** configure more restrictive security rules.
6.  Choose a location for your data and click **"Enable"**.

### Step 3: Creating the Flutter Project

With the environment ready, let's create our Flutter project.

1.  Open your terminal or command prompt.
2.  Run the command below to create the project:
    ```bash
    flutter create flutter_firebase_db_example
    ```
3.  Navigate into the newly created project folder. All subsequent commands should be run from here.
    ```bash
    cd flutter_firebase_db_example
    ```

### Step 4: Add Dependencies in Flutter

Now, let's add the necessary packages for our Flutter app to communicate with Firebase. The quickest and safest way to do this is through the terminal.

1.  In the root of your project, run the command below to add the `firebase_core` and `cloud_firestore` packages at once:
    ```bash
    flutter pub add firebase_core cloud_firestore
    ```
    This command will automatically find the latest versions of the packages, add them to your `pubspec.yaml` file, and run `flutter pub get` for you.

> **Troubleshooting on Windows:** If you encounter an error like `ERROR_INVALID_FUNCTION` when running the command above, it's very likely that your Flutter project is on a different drive than your SDK (e.g., project on the `D:` drive and SDK on the `C:` drive). To fix this, move your project folder to the same drive as the Flutter SDK (usually `C:`) and try again.

### Step 5: Connect the App to Firebase with FlutterFire

The FlutterFire CLI automates the process of connecting your project to Firebase.

1.  Install the FlutterFire CLI (if you haven't already):
    ```bash
    dart pub global activate flutterfire_cli
    ```
    > **Attention to the "Warning" on Windows:** After running the command above, you might see a warning that the `...Pub\Cache\bin` directory is not in your "Path". **This warning is important!** It means the terminal won't find the `flutterfire` command in the next step.
    >
    > **How to fix it:**
    > 1.  Search for "Environment Variables" in the Windows Start Menu and open "Edit the system environment variables".
    > 2.  In the window that opens, click "Environment Variables...".
    > 3.  In the "User variables" section, find and select the `Path` variable and click "Edit...".
    > 4.  Click "New" and paste the path that appeared in your warning (usually `C:\Users\YOUR_USER\AppData\Local\Pub\Cache\bin`).
    > 5.  Click "OK" on all windows to save.
    > 6.  **Close and reopen your terminal** for the changes to take effect.

2.  In the root of your project, run the configuration command:
    ```bash
    flutterfire configure
    ```
    The command will detect your Firebase project and ask which platforms you want to configure. **Make sure to select the `windows` option**. It will then generate the `lib/firebase_options.dart` file with the correct keys for your desktop application.
    > **Tip:** You don't have to select all platforms at once. The best practice is to select only the ones you are currently developing for (like `windows` for this guide). If you decide to build your app for Android, iOS, or web in the future, just run the `flutterfire configure` command again and add the new platform. The choice is not permanent!

### Step 6: The Application Code

Now for the fun part! Replace the entire content of your `lib/main.dart` file with the complete code below. It creates the interface to list and add users.

```dart
import 'package:flutter/material.dart';
import 'package:firebase_core/firebase_core.dart';
import 'package:cloud_firestore/cloud_firestore.dart';
import 'firebase_options.dart'; // Imports the configuration generated by FlutterFire

// Main entry point of the application
void main() async {
  // Ensures that the Flutter bindings are initialized
  WidgetsFlutterBinding.ensureInitialized();
  // Initializes Firebase using the options for the current platform (Windows, in this case)
  await Firebase.initializeApp(options: DefaultFirebaseOptions.currentPlatform);
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Flutter Desktop App with Firestore',
      theme: ThemeData(
        primarySwatch: Colors.indigo,
        visualDensity: VisualDensity.adaptivePlatformDensity,
      ),
      home: const HomePage(),
      debugShowCheckedModeBanner: false,
    );
  }
}

class HomePage extends StatefulWidget {
  const HomePage({super.key});

  @override
  State<HomePage> createState() => _HomePageState();
}

class _HomePageState extends State<HomePage> {
  // Controllers for the dialog text fields
  final TextEditingController _nomeController = TextEditingController();
  final TextEditingController _cpfController = TextEditingController();

  // Reference to the 'usuarios' collection in Firestore
  final CollectionReference _usuarios = FirebaseFirestore.instance.collection(
    'usuarios',
  );

  // Function to display the add user dialog
  Future<void> _mostrarDialogoAdicionar() async {
    // Clears the controllers before opening the dialog
    _nomeController.clear();
    _cpfController.clear();

    await showDialog(
      context: context,
      builder: (BuildContext context) {
        return AlertDialog(
          title: const Text('Add New User'),
          content: Column(
            mainAxisSize: MainAxisSize.min,
            children: [
              TextField(
                controller: _nomeController,
                decoration: const InputDecoration(labelText: 'Name'),
              ),
              TextField(
                controller: _cpfController,
                decoration: const InputDecoration(labelText: 'CPF'),
              ),
            ],
          ),
          actions: [
            TextButton(
              child: const Text('Cancel'),
              onPressed: () {
                Navigator.of(context).pop();
              },
            ),
            ElevatedButton(
              child: const Text('Add'),
              onPressed: () {
                final String nome = _nomeController.text;
                final String cpf = _cpfController.text;
                if (nome.isNotEmpty && cpf.isNotEmpty) {
                  // Adds a new document to the 'usuarios' collection
                  _usuarios.add({"nome": nome, "cpf": cpf});

                  // Closes the dialog
                  Navigator.of(context).pop();
                }
              },
            ),
          ],
        );
      },
    );
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('User Registration')),
      // StreamBuilder listens to changes in the Firestore collection in real-time
      body: StreamBuilder(
        stream: _usuarios.snapshots(), // The stream to listen to
        builder: (context, AsyncSnapshot<QuerySnapshot> streamSnapshot) {
          if (streamSnapshot.hasData) {
            // If there is data, build a ListView
            return ListView.builder(
              itemCount: streamSnapshot.data!.docs.length,
              itemBuilder: (context, index) {
                final DocumentSnapshot documentSnapshot =
                    streamSnapshot.data!.docs[index];
                return Card(
                  margin: const EdgeInsets.all(10),
                  child: ListTile(
                    title: Text(documentSnapshot['nome']),
                    subtitle: Text(documentSnapshot['cpf']),
                  ),
                );
              },
            );
          }
          // While data is loading, show a progress indicator
          return const Center(child: CircularProgressIndicator());
        },
      ),
      // Floating action button to add new users
      floatingActionButton: FloatingActionButton(
        onPressed: () => _mostrarDialogoAdicionar(),
        child: const Icon(Icons.add),
      ),
    );
  }
}
```
*Note: The complete source code for this project is available on GitHub: [flutter_firebase_db_example](https://github.com/thomazrb/flutter_firebase_db_example).*

### Step 7: Test and Build the Executable

With the code ready, let's run and compile our application for Windows.

1.  **Test in debug mode:**
    ```bash
    flutter run -d windows
    ```
    A native Windows window will open with your application. Try adding a few users and see the real-time magic happen.

2.  **Generate the executable (`.exe`):**
    ```bash
    flutter build windows
    ```
    After completion, Flutter will tell you where to find the executable. It will usually be in the `build\windows\runner\Release` folder. You can take the entire contents of this folder, zip it, and distribute it to other Windows computers!

### Conclusion

Congratulations! You have created a native desktop application for Windows with a cloud database, using a single codebase with Flutter. This demonstrates the incredible flexibility of the tool, allowing you to bring your ideas to virtually any platform without rewriting everything from scratch.