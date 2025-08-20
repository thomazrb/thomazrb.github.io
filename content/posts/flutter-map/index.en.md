---
title: "Creating an Interactive Map with Pins in Flutter using flutter_map"
date: 2025-08-19T22:20:15-03:00
draft: false
tags: ["Flutter", "Maps", "OpenStreetMap"]
categories: ["Flutter"]
---

Adding maps to a Flutter application is a powerful feature, but many developers think Google Maps is the only option. Fortunately, there are robust, open-source alternatives like the **`flutter_map`** package, which uses **OpenStreetMap** data and offers incredible flexibility.

In this guide, we'll build a simple application that displays a map with custom and interactive pins (markers). It's the perfect foundation for any project that needs geolocation, from delivery apps to tourist guides.

### Why use `flutter_map`?

* **It's Open Source:** Completely free and maintained by the community.
* **Highly Customizable:** Allows the use of various map providers and the creation of markers with any Flutter widget.
* **Excellent Web Support:** Works perfectly in web applications, as well as mobile and desktop.

### Step 1: Creating the Project and Adding Dependencies

Let's start by creating a new Flutter project and adding the necessary packages.

1.  **Create the project:**
    ```bash
    flutter create flutter_map_example
    ```

2.  **Navigate to the project folder:**
    ```bash
    cd flutter_map_example
    ```

3.  **Add the dependencies:** We'll need two packages. `flutter_map` for the map itself, and `latlong2` to make handling geographic coordinates easier.
    ```bash
    flutter pub add flutter_map latlong2
    ```
    This command adds the necessary lines to your `pubspec.yaml` file and downloads the packages.

### Step 2: The Map's Structure

The structure of `flutter_map` is based on layers. Think of it like stacking transparent sheets of paper: the first is the base map, the second contains the pins, the third could have polygons, and so on.

We will replace the content of the `lib/main.dart` file with our code. The basic structure will be:

* **`FlutterMap`**: The main widget that holds the entire map.
* **`MapOptions`**: Where we define the map's initial center and zoom level.
* **`children`**: A list of layers that will be drawn on the map.

Our first layer will be the `TileLayer`, which is responsible for fetching and displaying the map "tiles" from OpenStreetMap.

### Step 3: Creating Custom and Interactive Pins

The real magic of `flutter_map` is in the `MarkerLayer`. A `Marker` (pin) can have any Flutter widget as its `child`. This gives us complete freedom to create markers that are not just an icon, but also contain text, buttons, and gestures.

In our example, we'll create a helper function `_buildCityMarker` that returns a `Marker`. This marker will be composed of:

1.  An **`InkWell`** to capture taps and provide visual feedback (like the mouse cursor changing on the web).
2.  A `Column` to stack the icon and the text.
3.  An `Icon` for the visual pin.
4.  A `Container` with a `Text` to display the city's name.

### Step 4: The Complete Code

Now, let's put it all together. Replace the entire content of your `lib/main.dart` file with the code below. It's complete, commented, and ready to run.

```dart
import 'package:flutter/material.dart';
import 'package:flutter_map/flutter_map.dart';
import 'package:latlong2/latlong.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'ES Map',
      theme: ThemeData(
        primarySwatch: Colors.blue,
      ),
      home: const MapaCidadesEsPage(),
      debugShowCheckedModeBanner: false,
    );
  }
}

class MapaCidadesEsPage extends StatefulWidget {
  const MapaCidadesEsPage({super.key});

  @override
  State<MapaCidadesEsPage> createState() => _MapaCidadesEsPageState();
}

class _MapaCidadesEsPageState extends State<MapaCidadesEsPage> {
  // Geographic coordinates for the cities
  static const LatLng _pontoSaoMateus = LatLng(-18.7160, -39.8582);
  static const LatLng _pontoVitoria = LatLng(-20.3194, -40.3378);

  // A central point to initialize the map between the two cities
  static const LatLng _pontoCentral = LatLng(-19.5177, -40.0980);

  // A list that will hold our pins (markers)
  late final List<Marker> _markers;

  @override
  void initState() {
    super.initState();

    // Initialize the list of markers when the widget is created
    _markers = [
      // São Mateus Pin
      _buildCityMarker(
        point: _pontoSaoMateus,
        cityName: 'São Mateus',
        color: Colors.red,
      ),

      // Vitória Pin
      _buildCityMarker(
        point: _pontoVitoria,
        cityName: 'Vitória',
        color: Colors.blue,
      ),
    ];
  }

  /// Helper function to create a custom Marker (pin)
  Marker _buildCityMarker({
    required LatLng point,
    required String cityName,
    required Color color,
  }) {
    return Marker(
      point: point,
      width: 95, // Fixed width that accommodates the longest name well
      height: 65, // Fixed height for the pin
      child: InkWell(
        onTap: () {
          // Action on pin click: shows a simple notification (SnackBar)
          ScaffoldMessenger.of(context).showSnackBar(
            SnackBar(
              content: Text('You clicked on $cityName!'),
              backgroundColor: color,
              duration: const Duration(seconds: 2),
            ),
          );
        },
        // The visual content of the pin is a column with the icon and text
        child: Column(
          children: [
            Icon(
              Icons.location_pin,
              color: color,
              size: 40,
            ),
            Container(
              padding: const EdgeInsets.symmetric(horizontal: 5, vertical: 2),
              decoration: BoxDecoration(
                color: Colors.white.withAlpha(204),
                borderRadius: BorderRadius.circular(10),
              ),
              child: Text(
                cityName,
                style: const TextStyle(
                  fontWeight: FontWeight.bold,
                  fontSize: 12,
                ),
              ),
            ),
          ],
        ),
      ),
    );
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Map - Espírito Santo'),
        centerTitle: true,
      ),
      // The main widget that renders the map
      body: FlutterMap(
        // Map options, such as initial center and zoom
        options: const MapOptions(
          initialCenter: _pontoCentral,
          initialZoom: 8.0, // Zoom level that shows both cities
        ),
        // The map is built in layers
        children: [
          // Layer 1: The base map from OpenStreetMap
          TileLayer(
            urlTemplate: '[https://tile.openstreetmap.org/](https://tile.openstreetmap.org/){z}/{x}/{y}.png',
            userAgentPackageName: 'com.example.flutter_map_example',
          ),
          // Layer 2: Our pins (markers)
          MarkerLayer(
            markers: _markers,
          ),
        ],
      ),
    );
  }
}
```
### An Important Note about OpenStreetMap

When you run the app, you will notice an informational message in your debug console. It is not an error, but an important warning from the `flutter_map` community: the public OpenStreetMap (OSM) servers are not intended for heavy commercial use.

OSM is an incredible project maintained by volunteers, and its servers have limited capacity. For development, testing, and personal projects, their use is acceptable. However, if you plan to release an app to a large audience, the recommended practice is to use a commercial tile provider, many of which offer generous free tiers. Switching providers usually just involves changing the `urlTemplate` in your `TileLayer`.

### Conclusion

And there you have it! With just a few lines of code, we've created a functional and interactive map in Flutter. From here, the possibilities are enormous: you can load pins from an API, navigate to a detail screen when a marker is tapped, or even use other packages from the `flutter_map` ecosystem to add pop-ups and marker clustering.

`flutter_map` proves to be a powerful and accessible tool for any developer looking to integrate maps into their projects without relying on proprietary services.