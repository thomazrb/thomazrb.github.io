---
title: "Detailed Tutorial: Creating an Interactive Calendar Widget in Flutter"
date: 2025-09-07T10:52:27-03:00
draft: false
tags: ["Flutter", "Custom Widgets", "Dart", "UI", "State Management"]
categories: ["Flutter"]
---

In many applications, a calendar is more than just a grid of dates; it's a scheduling tool, an event planner, or a way to visualize data over time. Although ready-made packages exist, building your own calendar widget in Flutter offers unparalleled control over its appearance, functionality, and business logic.

In this tutorial, we will dive deep into the process of creating a monthly calendar widget. We'll dissect the source code snippet by snippet, explaining the logic behind each part. At the end, we will present the complete, clean files, ready to be copied into your project.

The full source code for this project is available on GitHub for reference:
**[Access the complete repository here!](https://github.com/thomazrb/flutter_calendar_example)**

### The Architecture: Separating Responsibilities

The key to a good reusable component is the separation of responsibilities:

1.  **`main.dart` (The Main Screen):** It's the "brain." It is responsible for managing the state, creating the data, and deciding what to do when a day is clicked.
2.  **`monthly_calendar.dart` (The Reusable Widget):** It's the "worker." It just receives data, draws it on the screen, and notifies the "brain" when an interaction occurs.

Let's analyze the code for each one.

### Part 1: The Application's Brain (`main.dart`)

This file sets up the main screen that orchestrates the use of our calendar.

#### Snippet 1: Basic Structure and State Management

First, we create the app structure and define our `HomePage` as a `StatefulWidget`. This is crucial because the screen needs to "remember" which date the user has selected, and this information is our "state variable," `_selectedDate`.

```dart
import 'package:flutter/material.dart';
import 'package:flutter_calendar_example/widgets/monthly_calendar.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Calendar Example',
      debugShowCheckedModeBanner: false,
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.deepPurple),
        useMaterial3: true,
      ),
      home: const HomePage(),
    );
  }
}

class HomePage extends StatefulWidget {
  const HomePage({super.key});

  @override
  State<HomePage> createState() => _HomePageState();
}

class _HomePageState extends State<HomePage> {
  DateTime? _selectedDate;

  @override
  Widget build(BuildContext context) {
    // ... The rest of the UI will come here
    return Scaffold(); // Placeholder
  }
}
```

#### Snippet 2: Preparing the Data

Inside the `build` method, we prepare the data that will be passed to the calendar. In a real application, this would come from an API or database. Here, we create sample maps for events and colors. Using `DateTime` as the key is the best practice as it makes the data unambiguous.

```dart
// Inside the build() method of _HomePageState

final today = DateTime.now();
final currentYear = today.year;
final currentMonth = today.month;

const monthNames = [
  'January', 'February', 'March', 'April', 'May', 'June',
  'July', 'August', 'September', 'October', 'November', 'December'
];

final Map<DateTime, Widget> monthEvents = {
  DateTime(currentYear, currentMonth, 5): const Icon(Icons.star, color: Colors.amber, size: 24),
  DateTime(currentYear, currentMonth, 13): const Text("TXT", style: TextStyle(fontSize: 12, fontWeight: FontWeight.bold, color: Colors.blue)),
  DateTime(currentYear, currentMonth, 20): Column(
    mainAxisAlignment: MainAxisAlignment.center,
    children: const [
      Text("L1", style: TextStyle(fontSize: 9)),
      Text("L2", style: TextStyle(fontSize: 9)),
    ],
  ),
  DateTime(currentYear, currentMonth, 22): const Icon(Icons.favorite, color: Colors.pink, size: 24),
};

final Map<DateTime, Color> monthColors = {
  DateTime(currentYear, currentMonth, 1): Colors.blue[100]!,
  DateTime(currentYear, currentMonth, 7): Colors.red[100]!,
  DateTime(currentYear, currentMonth, 13): Colors.orange[100]!,
  DateTime(currentYear, currentMonth, 14): Colors.red[100]!,
  DateTime(currentYear, currentMonth, 21): Colors.red[100]!,
  DateTime(currentYear, currentMonth, 22): Colors.green[100]!,
  DateTime(currentYear, currentMonth, 28): Colors.red[200]!,
};
```

#### Snippet 3: Building the UI and Using the Calendar

Now, we build the `Scaffold` and call our `MonthlyCalendar` widget, passing all the data we've prepared.

```dart
// Inside the build() method, returning the Scaffold

return Scaffold(
  appBar: AppBar(
    title: const Text('My Interactive Calendar'),
    backgroundColor: Colors.blueGrey[800],
    foregroundColor: Colors.white,
  ),
  body: Padding(
    padding: const EdgeInsets.all(16.0),
    child: Column(
      children: [
        Text(
          '${monthNames[currentMonth - 1]} $currentYear',
          style: const TextStyle(fontSize: 22, fontWeight: FontWeight.bold),
        ),
        const SizedBox(height: 20),
        Center(
          child: MonthlyCalendar(
            year: currentYear,
            month: currentMonth,
            width: MediaQuery.of(context).size.width / 2,
            dayContents: monthEvents,
            dayColors: monthColors,
            selectedDay: _selectedDate,
            onDaySelected: (date) {
              // ... The callback will be in the next snippet
            },
          ),
        ),
        // ... The feedback text will come here
      ],
    ),
  ),
);
```

#### Snippet 4: Implementing Interactivity with the Callback (`onDaySelected`)

This is the heart of the interactivity. We pass a function to the `onDaySelected` parameter. When a day is clicked inside the calendar, this function is executed here on the main screen. Inside it, we use `setState` to update our `_selectedDate` variable, which forces Flutter to redraw the screen with the new information.

```dart
// onDaySelected parameter and the feedback Text

// ... inside the Center()
child: MonthlyCalendar(
  // ... other parameters
  onDaySelected: (date) {
    setState(() {
      _selectedDate = date;
    });

    ScaffoldMessenger.of(context).hideCurrentSnackBar();
    ScaffoldMessenger.of(context).showSnackBar(
      SnackBar(
        content: Text('Selected date: ${date.day}/${date.month}/${date.year}'),
        duration: const Duration(seconds: 1),
      ),
    );
  },
),
// ...
const SizedBox(height: 20),
Text(
  _selectedDate == null
      ? 'No day selected'
      : 'Selected day: ${_selectedDate!.day}/${_selectedDate!.month}/${_selectedDate!.year}',
  style: const TextStyle(fontSize: 18, fontWeight: FontWeight.w500),
),
```

---

### Part 2: The Worker (`monthly_calendar.dart`)

Now, let's dissect the reusable widget that does all the drawing work.

#### Snippet 1: Structure and Parameters

We define the `MonthlyCalendar` class and all its input parameters (properties). This is the widget's "contract": the information it needs to receive to function.

```dart
import 'package:flutter/material.dart';

class MonthlyCalendar extends StatelessWidget {
  final int year;
  final int month;
  final double width;
  final Map<DateTime, Widget>? dayContents;
  final Map<DateTime, Color>? dayColors;
  final void Function(DateTime)? onDaySelected;
  final DateTime? selectedDay;

  const MonthlyCalendar({
    super.key,
    required this.year,
    required this.month,
    required this.width,
    this.dayContents,
    this.dayColors,
    this.onDaySelected,
    this.selectedDay,
  });
  
  @override
  Widget build(BuildContext context) {
    // ... Main logic
  }
}
```

#### Snippet 2: Main Logic in the `build` Method

The `build` method starts with the "smart" logic: it calculates how many days the month has and on which day of the week it starts. Then, it filters the event and color maps to get only the data relevant to the current month and year.

```dart
// Inside the build() of MonthlyCalendar

// 1. Date Calculations
final int daysInMonth = DateTime(year, month + 1, 0).day;
final firstDayOfMonth = DateTime(year, month, 1);
final int startingWeekday = (firstDayOfMonth.weekday % 7) + 1;

// 2. Data Filtering
final Map<int, Widget> filteredContents = {};
if (dayContents != null) {
  for (final event in dayContents!.entries) {
    if (event.key.year == year && event.key.month == month) {
      filteredContents[event.key.day] = event.value;
    }
  }
}

final Map<int, Color> filteredColors = {};
if (dayColors != null) {
  for (final color in dayColors!.entries) {
    if (color.key.year == year && color.key.month == month) {
      filteredColors[color.key.day] = color.value;
    }
  }
}
// ... The rest of the UI construction will come here
```

#### Snippet 3: Helper Functions (`_generateDayList` and `_WeekdayHeader`)

To keep the `build` method clean, we delegate some tasks to helper functions and widgets.

**`_generateDayList`**: This function creates the list of items that the grid will display. It calculates how many empty spaces (`null`) are needed at the beginning of the month and then adds the days from 1 to the end of the month.

```dart
// Helper function inside MonthlyCalendar
List<int?> _generateDayList(int startingWeekday, int daysInMonth) {
  final List<int?> days = [];
  final int emptyDaysAtStart = startingWeekday - 1;
  for (int i = 0; i < emptyDaysAtStart; i++) {
    days.add(null);
  }
  for (int i = 1; i <= daysInMonth; i++) {
    days.add(i);
  }
  return days;
}
```

**`_WeekdayHeader`**: This is a simple, stateless widget that just draws the row with the initials of the days of the week (S, M, T, W, T, F, S), ensuring each initial occupies the same space as a calendar cell.

```dart
class _WeekdayHeader extends StatelessWidget {
  final double cellSize;
  const _WeekdayHeader({required this.cellSize});

  @override
  Widget build(BuildContext context) {
    const weekdays = ['S', 'M', 'T', 'W', 'T', 'F', 'S'];
    return Row(
      children: weekdays.map((day) {
        return SizedBox(
          width: cellSize,
          child: Text(
            day,
            textAlign: TextAlign.center,
            style: const TextStyle(
              fontWeight: FontWeight.bold,
              color: Colors.black54,
            ),
          ),
        );
      }).toList(),
    );
  }
}
```

#### Snippet 4: Building and Drawing the Cell Grid (`_CalendarGrid`)

This widget is the visual heart of our calendar. It is designed to be "dumb," meaning it contains no complex date logic; it just receives a list of days and pre-processed data and draws them in a grid. The main tool for this is `GridView.builder`.

```dart
class _CalendarGrid extends StatelessWidget {
  // ... parameters ...

  @override
  Widget build(BuildContext context) {
    return GridView.builder(
      shrinkWrap: true,
      physics: const NeverScrollableScrollPhysics(),
      gridDelegate: const SliverGridDelegateWithFixedCrossAxisCount(
        crossAxisCount: 7,
      ),
      itemCount: days.length,
      itemBuilder: (context, index) {
        // ... Logic to build each cell ...
      },
    );
  }
}
```

**Dissecting `GridView.builder`:**

* **`GridView.builder`**: We use this constructor because it's the most efficient way to create grids in Flutter. It builds its children (`itemBuilder`) on demand as they become visible on the screen. For our calendar, the list is small, but it's an excellent practice.
* **`shrinkWrap: true` and `physics: const NeverScrollableScrollPhysics()`**: These two properties are essential because we are placing a `GridView` (which is a scrollable widget) inside a `Column`. `shrinkWrap` forces the grid to take up only the necessary space, and `NeverScrollableScrollPhysics` disables the grid's own scrolling, preventing scrolling conflicts.
* **`gridDelegate`**: This is where we define the grid's layout. `SliverGridDelegateWithFixedCrossAxisCount(crossAxisCount: 7)` instructs the grid to have exactly 7 columns, which is perfect for a calendar week.
* **`itemBuilder`**: This is the function that is called for each item in our `days` list. It receives the `index` of the item and must return the widget that represents it. This is where the logic for each cell happens.

#### Snippet 5: The Logic for Building Each Cell (`itemBuilder`)

Now, let's analyze the code inside the `itemBuilder`, which is executed for each day of the month.

```dart
// Snippet from the itemBuilder inside _CalendarGrid

final day = days[index];

if (day == null) {
  return const SizedBox.shrink();
}

final Widget? dayContent = (dayContents != null && dayContents!.containsKey(day)) ? dayContents![day] : null;
final Color backgroundColor = (dayColors != null && dayColors!.containsKey(day)) ? dayColors![day]! : Colors.grey[200]!;

final bool isSelected = selectedDay != null &&
    selectedDay!.year == year &&
    selectedDay!.month == month &&
    selectedDay!.day == day;

return Material(
  color: Colors.transparent,
  child: InkWell(
    borderRadius: BorderRadius.circular(4),
    onTap: () {
      if (onDaySelected != null) {
        final clickedDate = DateTime(year, month, day);
        onDaySelected!(clickedDate);
      }
    },
    child: Container(
      height: cellSize,
      width: cellSize,
      margin: const EdgeInsets.all(2),
      decoration: BoxDecoration(
        color: backgroundColor,
        borderRadius: BorderRadius.circular(4),
        border: isSelected
            ? Border.all(color: Colors.blueGrey[700]!, width: 2.5)
            : Border.all(color: Colors.black12, width: 1),
      ),
      child: Stack(
        children: [
          Positioned(
            top: 4,
            right: 4,
            child: Text(
              day.toString(),
              style: const TextStyle(
                fontSize: 12,
                fontWeight: FontWeight.w500,
                color: Colors.black54,
              ),
            ),
          ),
          if (dayContent != null)
            Center(child: dayContent),
        ],
      ),
    ),
  ),
);
```

**Analyzing the `itemBuilder`:**

1.  **`if (day == null)`**: The first check handles the empty spaces at the beginning of the month. If the list item is `null`, we return a `SizedBox.shrink()`, which is an empty widget with no dimensions.
2.  **Data Fetching**: We fetch the `dayContent` and `backgroundColor` from the maps the widget received. If there is no entry for the current day, we use a default value (null for content and gray for the background).
3.  **Selection Logic**: The `isSelected` variable becomes `true` only if the `selectedDay` received from the parent widget exactly matches the year, month, and day of the cell being built.
4.  **Interactivity (`InkWell`)**: Each cell is wrapped in an `InkWell` to capture taps. In the `onTap`, we execute the `onDaySelected` callback, passing the complete `DateTime` of the clicked day. This is how the cell "talks" back to the main screen.
5.  **Visuals (`Container` and `Stack`)**: The `Container` is responsible for the cell's appearance: background color, rounded corners, and a conditional highlight border if `isSelected` is true. Finally, a `Stack` is used to layer the day number (positioned in the corner with `Positioned`) and the custom content (centered with `Center`).

#### Technical Detail: Why use `Material` before `InkWell`?

A common question when looking at the code above is: why is the `InkWell` wrapped by a `Material` widget?

The answer lies in how Flutter draws Material Design visual effects. The `InkWell` is responsible for creating the ripple effect when it's tapped, but it doesn't draw this effect on itself. Instead, it looks for the nearest `Material` ancestor in the widget tree and "asks" it to draw the ripple.

If we only had the `InkWell` wrapping our colored `Container`, the ripple effect would be invisible because it would be drawn *behind* the `Container`'s solid background color. By adding a transparent `Material` as a parent, we give the `InkWell` a dedicated "canvas" to draw the effect on top of everything, ensuring it's visible to the user.

---

### Complete Source Code (Without Comments)

Here are the two complete files, ready for you to copy and paste into your project. **Please note that the code in the GitHub repository is the original version written in Portuguese.** For a fully commented version of the code, please refer to the repository.

#### `lib/main.dart`

```dart
import 'package:flutter/material.dart';
import 'package:flutter_calendar_example/widgets/monthly_calendar.dart'; // Ensure filename matches

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Calendar Example',
      debugShowCheckedModeBanner: false,
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.deepPurple),
        useMaterial3: true,
      ),
      home: const HomePage(),
    );
  }
}

class HomePage extends StatefulWidget {
  const HomePage({super.key});

  @override
  State<HomePage> createState() => _HomePageState();
}

class _HomePageState extends State<HomePage> {
  DateTime? _selectedDate;

  @override
  Widget build(BuildContext context) {
    final today = DateTime.now();
    final currentYear = today.year;
    final currentMonth = today.month;

    const monthNames = [
      'January', 'February', 'March', 'April', 'May', 'June',
      'July', 'August', 'September', 'October', 'November', 'December'
    ];

    final Map<DateTime, Widget> monthEvents = {
      DateTime(currentYear, currentMonth, 5): const Icon(Icons.star, color: Colors.amber, size: 24),
      DateTime(currentYear, currentMonth, 13): const Text("TXT", style: TextStyle(fontSize: 12, fontWeight: FontWeight.bold, color: Colors.blue)),
      DateTime(currentYear, currentMonth, 20): Column(
        mainAxisAlignment: MainAxisAlignment.center,
        children: const [
          Text("L1", style: TextStyle(fontSize: 9)),
          Text("L2", style: TextStyle(fontSize: 9)),
        ],
      ),
      DateTime(currentYear, currentMonth, 22): const Icon(Icons.favorite, color: Colors.pink, size: 24),
    };

    final Map<DateTime, Color> monthColors = {
      DateTime(currentYear, currentMonth, 1): Colors.blue[100]!,
      DateTime(currentYear, currentMonth, 7): Colors.red[100]!,
      DateTime(currentYear, currentMonth, 13): Colors.orange[100]!,
      DateTime(currentYear, currentMonth, 14): Colors.red[100]!,
      DateTime(currentYear, currentMonth, 21): Colors.red[100]!,
      DateTime(currentYear, currentMonth, 22): Colors.green[100]!,
      DateTime(currentYear, currentMonth, 28): Colors.red[200]!,
    };

    return Scaffold(
      appBar: AppBar(
        title: const Text('My Interactive Calendar'),
        backgroundColor: Colors.blueGrey[800],
        foregroundColor: Colors.white,
      ),
      body: Padding(
        padding: const EdgeInsets.all(16.0),
        child: Column(
          children: [
            Text(
              '${monthNames[currentMonth - 1]} $currentYear',
              style: const TextStyle(fontSize: 22, fontWeight: FontWeight.bold),
            ),
            const SizedBox(height: 20),
            Center(
              child: MonthlyCalendar(
                year: currentYear,
                month: currentMonth,
                width: MediaQuery.of(context).size.width / 2,
                dayContents: monthEvents,
                dayColors: monthColors,
                selectedDay: _selectedDate,
                onDaySelected: (date) {
                  setState(() {
                    _selectedDate = date;
                  });
                  ScaffoldMessenger.of(context).hideCurrentSnackBar();
                  ScaffoldMessenger.of(context).showSnackBar(
                    SnackBar(
                      content: Text('Selected date: ${date.day}/${date.month}/${date.year}'),
                      duration: const Duration(seconds: 1),
                    ),
                  );
                },
              ),
            ),
            const SizedBox(height: 20),
            Text(
              _selectedDate == null
                  ? 'No day selected'
                  : 'Selected day: ${_selectedDate!.day}/${_selectedDate!.month}/${_selectedDate!.year}',
              style: const TextStyle(fontSize: 18, fontWeight: FontWeight.w500),
            ),
          ],
        ),
      ),
    );
  }
}
```

#### `lib/widgets/monthly_calendar.dart`

```dart
import 'package:flutter/material.dart';

class MonthlyCalendar extends StatelessWidget {
  final int year;
  final int month;
  final double width;
  final Map<DateTime, Widget>? dayContents;
  final Map<DateTime, Color>? dayColors;
  final void Function(DateTime)? onDaySelected;
  final DateTime? selectedDay;

  const MonthlyCalendar({
    super.key,
    required this.year,
    required this.month,
    required this.width,
    this.dayContents,
    this.dayColors,
    this.onDaySelected,
    this.selectedDay,
  }) : assert(month >= 1 && month <= 12, 'Month must be between 1 and 12.');

  @override
  Widget build(BuildContext context) {
    final int daysInMonth = DateTime(year, month + 1, 0).day;
    final firstDayOfMonth = DateTime(year, month, 1);
    final int startingWeekday = (firstDayOfMonth.weekday % 7) + 1;

    final Map<int, Widget> filteredContents = {};
    if (dayContents != null) {
      for (final event in dayContents!.entries) {
        if (event.key.year == year && event.key.month == month) {
          filteredContents[event.key.day] = event.value;
        }
      }
    }

    final Map<int, Color> filteredColors = {};
    if (dayColors != null) {
      for (final color in dayColors!.entries) {
        if (color.key.year == year && color.key.month == month) {
          filteredColors[color.key.day] = color.value;
        }
      }
    }

    final List<int?> daysToDisplay = _generateDayList(
      startingWeekday,
      daysInMonth,
    );
    final double cellSize = width / 7;

    return SizedBox(
      width: width,
      child: Column(
        children: [
          _WeekdayHeader(cellSize: cellSize),
          const SizedBox(height: 4),
          _CalendarGrid(
            year: year,
            month: month,
            days: daysToDisplay,
            cellSize: cellSize,
            dayContents: filteredContents,
            dayColors: filteredColors,
            onDaySelected: onDaySelected,
            selectedDay: selectedDay,
          ),
        ],
      ),
    );
  }

  List<int?> _generateDayList(int startingWeekday, int daysInMonth) {
    final List<int?> days = [];
    final int emptyDaysAtStart = startingWeekday - 1;
    for (int i = 0; i < emptyDaysAtStart; i++) {
      days.add(null);
    }
    for (int i = 1; i <= daysInMonth; i++) {
      days.add(i);
    }
    return days;
  }
}

class _WeekdayHeader extends StatelessWidget {
  final double cellSize;
  const _WeekdayHeader({required this.cellSize});

  @override
  Widget build(BuildContext context) {
    const weekdays = ['S', 'M', 'T', 'W', 'T', 'F', 'S'];
    return Row(
      children: weekdays.map((day) {
        return SizedBox(
          width: cellSize,
          child: Text(
            day,
            textAlign: TextAlign.center,
            style: const TextStyle(
              fontWeight: FontWeight.bold,
              color: Colors.black54,
            ),
          ),
        );
      }).toList(),
    );
  }
}

class _CalendarGrid extends StatelessWidget {
  final int year;
  final int month;
  final List<int?> days;
  final double cellSize;
  final Map<int, Widget>? dayContents;
  final Map<int, Color>? dayColors;
  final void Function(DateTime)? onDaySelected;
  final DateTime? selectedDay;

  const _CalendarGrid({
    required this.year,
    required this.month,
    required this.days,
    required this.cellSize,
    this.dayContents,
    this.dayColors,
    this.onDaySelected,
    this.selectedDay,
  });

  @override
  Widget build(BuildContext context) {
    return GridView.builder(
      shrinkWrap: true,
      physics: const NeverScrollableScrollPhysics(),
      gridDelegate: const SliverGridDelegateWithFixedCrossAxisCount(
        crossAxisCount: 7,
      ),
      itemCount: days.length,
      itemBuilder: (context, index) {
        final day = days[index];

        if (day == null) {
          return const SizedBox.shrink();
        }

        final Widget? dayContent = (dayContents != null && dayContents!.containsKey(day)) ? dayContents![day] : null;
        final Color backgroundColor = (dayColors != null && dayColors!.containsKey(day)) ? dayColors![day]! : Colors.grey[200]!;

        final bool isSelected = selectedDay != null &&
            selectedDay!.year == year &&
            selectedDay!.month == month &&
            selectedDay!.day == day;

        return Material(
          color: Colors.transparent,
          child: InkWell(
            borderRadius: BorderRadius.circular(4),
            onTap: () {
              if (onDaySelected != null) {
                final clickedDate = DateTime(year, month, day);
                onDaySelected!(clickedDate);
              }
            },
            child: Container(
              height: cellSize,
              width: cellSize,
              margin: const EdgeInsets.all(2),
              decoration: BoxDecoration(
                color: backgroundColor,
                borderRadius: BorderRadius.circular(4),
                border: isSelected
                    ? Border.all(color: Colors.blueGrey[700]!, width: 2.5)
                    : Border.all(color: Colors.black12, width: 1),
              ),
              child: Stack(
                children: [
                  Positioned(
                    top: 4,
                    right: 4,
                    child: Text(
                      day.toString(),
                      style: const TextStyle(
                        fontSize: 12,
                        fontWeight: FontWeight.w500,
                        color: Colors.black54,
                      ),
                    ),
                  ),
                  if (dayContent != null)
                    Center(
                      child: dayContent,
                    ),
                ],
              ),
            ),
          ),
        );
      },
    );
  }
}
```

### Conclusion

We have built a cohesive and decoupled component system. The main screen manages state and data, while the calendar widget focuses exclusively on displaying that data and reporting user interactions.

This architecture of passing data down and emitting events up (through callbacks) is one of the most fundamental and powerful patterns in Flutter development, allowing for the creation of complex yet organized and maintainable UIs.