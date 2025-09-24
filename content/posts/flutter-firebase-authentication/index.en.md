---
title: "Authentication with Flutter and Firebase: A Guide to Email & Password Login"
date: 2025-09-24T16:55:38-03:00
draft: false
tags: ["Flutter", "Firebase", "Authentication", "Login"]
categories: ["Flutter"]
---

Authentication is the gateway to most modern applications. Protecting routes, personalizing the user experience, and ensuring data security are fundamental tasks. Fortunately, Firebase Authentication drastically simplifies this process.

In this guide, we will build a simple Flutter application with a login screen (using email and password) and a home screen that can only be accessed after authentication. The focus is to create a solid and easy-to-understand foundation that you can implement in your own projects.

### Prerequisites

The required environment is very similar to other Flutter/Firebase projects:

1.  **The Flutter SDK** installed and configured in your PATH.
2.  **Support for your desired platform enabled** (Android, iOS, Web, Windows, etc.). Check with the `flutter doctor` command.
3.  **A Google account** to access Firebase.
4.  **Node.js and npm** installed (required for the Firebase CLI).

### Step 1: Preparing the Firebase Environment

If you already use the Firebase CLI, you can skip this step.

1.  **Install the CLI globally** via npm:
    ```bash
    npm install -g firebase-tools
    ```

2.  **Log in** to your Google account to authenticate your machine:
    ```bash
    firebase login
    ```
    This will open a window in your browser for you to complete the login.

### Step 2: Configure the Project in the Firebase Console

