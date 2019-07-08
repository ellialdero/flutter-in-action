> **In this chapter:**
>* Dart unit
>* Mocking HTTP calls
>* Flutter Widget testing
>* Flutter integration testing
>* Accessibility Widgets in Flutter

In the roughly 100 interviews I’ve endured in my life, one of the most common questions the interview asks is "what kind of testing do you do at your current job?". Trying to mask my inadequacy with humor, and hoping that the interviewer will just forget they care about testing code and move on, I always say, "Not as much as we should!" Which can be translated to "not at all". The only feeling of comfort I get from that is knowing that I’m not alone. (And, for the record, it’s never worked.)

The reason they ask about that, I think, is because it’s important, but often an afterthought. And, that’s what this final chapter is about: subjects that are important, but often forgotten. First, I want to talk about testing Flutter apps. Towards the end, I’ll cover some built-in accessibility features in Flutter. You can think of this chapter as "things you should absolutely do for a production app, but probably aren’t needed for projects that’ll never leave your machine."

#### 11.1  Testing Flutter apps
Testing in Flutter can be split into three categories:

1. Dart unit tests
When you need to test classes or functions, but there are no widgets involved. I’ll use the `Mockito` package for this to test HTTP calls.
2. Widget tests
When you want to do simple tests on widgets.
3. Integration tests
Tests that can move through your app like a user would, and make sure large features work, as well as provide performance feedback.'

To run these tests, I’ll continue to use the simple Todo App from the last chapter. I understand the argument for testing a more "robust" app, but I think it’s helpful to work with an app that won’t bog you down in the source code.

