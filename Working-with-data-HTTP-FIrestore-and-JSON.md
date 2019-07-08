> **In this chapter:**
>* Serialize JSON data
>* Using HTTP to talk to a backend
>* Using Firebase as a backend
>* Using Firestore NoSQL database
>* Dependency injection

At this point in the book, if you’ve been following along in order, you’re ready to build full, production apps in Flutter. Truly, you’re finished! If you work at a company that’s considering building a Flutter app, you have all the information you need to start that project.

But, there are an infinite number of topics that, although similar in Flutter to all SDKs, are pertinent writing good software. For the rest of the book, I’m going to depart from Flutter focus on topics you need to leverage in any mobile app, but they aren’t (necessarily) Flutter specific.

Particularly, you probably want to know how to work with a backend or data store. And, to talk to almost any backend, you’ll probably want to turn Dart objects in JSON. That’s what this chapter is about.

With that in mind, the UI work for the remainder of the book is light. In fact, the app that I’m going to make in this chapter looks like this:
![](https://dpzbhybb2pdcj.cloudfront.net/windmill/v-7/Figures/todo_app.png)

Pretty plain. But, there’s a lot to learn.

#### 10.1  HTTP and Flutter
While writing this book and thinking about what apps to build as the examples, I’ve tried my hardest to leave as much "set-up" out of the apps as possible. And that continues in this section, because I’m using a free, killer JSON-placeholder-test-API called Typicode to get some data over the network. [1] So, there’s no backend code to write or database’s to set up.

The goal for this first part of the chapter is to make an HTTP "GET" request and "POST" request, to simulate using a "real" backend. Using Typicode, I’m going to fetch a list of "to-dos", turn it into Dart objects we can use, and then render them to the screen. I’ll also write a "POST" request, which will update the todos when marked "complete".

In general, this requires 4 steps:

1. Adding the Flutter `http` package to your project.
2. Get the todos with the `http` package.
3. Convert the JSON response into Dart.
4. Display the data using a `ListView`.

#### 10.1.1  HTTP package
The Flutter `http` package makes it super easy on us. To start, it must be added to the `pubspec.yaml`.

```dart
// backend/pubspec.yaml -- line ~19
dependencies:
  http: ^0.12.0+2
```

Once a dependency is added, you can grab it from the internet by running `flutter packages get` from the project root (in this case `backend`).

#### 10.1.2  GET request
Now that it’s installed, you can see how it’s used in the services directory of the app.
```dart
Listing 10.1. Making an HTTP GET request

// backend/lib/services/todos.dart -- line ~13
class HttpServices implements Services {
  Client client = new Client();

  Future<List<Todo>> getTodos() async {
    final response = await client.get('https://jsonplaceholder.typicode.com/todos');

    if (response.statusCode == 200) {
      // If the call to the server was successful, parse the JSON
      var all =  AllTodos.fromJson(json.decode(response.body));
      return all.todos;
    } else {
      // If that call was not successful, throw an error.
      throw Exception('Failed to load todos ');
    }
  }
 }
```

 That probably seems pretty simple — and it is. There isn’t much to making requests. (Of course, this JSON server doesn’t require headers or authentication, so this is a basic example). The point is, making `http` requests in Flutter (and Dart server side apps) is pretty easy.

By far the part that was the most glossed over in that code example is the JSON serialization, which is definitely the more involved part, and I’ll cover now.

#### 10.1.3  JSON Serialization
First, I guess, what do we mean by `serialization`? In the the context of making network calls, it’s a term that means something like "converting objects in a programming language to a lightweight, standard data format that can be sent over the network."

Usually, that standard data format is JSON. So, I’m specifically talking about turning JSON into Dart objects and Dart objects back into JSON. In Flutter apps, you have a few options when it comes to serialization.

1. Manual serialization
2. Auto generated serialization using a package

I want to to talk about both. Manual serialization is worth doing in small apps with classes that aren’t too robust. And, if you aren’t familiar with serialization, seeing it done makes it crystal clear. That said, auto-generating classes with serialization is super easy, and you’ll almost always want to do that in the real world. If you’re a veteran app developer, you can probably just skip the section about auto-generation.

**Manual Serialization**
So far, I’ve only showed you one code snippit in this chapter: making an http GET request. That request returns a JSON object (which is actually just a string). It looks like this when we get it from the get request:

```dart
Listing 10.2. JSON object from getPosts call

// JSON
[
  {
    "userId": 1,
    "id": 1,
    "title": "delectus aut autem",
    "completed": false
  },
  {
    "userId": 1,
    "id": 2,
    "title": "quis ut nam facilis et officia qui",
    "completed": false
  },
  {
    "userId": 1,
    "id": 3,
    "title": "fugiat veniam minus",
    "completed": false
  },
  // ... more todos
```

It’s a "list" of "maps", but it’s reall just a string. We want to use that data in the Flutter app like this: (don’t worry about looking up this code snippit right now, we’ll come back to it).
```dart
Listing 10.3. Using todos in the UI

ListView.builder(
  itemCount: todos != null ? todos.length : 1,
  itemBuilder: (ctx, idx) {
    if (todos != null) {
      return CheckboxListTile(
        onChanged:(bool val) => updateTodo(todos[idx], val),
        value: todos[idx].completed,
        title: Text(todos[idx].title),
      );
    } else {
      return Text("Tap button to fetch todos");
    }
  });
```

This takes a few steps:

1. Parse the JSON string into a generic Dart object (like a `Map`).
2. turn that object into a specific type (`Todo`).

And this app has an extra step because the JSON is actually a `List` of `Map` types. For now, though, let’s just look at turning a single todo from the JSON into a `Todo`. That JSON will look like this:
```dart
{
    "userId": 1,
    "id": 1,
    "title": "delectus aut autem",
    "completed": false
  }
```
And, the `Todo` class looks like this:
```dart
Listing 10.4. Todo class

// backend/lib/models/todo.dart
class Todo {
  final int  userId;
  final int id;
  final String title;
  bool completed;

  Todo(this.userId, this.id, this.title, this.completed);

// ...
```
The first step, when writing the code, is actually to write the method that will turn a Dart `Map` into the `Todo`. We’ll worry about converting the string from the HTTP response into a map next.
NOTE
```dart
Listing 10.5. fromJson factory methods

// backend/lib/models/todo.dart
class Todo {
  final int  userId;
  final int id;
  final String title;
  bool completed;

  Todo(this.userId, this.id, this.title, this.completed);

  factory Todo.fromJson(Map<String, dynamic> json) {
    return Todo(
        json['userId'] as int,
        json['id'] as int,
        json['title'] as String,
        json['completed'] as bool
    );
  }
 }
```

This instance of a `fromJson` method is fairly simple. You’re literally just extracting properties from the map using square-bracket notation. Using the `as` keyword will ensure that properties are the correct type. (Or throw an error if they can’t be parsed into the right type.)

That’s most of what’s required (by the developer) to convert JSON into an object. You can probably guess that a big, complicated class would be much less fun to de-serialize manually.

There’s one more step, though, and it happens in the "GET" request we looked at earlier:
```dart
Listing 10.6. Parsing data out of an HTTP response

// backend/lib/services/todos.dart -- line 18
  Future<List<Todo>> getTodos() async {
    final response = await client.get('https://jsonplaceholder.typicode.com/todos');

    if (response.statusCode == 200) {
      // If the call to the server was successful, parse the JSON
      var all =  AllTodos.fromJson(json.decode(response.body));
 // ...
```

Recall that the `Todo.fromJson` factory method requires a `Map<String,dynamic>` type as an argument, but the data we get from the response is really a `String`. Part of the Dart standard library is a nice Json converter. Simply calling `json.decore(string)` will turn that into a `Map` for you.

**Auto generated Json serialization**
There are multiple packages that will generate Dart classes for you. The simplest, and the one I like to use, is called `json_serializable`. When using this package, you write classes as you always have, and you also write a `fromJson` and `toJson` method on those classes. Then, you run the package and it generates those `fromJson` and `toJson` methods for you. It’s pretty slick.

To use `json_serializable`, you actually need to add three dependencies to your project:
```dart
// backend/package.yaml -- line ~9
dependencies:
  flutter:
    sdk: flutter
  http: ^0.12.0
  json_annotation: ^2.0.0

dev_dependencies:
  flutter_test:
    sdk: flutter
  build_runner: ^1.0.0
  json_serializable: ^2.0.0
```
Once those are installed and `flutter packages get` has been run, you can start generating code.


**Updating the Todo class**
The `Todo` class in the project, which uses `json_serializable` actually looks like the following snippit, which has everything you need to generate some code.
```dart
import 'package:json_annotation/json_annotation.dart';

part 'todo.g.dart';

@JsonSerializable()
class Todo {
  final int  userId;
  final int id;
  final String title;
  bool completed;

  Todo(this.userId, this.id, this.title, this.completed);

  factory Todo.fromJson(Map<String, dynamic> json) => _$TodoFromJson(json);

  Map<String, dynamic> toJson() => _$TodoToJson(this);
}
```

If this was a new project, and the generated code didn’t exist yet, you’d have errors in that file, because `todo.g.dart` doesn’t exist yet, nor do the methods that are being called from that file. (In fact, if you’re following along with the source code, you delete the `todo.g.dart` file and watch the magic.)

To generate that file, you need to go to your terminal and run `flutter packages pub run build_runner build`. By running that command in the root of your project directory, the `build_runner` will find all classes that need some code generation, and generate it. It will create (or overwrite) the `todo.g.dart` file, which has this logic in it:
```dart
Listing 10.7. Code generated by the json_serialization package

// backend/lib/model/todo.g.dart
// GENERATED CODE - DO NOT MODIFY BY HAND

part of 'todo.dart';

// **************************************************************************
// JsonSerializableGenerator
// **************************************************************************

Todo _$TodoFromJson(Map<String, dynamic> json) {
  return Todo(json['userId'] as int, json['id'] as int, json['title'] as String,
    json['completed'] as bool);
}

Map<String, dynamic> _$TodoToJson(Todo instance) => <String, dynamic>{
  'userId': instance.userId,
  'id': instance.id,
  'title': instance.title,
  'completed': instance.completed
};
```
That’s all there is to it. If you have a big app, with robust classes, it much easier to run a command in the terminal then to parse maps on your own.

#### 10.1.4  Bring it all together in the UI
Now, that you know the app can grab data over the network, and you know that the data can be serialized into proper Dart classes, it’s time to bring it all together for its original purpose, to display that information in the UI.

For the sake of focusing on the task at hand, I chose to use, basically, no state management pattern. The information is fetched from a controller right from the widgets. There are three pieces of code involved here.
1. Todo controller
2. updates to main.dart
3. The widgets in todo_page.dart
   
**Todo Controller**

This is a class that basically acts as a messenger between the Http services and the widgets. It’s responsible for telling the UI what the todos are, and what to render. That’s a bit abstract, so let’s look at the code:

```dart
Listing 10.8. Todo controller

// backend/lib/controllers/todo.dart
class TodoController {
  final Services services;
  List<Todo> todos;

  StreamController<bool> onSyncController = new StreamController();
  Stream<bool> get onSync => onSyncController.stream;

  TodoController(this.services);

  Future<List<Todo>> fetchTodos() async {
    onSyncController.add(true);
    todos = await services.getTodos();
    onSyncController.add(false);
    return todos;
  }
 }
```
This controller method, in human English, is saying "Oh, UI, you want some data to render? Okay, then set your status to loading while I grab that data from the services." Then, some time passes. Then it says, "Okay, I got your data, you aren’t loading anymore, you can render this."

The point of this controller is basically to keep the UI as dumb as possible.

The UI knows about this controller because it’s passed in from `main.dart`.

**Create the controller in the main function**
The main function can be used for any "set up" that needs to be done before the app renders. In this case, we need to create the controllers and services that the app will use.
```dart
Listing 10.9. The root of the Flutter app

void main() async {
  var services = new HttpServices();
  var controller = new TodoController(services);

  runApp(TodoApp(controller: controller));
}

class TodoApp extends StatelessWidget {
  final TodoController controller;

  TodoApp({this.controller});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: TodoPage(controller: controller),
    );
  }
}
```

There isn’t much to this. I just wanted to show you this so that when I show you the widget, you know where it got it’s reference to the `controller`.

**Todo Page UI**
The todo page is a `StatefulWidget` that just grabs the todos and displays them in a list.

![](https://dpzbhybb2pdcj.cloudfront.net/windmill/v-7/Figures/todo_app.png)

The state object of this widget has three aspects.

1. It displays a `ListView` of todos.
2. It displays a `CircularProgressIndicator` instead if the todos are loading.
3. It fetches the todos when a button is tapped.

```dart
Listing 10.10. The Todo list page
// backend/lib/todo_page.dart -- line ~14
class _TodoPageState extends State<TodoPage> {
  List<Todo> todos;
  bool isLoading = false;

    void initState() {
      super.initState();
      widget.controller.onSync.listen((bool syncState) => setState(() {
            isLoading = syncState;
          }));
    }

  void _getTodos() async {
    var newTodos = await widget.controller.fetchTodos();
    setState(() => todos = newTodos);
  }
// ...
```
That’s the first half the `_TodoPageState` object. That’s the funcationality, so you have context for the widgets in the `build` method.
```dart
Listing 10.11. TodoPageState build

// backend/lib/todo_page.dart -- line ~37
Widget get body => isLoading
      ? CircularProgressIndicator()
      : ListView.builder(
          itemCount: todos != null ? todos.length : 1,
          itemBuilder: (ctx, idx) {
            if (todos != null) {
              return CheckboxListTile(
                onChanged:(bool val) => updateTodo(todos[idx], val),
                value: todos[idx].completed,
                title: Text(todos[idx].title),
              );
            } else {
              return Text("Tap button to fetch todos");
            }
          });

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text("Http Todos"),
      ),
      body: Center(child: body),
      floatingActionButton: FloatingActionButton(
        onPressed: () => _getTodos(),
        child: Icon(Icons.add),
      ),
    );
  }
```

So, this screen really has three states. "Loading", "Not loading, but theres no data", and "Not loading, and there is data."
![](https://dpzbhybb2pdcj.cloudfront.net/windmill/v-7/Figures/todo_app_progression.png)

This sample so far has been fairly straight forward if you’re both comfortable with Flutter and a vertan app builder. But, this isn’t necesarily the "Flutter" way. The Flutter team seems to be all about using Firebase as a backend. (To be clear and not mis-represent anyone, they didn’t say that, it’s just that all their examples and docs use Firebase, not "traditional" methods.)

So, because of that, the rest of this chapter will be devoted to using Firebase, rather than the Http package.

[1] The service is called Typicode JSON Placeholder, and can be found at <jsonplaceholder.typicode.com>

#### 10.2  Firebase and Flutter
Firebase is a cloud platform by Google that has a ton of great features. In the simplest terms, it’s a backend-as-a-service. It can handle auth, it has a databse, it can be used to store files like images, and more. It’s a pretty incredible product, really. But what we care about is the database service, which is called `Firestore`.

Firestore is a "NoSQL" database, like Mongo (but it’s also much more). SQL (and NoSQL) are outside of the scope of this book. The few sentence explanation is this: NoSQL databses store your data as nested objects, like a giant JSON blob. There aren’t tables and records. Instead, there are `collections` and `documents`. Collections are basically maps, and documents are records. Documents can have collections as properties. In a robust app, your data basically ends up as one giant key-value map.

Firestore is a NoSql database with some added functionality. Namely. you can "subscribe" to the data. Anytime it changes, your app knows. We aren’t going to be concerned with that in this simple app, but it’s helpful to keep in mind that Firestore is all *about reactive programming*.

Now, I have to break the one rule I’ve tried to stick to through out this book: avoid set-up. Firebase can’t be used with out doing some set up in the `android` and `iOS` folders of the Flutter lib. So bear with me while I give you step by step instructions to do this boring task. The good news is, it’s fast. If there are no issues, you can be set up in a a couple minutes. The other good news is, you don’t have to write any Objective-C or Java.

In general, you need to follow these steps:

1. Sign up for a (free) Firebase account.
2. Start a new Firebase project.
3. Add Firestore database to your project.
4. Register your `Android` or `iOS` app with Firestore.
5. Tinker with the native folders.
6. Add Firebase and Firestore to your `pubspec.yaml` file.
7. Use them.

A giant disclaimer here is that this is (for some readers) a book. And a book doesn’t have WiFi or a data-plan, so you can’t click links. That said, I’m going to tell you exactly where you need to go, and it’s all well documented. These steps are mostly an outline.

If you run into a problem and my explanation isn’t getting you anywhere, Google has provided this thorough guide: <codelabs.developers.google.com/codelabs/flutter-firebase/>

#### 10.2.1  Create a Firestore project
First, you have to go to firebase.google.com and set up an account. It’s a standard, quick, process.

Then, create a project. In Firebase, you’ll basically have a new project for every app you build. Here are some instructions directly from the Firebase docs: [2]

1. In the Firebase console, click Add project, then select or enter a Project name.
2. (Optional) Edit the Project ID. Firebase automatically assigns a unique ID to your Firebase project. After Firebase provisions resources for your Firebase project, you cannot change your project ID. To use a specific identifier, you must edit your project ID during this setup step.
3. Follow the remaining setup steps in the Firebase console, then click Create project.
4. Firebase automatically provisions resources for your Firebase project. When the process completes, you’ll be taken to the overview page for your Firebase project in the Firebase console.

#### 10.2.2  Configure your app
Now comes the fun part. Before we can use the Firebase packages in Flutter, we need to tell the native app platforms (iOS and Android) that we’re using it. The process is different for the two platforms, so you should follow the one that you use to develop. For example, I do all my testing for Flutter in "iOS", so I wouldn’t bother adding Firebase to the Android app unless I plan on releasing the app to production.

**Configure iOS**
1. In the Firebase console, select Project Overview in the left nav, then click the iOS button under "Get started by adding Firebase to your app". You’ll see the dialog shown in the following modal:

2. The important value to provide is the iOS bundle ID, which you’ll obtain using the following three steps.
3. In the command line tool, go to the top-level directory of your Flutter app.
4. Run the command `open ios/Runner.xcworkspace` to open Xcode.
5. In Xcode, click the top-level Runner in the left pane to show the General tab in the right pane, as shown in the image below. Copy the Bundle Identifier value.
6. Back in the Firebase dialog, paste the copied Bundle Identifier into the iOS bundle ID field, then click Register App.
7. Continuing in Firebase, follow the instructions to download the config file `GoogleService-Info.plist`.
8. Go back to Xcode. Notice that Runner has a subfolder also called Runner (as shown in the image above).
9. Drag the GoogleService-Info.plist file (that you just downloaded) into that Runner subfolder.
10. In the dialog that appears in Xcode, click Finish.
11. Go back to the Firebase console. In the setup step, click Next, then skip the remaining steps and go back to the main page of the Firebase console.

**Configure Android**
I think Android is less cumbersome to set up, because you don’t have to deal with any third application like Xcode

1. In the Firebase Console, select Project Overview in the left nav, then click the Android button under "Get started by adding Firebase to your app". You’ll see the dialog shown in the following modal:
2. The important value to provide is the Android package name, which you’ll obtain using the following two steps.
3. In your Flutter app directory, open the file `android/app/src/main/AndroidManifest.xml`
4. In the manifest element, find the string value of the package attribute. This value is the Android package name (something like `com.yourcompany.yourproject`). Copy this value.
5. In the Firebase dialog, paste the copied package name into the Android package name field.
6. Click Register App.
7. Continuing in Firebase, follow the instructions to download the config file `google-services.json`.
8. Go to your Flutter app directory, then move the `google-services.json` file (that you just downloaded) into the android/app directory.
9. Back in the Firebase console, skip the remaining steps and go back to the main page of the Firebase console.
10. Finally, you need the Google Services Gradle plugin to read the google-services.json file that was generated by Firebase. In your IDE or editor, open android/app/build.gradle, then add the following line as the last line in the file:
```js
apply plugin: 'com.google.gms.google-services'
```
1.  Open android/build.gradle, then inside the buildscript tag, add a new dependency:
```json
buildscript {
   repositories {
       // ...
   }
   dependencies {
       // ...
       classpath 'com.google.gms:google-services:3.2.1'   // new
   }
}
```
And you’re done!

#### 10.2.3  Add Firebase to your pubspec
I’m sorry if that was painful. But now, we can get back to Flutter. The last set up step is adding the right packages to your `pubspec.yaml` file. This is the finished `dependencies` list:

```dart
Listing 10.12. The finished pubspec file

// backend/pubspec.yaml
dependencies:
  flutter:
    sdk: flutter
  http: ^0.12.0
  json_annotation: ^2.0.0
  firebase_core: ^0.3.4
  cloud_firestore: ^0.9.1
```

#### 10.2.4  Use Firestore
From a super high level, there are two steps to using Firebase now. First, write the services that talk to Firestore. Then, call the services in the app. Originally, the app was calling Http services, so those have been switched out. I’ll cover that in shortly.

> NOTE
```dart
Listing 10.13. Implementing Firebase services

import 'package:cloud_firestore/cloud_firestore.dart';

class FirebaseServices implements Services {
    // ...

  @override
  Future<List<Todo>> getTodos() async {
    QuerySnapshot snapshot = await Firestore.instance.collection("todos").getDocuments();
    AllTodos todos = AllTodos.fromSnapshot(snapshot);
    return todos.todos;
  }
}
```

The important part of that code, for this section, is the line that deals with the object `QuerySnapshot`. There’s a lot going on there. Let me talk about it piece by piece.

* `QuerySnapshot` is a class that represents some data from the database at any given moment. The term "snapshot" is used because Firestore is a real-time database, so the data is theoretically changing all the time. A snapshot says "This is the data you wanted in the moment that you asked for it." It’s a common term in NoSQL databases.
* On the other side of the equals sign, the first important chunk is `Firestore.instance`. `instance` is a static getter on the `Firestore` package which represents the database itself. All calls to Firestore in your app will start by grabbing `Firestore.instance`.
* `collection` is a method that retrieves a collection from your databse. There are two types of objects in Firestore: documents, and `collections`, which are a `Map` of documents. The `collection` expects a "path" that corresponds to the data in your databse. In this case, "todos" is a top-level collection in the data base. If you were looking for sub-todos of a todo, the path might by `todos/$id/subtodos`, where `id` represents a specific todo, and `subtodos` is a collection on that document. (This app doesn’t deal with nested data, so that’s just an example.)
* Finally, `getDocuments` grabs all the documents from that collection and returns them as a `QuerySnapshot`.

So, from a highlevel, this function is basically saying "Hey Firestore, give me all the documents you have nested under the key 'todos' at this very moment". Then, the function passes that `QuerySnapshot` to `AllTodos.fromSnapshot`, which turns that snapshot into some Dart classes we care about.

The `AllTodos.fromSnapshot` method looks like this:

```dart
factory AllTodos.fromSnapshot(QuerySnapshot s) {
    List<Todo> todos = s.documents.map((DocumentSnapshot ds) {
      return new Todo.fromJson(ds.data);
    }).toList();
    return AllTodos(todos);
  }
```

That’s a simple example, and maybe seem like there’s a lot more that I haven’t covered yet, but that’s what using Firestore is about. If you have a good understanding of streams, (which you hopefully do from the previous chapter), and can work with `QuerySnapshots` and the `Firestore.instance`, that’s 99% of what you need to know to work with Firestore. Basically, using Firestore is all about making queries and then deserializing data into Dart objects.

[2] These steps come from <firebase.google.com/docs/flutter/setup>

#### 10.3  Dependency Injection
Dependency injection is a jargon-y term. But, it’s an important concept if you’re build clients like webapps and Flutter apps. Its important because it makes your code highly-reusable, and let’s you share code accross many platforms. If that sounds abstract, consider this: In Dart, server-side apps, web apps, and Flutter apps can all be written. But, because web-apps run in the browser and the rest don’t, HTTP calls are made in two different ways, by two different packages. So, if you have a Flutter app and a web app, the service that these two apps use to get the same data from Typicode is different.

Wouldn’t it be nice if you could write controllers that don’t care about which platform is being used? By that I mean, wouldn’t it be nice if the UI could just call `service.getTodos`, but didn’t really care what exactly the service is. That way, the web UI could call the same method as the Flutter UI, but the Http request that’s made would be different. This concept is called dependency injection, and it’s a way to share code between multiple apps (among other things).

In the `backend` app I wrote for this chapter, I use dependency injection to switch between using Firestore and Http seamlessly. Notice that the `_TodoPageState` object, when fetching todos, calls `widget.controller.fetchTodos`. And that method calls `services.getTodos`. And the `TodoController` is passed in a `Services` obeject. So, does the controller really care about what services are called? No. It only cares that it gets back a list of Todo objects when it calls `services.getTodos`.

If this is confusing, consider the `TodoController` for a moment. In declares a member with the line `final Services services`. But, the `lib/services/todos.dart` file. This file has three classes in it:

```dart
Listing 10.14. Using abstract classes for dependency injection

abstract class Services {
  Future<List<Todo>> getTodos();
  Future<Todo> updateTodo(Todo todo);
  Future addTodo();
}

class HttpServices implements Services {
// ...

class FirebaseServices implements Services {
//...
```

The first class is abstract. In some languages, this is similar to an `interface`. If you aren’t familiar, that is a class that you can’t use directly, but it keeps the classes that you do use honest.

The following two classes implement the abstract class, effectively saying "I am of type `Service`, but my methods have logic that may be different than other classes that implement `Service`.

To make that example concrete, let’s look at the `main.dart` file. At the very top of the `main` function you can inject the proper service into the controller.

```dart
Listing 10.15. Using dependency injection

//backend/lib/main.dart

void main() async {
//  var services = new HttpServices();
  var services = new FirebaseServices();
  var controller = new TodoController(services);
```

The point is that the controller doesn’t care which instance of the `Services` class it gets, because it just calls `Services.getTodos`. Because the base `Services` class declares that it will have a function called `getTodos`, all classes that implement it are required to have that method, and return the same type from those methods. So, the `HttpServices` and `FirebaseServices` classes will have a method called getTodos that returns a `Future<List<Todo>`.

In real life, this example is somewhat contrived. It’s unlikely that you’d have two different service implentations for the same client (such as a Flutter app), but you may have a different `Services` implementation for a web app and a Flutter app, but using dependency injection both apps can use the same `controller` class. Which means you don’t have to rewrite the controller.

This is a powerful approach to sharing code with Dart between web apps and mobile apps, which is now possible thanks to Flutter. If you’re building an app with multiple clients, this is an excellent way to guarantee all the apps are on the same page. At my day job, whenever we have a bug in our app, we can fix it once and it just works in both web and mobile. And that saves a ton of time and resources.

#### 10.4  Summary
* Google has provided packages for Http if you want to use a traditional backend.
* There’s also Firebase packages, which let you use Firebase as a backend with ease.
* Breaking your controller logic out of your UI, and making it a middle-man between the UI and services makes your logic layer highly reusable.
* You can share code between a webapp and mobile app using dependency injection.
* Regarless of the backend you’re using, JSON serialization allows you to gather data from external sources and turn it into proper Dart objects.
* JSON serialization can be done manually, or with packages that generate code automatically.