1.  Go to the [Firebase Console](https://console.firebase.google.com/).
2.  Click **"Add project"** and give it a name (e.g., `flutter-auth-example`).
3.  In the left menu, navigate to **Build > Authentication**. This is the Firebase section dedicated to user management. It offers a complete backend service, easy-to-use SDKs, and ready-made UI libraries to authenticate users in your app. With this, you don't have to worry about creating and maintaining your own authentication server.
4.  Click the **"Get started"** button.
5.  In the **"Sign-in method"** tab, you will see a list of authentication providers. Firebase is extremely flexible, offering methods like logging in with Google, Facebook, Apple, and GitHub accounts, as well as options like anonymous login or by phone number. Each serves a different purpose. To keep this guide focused and simple, **select "Email/Password"**, which is the most traditional login method and a great foundation to start with.
6.  **Enable** the option and click **"Save"**.
7.  Still in the Authentication tab, go to the **"Users"** tab and click **"Add user"**. Create a test user with an email and password so we can test our login later.

### Step 3: Creating the Flutter Project

Let's create a new Flutter project from scratch.

1.  Open your terminal and run the command:
    ```bash
    flutter create flutter_auth_example
    ```

2.  Navigate into the newly created project folder:
    ```bash
    cd flutter_auth_example
    ```

### Step 4: Adding Flutter Dependencies

We need two main packages for this project: `firebase_core` to initialize the connection and `firebase_auth` to handle authentication.

1.  In the root of your project, run the command:
    ```bash
    flutter pub add firebase_core firebase_auth
    ```

### Step 5: Connecting the App to Firebase with FlutterFire

The FlutterFire CLI is the tool that connects your Flutter code to the project we created in the Firebase console.

1.  If you haven't already, install the FlutterFire CLI:
    ```bash
    dart pub global activate flutterfire_cli
    ```
    > **Attention:** If the terminal warns you that the `Pub\Cache\bin` directory is not in your "Path", follow the instructions in the warning to add it to your environment variables and **restart the terminal**.

2.  In the root of your project, run the configuration command:
    ```bash
    flutterfire configure
    ```
    The tool will list your Firebase projects. Select the `flutter-auth-example` we created. Then, choose the platforms for which you want to configure the app (e.g., android, ios, web). At the end, the `lib/firebase_options.dart` file will be generated automatically.

### Step 6: The Application Code

Now, let's get to the code! for better organization, we will split our logic into three files. Create a new `pages` folder inside your `lib` folder.

**1. Create the file `lib/pages/login_page.dart`**
This file will exclusively contain the widget for our login screen. A `StatefulWidget` is used here because we need to manage the form's state (what the user types) and the loading state (`_isLoading`).

```dart
import 'package:flutter/material.dart';
import 'package:firebase_auth/firebase_auth.dart';

// The login screen is a StatefulWidget so we can manage the
// loading state and the text field controllers.
class LoginPage extends StatefulWidget {
  const LoginPage({super.key});

  @override
  State<LoginPage> createState() => _LoginPageState();
}

class _LoginPageState extends State<LoginPage> {
  // Controllers to capture the text from the Email and Password fields.
  final TextEditingController _emailController = TextEditingController();
  final TextEditingController _passwordController = TextEditingController();
  // Variable to control the display of the progress indicator (loader).
  bool _isLoading = false;

  // Asynchronous function to handle the login process.
  Future<void> _login() async {
    // Update the UI to show the loader.
    setState(() {
      _isLoading = true;
    });

    try {
      // Use the FirebaseAuth SDK to attempt to sign in with the provided credentials.
      // The `.trim()` method removes whitespace from the beginning and end of the string.
      await FirebaseAuth.instance.signInWithEmailAndPassword(
        email: _emailController.text.trim(),
        password: _passwordController.text.trim(),
      );
      // If the login is successful, the StreamBuilder in main.dart (AuthWrapper)
      // will detect the state change and automatically redirect the user.
    } on FirebaseAuthException catch (e) {
      // If Firebase returns an error (e.g., wrong password, user not found),
      // it will be caught here.
      final snackBar = SnackBar(
        content: Text('Failed to log in: ${e.message}'),
        backgroundColor: Colors.red,
      );
      // Display an error message at the bottom of the screen.
      ScaffoldMessenger.of(context).showSnackBar(snackBar);
    } finally {
      // The 'finally' block will always execute, regardless of success or error.
      // The 'if (mounted)' check ensures that the widget is still on the screen
      // before trying to update its state, preventing errors.
      if (mounted) {
        setState(() {
          _isLoading = false; // Hide the loader.
        });
      }
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Login')),
      body: Center(
        child: SingleChildScrollView(
          padding: const EdgeInsets.all(16.0),
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              const Icon(Icons.lock, size: 80, color: Colors.blue),
              const SizedBox(height: 20),
              TextField(
                controller: _emailController,
                decoration: const InputDecoration(
                  labelText: 'Email',
                  border: OutlineInputBorder(),
                  prefixIcon: Icon(Icons.email),
                ),
                keyboardType: TextInputType.emailAddress,
              ),
              const SizedBox(height: 16),
              TextField(
                controller: _passwordController,
                decoration: const InputDecoration(
                  labelText: 'Password',
                  border: OutlineInputBorder(),
                  prefixIcon: Icon(Icons.vpn_key),
                ),
                obscureText: true, // Hides the password text.
              ),
              const SizedBox(height: 24),
              // Conditional rendering: if _isLoading is true, show the loader.
              // Otherwise, show the "Log In" button.
              _isLoading
                  ? const CircularProgressIndicator()
                  : ElevatedButton(
                      onPressed: _login,
                      style: ElevatedButton.styleFrom(
                        padding: const EdgeInsets.symmetric(
                            horizontal: 50, vertical: 15),
                      ),
                      child: const Text('Log In'),
                    ),
            ],
          ),
        ),
      ),
    );
  }
}
```

**2. Create the file `lib/pages/home_page.dart`**
This file will contain the widget for the main screen. It is a `StatelessWidget` because its only job is to display information (the user's email) that it receives from the outside, without managing any complex internal state.

```dart
import 'package:flutter/material.dart';
import 'package:firebase_auth/firebase_auth.dart';

// The Home screen is a StatelessWidget because it only displays data
// and doesn't need to manage an internal state that changes over time.
class HomePage extends StatelessWidget {
  // Receives the User object, which contains the logged-in user's information.
  final User user;

  const HomePage({super.key, required this.user});

  // Asynchronous function to log out.
  Future<void> _logout() async {
    // Calls the signOut method from Firebase to log the user out.
    await FirebaseAuth.instance.signOut();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Home Page'),
        // Actions are widgets that appear at the end of the AppBar.
        actions: [
          IconButton(
            icon: const Icon(Icons.logout),
            onPressed: _logout, // Calls the logout function when pressed.
          ),
        ],
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            const Text(
              'Welcome!',
              style: TextStyle(fontSize: 24, fontWeight: FontWeight.bold),
            ),
            const SizedBox(height: 16),
            // Displays the user's email. The '??' is a null-check operator:
            // if user.email is null, it will use the text on the right.
            Text(
              user.email ?? 'Email not available',
              style: const TextStyle(fontSize: 18),
            ),
          ],
        ),
      ),
    );
  }
}
```

**3. Replace the content of `lib/main.dart`**
Now, our main file becomes cleaner. Its main responsibility is to initialize Firebase and use the `AuthWrapper` to decide which screen (Login or Home) should be shown to the user based on their authentication status.

```dart
import 'package:flutter/material.dart';
import 'package:firebase_core/firebase_core.dart';
import 'package:firebase_auth/firebase_auth.dart';
import 'firebase_options.dart'; // Import the configuration generated by FlutterFire.
import 'pages/home_page.dart';   // Import the HomePage.
import 'pages/login_page.dart';  // Import the LoginPage.

// The main function is the application's entry point.
// It is 'async' because Firebase initialization is an asynchronous operation.
void main() async {
  // Ensures that all Flutter bindings are initialized before running the app.
  // Essential for using plugins like Firebase.
  WidgetsFlutterBinding.ensureInitialized();
  // Initializes Firebase using the configurations specific to the current platform.
  await Firebase.initializeApp(
    options: DefaultFirebaseOptions.currentPlatform,
  );
  runApp(const MyApp());
}

// MyApp is the root widget of your application.
class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Flutter Firebase Auth',
      theme: ThemeData(
        primarySwatch: Colors.blue,
        visualDensity: VisualDensity.adaptivePlatformDensity,
      ),
      // The 'home' of our app is the AuthWrapper, which will control the initial navigation.
      home: const AuthWrapper(),
      debugShowCheckedModeBanner: false,
    );
  }
}

// The AuthWrapper is the heart of our authentication logic.
// It decides which screen to show based on the user's login status.
class AuthWrapper extends StatelessWidget {
  const AuthWrapper({super.key});

  @override
  Widget build(BuildContext context) {
    // StreamBuilder is a widget that rebuilds itself every time it receives
    // a new value from a Stream (a flow of data).
    return StreamBuilder<User?>(
      // The 'authStateChanges' stream from Firebase notifies the app whenever
      // the user's authentication state changes (login, logout).
      stream: FirebaseAuth.instance.authStateChanges(),
      builder: (context, snapshot) {
        // If the connection is still waiting for data, we show a loader.
        if (snapshot.connectionState == ConnectionState.waiting) {
          return const Scaffold(
            body: Center(child: CircularProgressIndicator()),
          );
        }
        // If the snapshot has data (snapshot.hasData), it means the stream
        // emitted a 'User' object, so the user is logged in.
        if (snapshot.hasData) {
          // Redirect to the HomePage, passing the user object.
          return HomePage(user: snapshot.data!);
        }
        // If there is no data, the user is not logged in.
        // Redirect to the LoginPage.
        return const LoginPage();
      },
    );
  }
}
```

### Step 7: Testing the Application

With the code in place and the test user created in the Firebase console, it's time to test.

1.  **Run the application** on your preferred platform (Android, iOS, Web, etc.):
    ```bash
    flutter run
    ```

2.  The login screen should appear.
3.  Use the **email and password** of the user you registered in Step 2.
4.  When you click "Log In", you should be redirected to the home screen, which will display your email.
5.  Click the "logout" icon in the app bar. You will be taken back to the login screen.
6.  Try to log in with an incorrect password to see the error message appear at the bottom of the screen.

### Conclusion

Congratulations! You have implemented a complete and secure authentication flow with Flutter and Firebase. The structure we created, using a `StreamBuilder` to "listen" for the login state and separating the screens into different files, is one of the most robust and efficient ways to manage user sessions in a Flutter application. From here, you can easily expand to include a registration screen, a "forgot password" feature, or login with social providers like Google and Facebook.