![](https://dpzbhybb2pdcj.cloudfront.net/windmill/v-7/Figures/todo_app_eleven.png)

#### 11.1.1  Setup for Dart Unit tests
The first set of tests have nothing to do with Flutter directly. You could test any Dart web app or server with the same approach. These unit tests are best used to test a single class or method. In the context of Flutter, you’ll likely be testing controllers, blocs, models, or utility functions.

Unit tests can also test dependencies using the `mockito` package. This package basically lets you create "mock" classes that call out to HTTP, or any other external source. It’s quite handy.

First, the proper dependencies need to be added to a test. For this section, two testing dependencies matter:
```dart
Listing 11.1. Todo app pubspec dependencies

// backend/pubspec.yaml
// backend/pubspec.yaml
dev_dependencies:
  flutter_test:
    sdk: flutter
  flutter_driver:
    sdk: flutter
  build_runner: ^1.0.0
  json_serializable: ^2.0.0
  mockito: 4.0.0
  test: any
```
After you’ve added those to the `pubspec.yaml` file and ran `flutter pub get`, you can create the test file. The file should live in a directory called `test` in the root of your project.
```
| backend
├── lib
├── pubspec.lock
├── pubspec.yaml
├── test
│   ├── dart_test.dart
```

That’s all the set up needed to start writing tests.

**The code to test**
I purposefully made some functionality in this app contrived, so we can talk through some examples of the basic dart unit tests. The code will get more complicated and "correct" as we move through the chapter.

The feature that this code corresponds to is the "completed todo counter" in the app bar to the right-hand side.

![](https://dpzbhybb2pdcj.cloudfront.net/windmill/v-7/Figures/completed_todos_counter.png)

The state of that feature is controlled in the `todos_controller.dart` file, by two classes. The controller itself, and a class called `CompletedTodoCounter`. This second class is the contrived code, and what I want to test first. Here’s the code we care about:
```dart
Listing 11.2. Testable code from the todos controller

// backend/lib/controllers/todo_controller.dart
class CompletedTodoCounter {
  int completed = 0;
  void increaseCounter() => completed++;
  void decreaseCounter() => completed--;
  void resetCounter() => completed = 0;
}

class TodoController {
  var counter = CompletedTodoCounter();

  // ... other class methods

  int getCompletedTodos() {
    counter.resetCounter();
    todos?.forEach((Todo t) {
      if (t.completed) {
        counter.increaseCounter();
      }
    });
    return counter.completed;
  }
}
```

The reason this is contrived is because there’s way too much code to simply keep a count. But, the reason I’ve done it this way will make sense when we move to using `mockito`.

#### 11.1.2  Writing Dart unit tests
Dart unit tests rely on three main functions from the `test` lib: `test`, `expect`, and `group`. The easiest way to explain how to write tests is to show some examples. This isn’t the full example, but an example of a single test. The most basic test possible. If you’ve written tests any other modern language, this will be pretty easy to read.

```dart
Listing 11.3. Basic unit test in Dart
// backend/test/dart_test.dart
test("counter increases", () {
    final counter = CompletedTodoCounter();
    counter.increaseCounter();

    expect(counter.completed, 1);
// ...
```

This test is basically saying "Run a new test called 'counter increases', and then test it by calling this function that I’ve provided. After creating the instance and calling `increaseCounter`, the `counter.completed` property should evaluate to 1."

The `expect` function works by taking in an `actual` as the first argument, which is what you’re testing, and a `matcher`, which is the expected outcome for the actual. If the two are the same, then the test will pass. Other wise, `expect` will throw an error and fail the test.

![](https://dpzbhybb2pdcj.cloudfront.net/windmill/v-7/Figures/expect_method.png)

Test’s can be as complicated as you’d like, though. You can have multiple expect function calls in a single test. Consider this example:

```dart
Listing 11.4. Multiple expect calls in a test

// backend/test/dart_test.dart
test("counter increases and decreases", () {
    final counter = CompletedTodoCounter();

    counter.increaseCounter();
    expect(counter.completed, 1);

    counter.increaseCounter(); // +1
    counter.decreaseCounter(); // -1
    counter.increaseCounter(); // +1
    counter.increaseCounter(); // +1
    counter.decreaseCounter(); // -1
    expect(counter.completed, 2);
});
```

**Use "group" for multiple tests**
If you have multiple tests that are testing the same function or class, you can "group" them together, and they can use the same resources. Here’s the final example of basic Dart unit tests:

```dart
Listing 11.5. Full unit test example

void main() {
    group("counter keeps track of completed todos", () {
        final counter = CompletedTodoCounter();
        test("counter increases and decreases", () {
            counter.increaseCounter();
            expect(counter.completed, 1);

            counter.increaseCounter(); // +1
            counter.decreaseCounter(); // -1
            counter.increaseCounter(); // +1
            counter.increaseCounter(); // +1
            counter.decreaseCounter(); // -1
            expect(counter.completed, 2);
        });

        test("counter resets to 0", () {
            counter.increaseCounter();
            counter.increaseCounter();
            counter.increaseCounter();
            counter.increaseCounter();

            counter.resetCounter();

            expect(counter.completed, 2);
        });
    });
}
```

The advantage to using groups with many tests, rather than less tests with many `expect` calls, is that tests are more modular this way. Some tests can pass, and some can fail. In this example, it’s reasonable that the `increaseCounter` and `decreaseCounter` methods work, but the `resetCounter` doesn’t. Testing this way would catch that.

#### 11.1.3  Using Mockito to test methods that need external dependencies
In our test app, when using Http, we want to make sure that the app behaves how we want regardless of the response of the Http call. For example, maybe the API that we get todo data from is down, and sending a `404` return message. That’s a bummer, but there’s nothing we can do about it. And, we should plan on it happening from time to time. So, we can "mock" all the situations we are concerned about possibly occurring.

You can handle this by writing alternative, "mock" versions of classes. Recall that the services in this app are all based off of an abstract class called `Services`. You could create a new implementation of that class that doesn’t do anything. I’ve actually done that (for a later test). It looks like this:

```dart
Listing 11.6. Mock services class

// backend/lib/controllers/todo_controller.dart
class MockServices implements Services {
    // ...
  @override
  Future<List<Todo>> getTodos() async {
    return [
      new Todo(1, 1, "delectus aut autem", false),
      new Todo(1, 2, "quis ut nam facilis et officia qui", false),
      new Todo(1, 3, "fugiat veniam minus", false),
      new Todo(1, 4, "et porro tempora", true),
      new Todo(1, 8, "et porro tempora", true),
      new Todo(1, 9, "et porro tempora", true),
      new Todo(1, 10, "et porro tempora", true),
      new Todo(1, 11, "et porro tempora", true),
      new Todo(1, 12, "et porro tempora", true),
      new Todo(1, 13, "et porro tempora", true),
      new Todo(1, 14, "et porro tempora", true),
      new Todo(1, 15, "et porro tempora", true),
      new Todo(1, 16, "et porro tempora", true),
    ];
  }
  // ...
```
That’s great, but can quickly become unruly if you have a big app with hundreds of classes. That’s where `mockito` comes in. It basically fakes these classes for you. We’ll use it to mock the `getTodos` call on the services class.

For the sake of simplicity, I’ve copied the `getTodos` method into the `http_test.dart` file, so it’s all in one place. This is what the `getTodos` function looks like:

```dart
Listing 11.7. getTodos makes an http request

// backend/test/http_test.dart
Future<List<Todo>> getTodos(Client client) async {
  final response =
      await client.get('https://jsonplaceholder.typicode.com/todos');

  if (response.statusCode == 200) {
    var all = AllTodos.fromJson(json.decode(response.body));
    return all.todos;
  } else {
    throw Exception('Failed to load todos');
  }
}
```
Using the `mockito` package is easiest demonstrated with code:
```dart
// backend/test/http_test.dart
class MockClient extends Mock implements Client {}

void main() {
  group('getTodos', () {
    test('returns a list of todos if the http call completes', () async {
      final client = MockClient();
      when(client.get('https://jsonplaceholder.typicode.com/todos'))
          .thenAnswer((_) async => Response('[]', 200));

      expect(await getTodos(client), isInstanceOf<List<Todo>>());
    });
```
A bulk of whats happening is in the `when/thenAnswer` call, so I’d like to break that down.

This test is going to call `getTodos`, but rather than passing in a "real" `Client` from the `dart:http` library, it’s passing in a "mock client". This mock client knows that it should fake whichever calls are made on it *that we tell it to* from the `when` function.

As a reminder, the `when` function call looks like this:
```dart
when(client.get('https://jsonplaceholder.typicode.com/todos'))
           .thenAnswer((_) async => Response('[]', 200));
```

So, whenever the `get` method is called on the mocked `Client` class, it knows to return the `Response('[]', 200));`, rather than actually making the Http call and responding. Pretty neat.

Here’s another test that mocks a failed call:
```dart
// backend/test/http_test.dart
test('throws an exception if the http call completes with an error', () {
      final client = MockClient();
      when(client.get('https://jsonplaceholder.typicode.com/todos'))
          .thenAnswer((_) async => Response('Not Found', 404));
      expect(getTodos(client), throwsException);
    });
```

The point of `mockito` is to test that your app responds gracefully to external dependencies that it can’t control. It’s a powerful tool to carry in your tool-belt.

#### 11.1.4  Flutter widget tests
Widget tests in Flutter build on Dart unit tests. Rather than using the `test` package, you use the `flutter_test` package. The API is the same, but there are additional methods to test widgets and UI.

In general, the testing goes like this:
1. Tell the test which widget to use as the entry point. The test will build that widget and it’s sub-tree.
2. "find" widgets in that subtree.
3. Test that certain properties in this widget tree are true.

It isn’t so different, but there are a couple caveats and some new jargon to learn. In this section, I’ll first show you a code example of a widget test. And that will include some weird jargon that you probably won’t know. That’s okay, because this code example is just here to provide context. Then, the jargon will be easier to explain. But first, of course, we have to do some set up.

**Set up Flutter tests**

The app is already set up to do tests, and there’s only really one difference in the way you set up widget tests. Add the right libraries to your `pubspec.yaml`.
```dart
// backend/pubspec.yaml
dev_dependencies:
  flutter_test:
    sdk: flutter
  flutter_driver:
    sdk: flutter
  build_runner: ^1.0.0
  json_serializable: ^2.0.0
  mockito: 4.0.0
  test: any
```

That’s all there is to set up this portion, actually.

**Writing widget tests**

First, take a look at this example. This test ensures that there is a title in the `AppBar` of the widget.
```dart
Listing 11.8. Basic widget test function

// backend/test/widget_test.dart
  testWidgets('App has a title', (WidgetTester tester) async {
    var services = new MockServices();
    await tester.pumpWidget(TodoApp(controller: new TodoController(services)));
    final titleFinder = find.text("HTTP Todos");
    expect(titleFinder, findsOneWidget);
  });
```

You can see that the general approach to writing tests is the same. The difference is that you have to use some `flutter_test` specific methods and objects to ensure the tests work. I’ll walk through them here.

1. Finder object
Finders are objects that scan the widget tree and find widgets by specific properties. The `find` object is collection of many `CommonFinders`, including `byText`, `byWidget`, `byKey`, and `byType` (among others). You use these finders almost all the time. The `find.byText` call that I’m using in the above example literally searches for a widget with that exact text.

2. `Matcher` objects
Matchers are the same as they were in the unit tests. They provide a way to compare the actual case with what you expect. Because widgets are more complicated than simple values (like an `int`, from the unit test example earlier in the chapter), the `flutter_test` library comes with a number of built in matchers that do the hard work for you. The most common matchers are `findsNothing`, `findsOneWidget`, and `findsNWidgets(int n)`. Testing widgets is largely about making sure they exist, which is where these come in.

3. `WidgetTester`
The `WidgetTester` class is the base for all the `flutter_test` functionality. It’s purpose is to interact with widgets the same way that users would. It can build widgets, tap on widgets, and more.
4. `WidgetTester.pump` (and similar methods)
There are a collection of methods that involve "pumping" the widget tester. The simplest way to describe these methods is that they call "Widget.build". Any time anything in the test imitates user interaction, you need to `pump` the app before doing anything else. This will make more sense when you see the code sample below.

With all that in mind, let me walk through that last code sample again.

```dart
Listing 11.9. Basic widget test function

// backend/test/widget_test.dart
  testWidgets('App has a title', (WidgetTester tester) async {
    var services = new MockServices();
    await tester.pumpWidget(TodoApp(controller: new TodoController(services)));
    final titleFinder = find.text("HTTP Todos");
    expect(titleFinder, findsOneWidget);
  });
```

The long-short of it is that Flutter tests aren’t much different than Dart unit tests, so long as you know the API to interact with widgets. But, the `flutter_test` package provides many more ways to interact with widgets. You can do things like `tap` on buttons, drag on the screen, and input text into input fields.

Consider the Todo App from this chapter, and how it works. When you load the app, it does nothing. But, after tapping the `FloatingActionButton`, the app makes an Http call to get some data, and then displays that data in the form of a todo list.

To test this that that button makes a call and renders the list, we need to have the test suite "tap" the `FloatingActionButton`. That’s perfectly possible, and rather easy in Flutter.

This following sample shows how that works, as well as finding widgets by keys.

```dart
Listing 11.10. Finding and tapping on a button

// backend/test/widget_test.dart
testWidgets('finds and taps the floating action button', (WidgetTester tester) async {
    var services = new MockServices();
    await tester.pumpWidget(TodoApp(controller: new TodoController(services)));
    Finder floatingActionButton = find.byKey(new Key('get-todos-button'));
    await tester.tap(floatingActionButton);

    // rebuild the app
    await tester.pumpAndSettle(Duration(seconds: 2));
    final firstTodoFinder = find.text("delectus aut autem");
    expect(firstTodoFinder, findsOneWidget);
});
```

There are two things to note before I move on integration tests. First, finding elements by key’s is easy, and what I recommend when you’re looking for a single widget. In the above example, I find the `FloatingActionButton` by key. The code in the app that looks like this just has a simple `Key` on it: ` .Finding widgets by keys in tests

```dart
// backend/lib/todo_page.dart -- line ~ 78
floatingActionButton: FloatingActionButton(
  key: Key("get-todos-button"),
  onPressed: () => _getTodos(),
  child: Icon(Icons.add),
),

// And look for it with a test
// backend/test/widget_test.dart
testWidgets('finds and taps the floating action button', (WidgetTester tester) async {
  var services = new MockServices();
  await tester.pumpWidget(TodoApp(controller: new TodoController(services)));
  Finder floatingActionButton = find.byKey(new Key('get-todos-button'));
```
That’s pretty straight forward, I think.

The other notable piece of the test above is all the references to `pump`. There are two things to keep in mind: 1. The widget you’re testing must be wrapped in an `App` and `Scaffold`, because they provide objects like `MediaQuery` and other context to the test. Which means you should be passing in the whole app when you "pump" it. If you don’t want to pump a whole, giant app, you can make a tiny test app that wraps the widget you want to test in a `MaterialApp` and `Scaffold`. 2. You have to pump the widget everytime you want `Widget.build` to be called! So after any tap or other interaction, you have to `pump`. After any call to `setState` in the your widgets, you have to `pump`!

These tests are ideal for simple widget tests, but can be cumbersome if you’re trying to test a full UI workflow. Luckily, there are some nice tools you can use to write integration tests, because Flutter thought of everything.

#### 11.1.5  Flutter integration tests
It’s no secret by now that I’m a big fan of the problems that Flutter has solved for you. Integration testing is no exception. There’s a nice library, built by the Flutter team, that provides a nice way to run integration tests. It makes it easy to test how everything works together, and profiles the performance of your app.

This is done with the `flutter_driver` package. Tests with Flutter driver are written similarly to widget tests, but the package is really made to simulate a user. Driver tests actually run the app, and then interact with it. You can watch it scroll through your app and tap buttons that you’ve described. If you’ve used something like Selenium Driver before, this is similar. If you haven’t, prepare to be wowed.

**Setup Flutter Driver**

`flutter_driver` is a bit pickier about set up than previous tests we’ve looked at. But, it’s still straight-forward enough. First, of course, you need to import the package in your `pubspec.yaml`.
```dart
Listing 11.11. Todo app pubspec dependencies

// backend/pubspec.yaml
dev_dependencies:
  flutter_test:
    sdk: flutter
  flutter_driver:
    sdk: flutter
  build_runner: ^1.0.0
  json_serializable: ^2.0.0
  mockito: 4.0.0
  test: any
```
After you’ve imported those dependencies, you need to create the relevant files. The driver files should go in their own, separate directory from the rest of the tests. By convention, the directory is usually called `test_driver`. It’s located in the root of the Flutter project.

Then, you need to add two files. One to run the app, and one to run the tests. Remember, the app actually runs when you use `flutter_driver`. It must be running in a different process, and therefor must be in a different file.

This is what the project will end up looking like:
```
| backend
├── lib
├── pubspec.lock
├── pubspec.yaml
├── test
│   ├── dart_test.dart
├── test_driver
│   ├── app.dart
│   ├── app_test.dart
```

The file that runs the app (`app.dart`), is only a few lines of code. It looks like this:
```
Listing 11.12. Set up the flutter_driver extension

// backend/test_drive/app.dart
import 'package:flutter_driver/driver_extension.dart';
import 'package:backend/main.dart' as app;

void main() {
  // This line enables the extension
  enableFlutterDriverExtension();
  // run the app
  app.main();
}
```

That’s all there is to that file. Of course, in really complicated apps, you may have to do some set up in this file, like anything that needs to be pre-processed in the app.

Lastly, you have to have a device connected to run the app on. Whatever you usually use will do fine. (iOS emulator, Android studio emulator, or an actual connected device). But, the app literally runs, so it has to have a place to run.

**Write tests for flutter_driver**

The tests are in the `test_app.dart` file. Again, the tests aren’t too different. You use Finders (though they’re called slightly differently), Matchers, and use the same `expect` calls.

The difference mainly lies in the setup. You have access to special functions that are used to connect to Flutter driver. This is the setup:
```dart
Listing 11.13. Writing flutter_driver integration tests

// backend/test_drive/app.dart
import 'package:flutter_driver/flutter_driver.dart';
import 'package:test/test.dart';

void main() {
  group("Todo App", () {
    final buttonFinder = find.byValueKey("get-todos-button");
    final completedTodoCounter = find.byValueKey("counter");
    final listViewFinder = find.byValueKey("list-view");
    final lastTodoFinder = find.byValueKey("todo-19");
    final lastTodoSubtitleFinder = find.byValueKey('todo-19-subtitle');

    FlutterDriver driver;

    // Connect to the Flutter driver before running any tests
    setUpAll(() async {
      driver = await FlutterDriver.connect();
    });

    // Close the connection to the driver after the tests have completed
    tearDownAll(() async {
      if (driver != null) {
        driver.close();
      }
    });
```
That’s all there is to set up. Basically, connect to the driver before you run the tests, and close the connection when you’re done. Closing the connection effectively just stops running the app.

For the tests themselves, I decided to test two things. One, that `FloatingActionButton` can be tapped, and will result in fetching todos. I’m testing that by checking the count of completed todos. (This only works because I know the test data. Otherwise, it wouldn’t be safe to assume there are always more than 0 completed todos in the data set. But, since I’m using a mock API, I know that the data is going to be the same every time.)

![](https://dpzbhybb2pdcj.cloudfront.net/windmill/v-7/Figures/completed_todos_counter.png)

Secondly, I’m testing to make sure that it scrolls, and that there are the number of items in the todo lists that I expect. This test is about showing two things: 1. The power of `flutter_driver` (you can watch it scroll through your app), and two, how to use `flutter_driver` package to profile your apps performance. I’ll discuss that more shortly.

This is what the tests look like that I wrote for the todo app:

```dart
//lib/test_drive/test_app.dart
 test('taps fab button', () async {
      await driver.tap(buttonFinder);
      expect(await driver.getText(completedTodoCounter), isNot("0"));
    });

    test('can scroll to bottom', () async {
      final timeline = await driver.traceAction(() async {
        await driver.scrollUntilVisible(
          listViewFinder,
          lastTodoFinder,
          dyScroll: -150.0,
        );
        expect(await driver.getText(lastTodoSubtitleFinder), "todo num: 19");
```

There are basically three big differences in that test compared to the widget tests. First `tester` is replaced with `driver`. But, one or the other is the object you’re using to access most of the test features.

The second is less subtle. The `scrollUntilVisible` method on driver is powerful! You actually can scroll with the normal `flutter_test` package, but it’s not as "smart". `scrollUntilVisible` does much more for you. In the test example above, the test will pass if it can tap the button, display some todos, and scroll down until it finds a todo with the Key `todo-19`. If there were 5,000 todos, it would be able to handle that, too. It just scrolls a bit, checks to see if the item is visible, and then continues.

`scrollUntilVisible` is just one example of what `flutter_driver` can do. It makes it easy to simulate actual user usability.

The last big difference in this test (compared to widget tests), is the `driver.traceAction` method. Which is used to profile your app’s performance, and I’ll talk about next.

#### 11.1.6  Performance profiling integration tests

The reason that I specifically made a test that deals with scrolling is because scrolling is a costly task for a mobile app. While scrolling, the app is re-rendering several times per second. (Flutter renders at 60FPS, if you’re curious.) So, which you’re scrolling, your app is re-rendering the entire page over and over. It’s an animation that’s animated for you. Which means, it’s not too hard for scrolling to get "janky".

`flutter_driver` provides a way to keep your app jank-free, via it’s built in performance profiling. In a nutshell, you can tell your app to record performance metrics during any integration test, and it will give you the resulting data. The profiling is all about the UI itself, such as how smooth the animations are, what FPS the rendering is achieving, etc.

> NOTE
Profiling the app with `flutter_driver` only really requires two quick steps. First, tell it what test to profile with the `traceAction` method. Then, output the summary with an object called `TimelineSummary`. The test I showed you earlier (the scroll test) was actually in-complete. Here’s the full snippit.

```dart
Listing 11.14. Profiling you app with flutter_driver
// backend/test_drive/app_test.dart
test('can scroll to bottom', () async {
  final timeline = await driver.traceAction(() async {
    await driver.scrollUntilVisible(
      listViewFinder,
      lastTodoFinder,
      dyScroll: -150.0,
    );
    expect(await driver.getText(lastTodoSubtitleFinder), "todo num: 19");
  });
  final summary = new TimelineSummary.summarize(timeline);
  summary.writeSummaryToFile("scrolling_summary", pretty: true);
  summary.writeTimelineToFile("scrolling_timeline");

});
```

So, this is all cool. We can all agree that it’s neat that Flutter will profile your app for you. But reading the data is a different story. First, to actually run the test, do two steps:

1. Make sure you have a device running (i.e. the iOS emulator).
2. run `flutter drive --target=test_driver/app.dart` in the root of your project.

This’ll take a minute because it has to build the app and run it. Once the tests run, two files will have been generated, and both can be found in the `build` folder that’s generated whenever you run a Flutter app.

The first, `backend/build/scrolling_summary`, is full of JSON that gives some quick insights. Mine looks like this:

```dart
//backend/build/scrolling_summary.json
{
  "average_frame_build_time_millis": 4.57943661971831,
  "90th_percentile_frame_build_time_millis": 8.045,
  "99th_percentile_frame_build_time_millis": 10.86,
  "worst_frame_build_time_millis": 13.751,
  "missed_frame_build_budget_count": 0,
  "average_frame_rasterizer_time_millis": 2.4071690140845066,
  "90th_percentile_frame_rasterizer_time_millis": 2.85,
  "99th_percentile_frame_rasterizer_time_millis": 3.234,
  "worst_frame_rasterizer_time_millis": 3.551,
  "missed_frame_rasterizer_budget_count": 0,
  "frame_count": 71,
  // ...
```
To me, this is a little too much to digest. Those numbers really only have value when you’ve run many tests and can see the data points change over time. They don’t have much context on their own. But, you can see things `missed_frame_build_budget_count`, which is 0. That’s good. That means Flutter could handle every new frame it tried to render.

However, there is an even more detailed set of data that will make this easier to digest. And that’s the `scrolling_timeline.json` file.

On it’s own, it’s a mess. It’s so many numbers that it doesn’t make any sense. But, there’s a tool inside Google’s Chrome browser that can display this data in a graph. Digesting profiling data is beyond the scope of this book, but I will show you how to get there and a few hot tips.

The tool that turns the data into graphs is free. And, if you use the Chrome browser, you already have the tool. In the Chrome browser, you can go to the address `chrome://tracing/`. (Put that in the URL bar and navigate to the page.) This is what you’ll see.

![](https://dpzbhybb2pdcj.cloudfront.net/windmill/v-7/Figures/blank_profiler.png)

Basically a blank screen. But on that screen, click the `Load` button and open the `scrolling_timeline.json` file. Now, the data is loaded, and you’ll see this:

![](https://dpzbhybb2pdcj.cloudfront.net/windmill/v-7/Figures/profiler_visuals.png)

Now you can see graphs. This graph reminds me of the Chrome dev tools `Network` tab, if you come from the web. Basically it allows you to zoom in on moments of time and see what’s taking the most time for Flutter to handle.

![](https://dpzbhybb2pdcj.cloudfront.net/windmill/v-7/Figures/profiler_list_view.png)

The data tells you exactly how long each process took, which gives you an idea of what you can whittle down. Again, these profiling tools are much more important with context. Trying to make sense of them in a test app won’t be as useful as using them in real life. So with that in mind, I leave you knowing that the tools are there, and next time you’re building a Flutter app that seems janky, you know how to investigate it.

#### 11.2  Accessibility with the Semantics widgets

In general, accessibility is about making sure your app is usable by everyone. The best example of this is color usage on screens: A lot of people are colorblind, and it’s important that the colors you use in any Flutter app, web app, or literally anything else that humans have to read, are readable by everyone, even if their vision is impaired. For instance, using heavy contrast between text and it’s background.

This is a pretty short topic in Flutter, because most of principals and practices aren’t specific to Flutter (For example, using images with high contrast, or using large enough text). You could (and many people have) written full books about accessibility. I encourage you learn all that you can and use these practices.

That said, Flutter does have one family of widgets that are specifically used for accessibility: the `Semantics` widgets.

These widgets are used in the same way the `alt` property on the `<a>` tag in HTML is used. It provides information about what the app does and how it works. The `Semantics` widget annotates the widget tree with a description of it’s direct child. For example, you can use semantics to annotate a `Button` widget with what the button does. Or, you can use it to annotate certain text that may be challenging for a visually-impaired person to read. This widget allows a screen-reader to be able to decipher what’s going on, and report back to the user.

In Flutter, as we’ve learned, there are multiple trees that represent the app. This is how the `Semantics` widget works, too. There’s (yet another) tree called the semantics tree that holds the semantics information for screen readers. Some of the work of building a good semantics tree is done for you, but you can add to it yourself, too.

You add nodes to the semantics tree by simply wrapping widgets with a `Semantics` widget as a parent. Here’s an example:
```dart
Semantics(
  container: true,
  properties: SemanticsProperties(
      button: true,
      hint: "Performs action when pressed",
      onTap: () => { ... },
  ),
  child: Button(
  // ...
```
This is a bulk of what you need to keep in mind when it comes to using `Semantics` widgets. There are many, many properties on the widget. But, the important thing to note here is that Flutter does consider accessibility, provide a way to make apps more accessible, and you should absolutely use them to make your app production ready.

#### 11.3  Wrap up
(Cue "You’re Still the One" by Shania Twain on the jukebox.)

"Looks like we made it…"

(Credits roll…)

Welp, here we are. At the end of the book. And look how far we’ve come. As I’m writing this, I’m happy to say that there are some big, exciting things coming in the near future for Flutter. So, you’ll probably hear from me again, soon.

It’s funny, this is the end of the testing chapter. So, I’m probably writing this for no one. No one reads the testing chapter. Perhaps I should’ve started the book with testing, because it’s better to burn out then fade away. But, then you likely would’ve just skipped the chapter. Anyways, testing is important and I hope you take advantage of these tools.

#### 11.4  Summary
* There are three ways to test Flutter code: Dart unit tests, widget tests, and integration tests.
* Unit tests are great for testing classes and functions.
* Widget tests are best used to test specific widgets.
* Integration tests can test how all the moving pieces of a feature work together.
* You can profile the performance of your app with integration tests.
* Integration tests in Flutter are done with the `flutter_driver` package.
* Accessibility is important in production apps.
* Flutter helps you write more accessible apps via the `Semantics` widget. Use it!
