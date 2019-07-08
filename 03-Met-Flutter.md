>In this chapter:
>* Dissecting Flutter Basics via the Increment App
>* Flutter Widget classes
>* BuildContext, the widget tree, and the element tree
>* Flutter development environment and tips

I imagine, because you’re reading this, that you’re at least intrigued by Flutter. By the end of this chapter, I hope you’ll be excited about it. And, (even better) you’ll have a deep understanding of how Flutter works.

I often find new programming frameworks and libraries so mysterious and magical that I feel like I’m doing something wrong. I find myself thinking, 'It can’t be this easy. Whats the catch? What am I missing?' In most cases, it turns out, if you dig deep enough, it’s not magical at all. And while you don’t have to dig deep, its empowering.

I write UI code in two different ways: Sometimes, I guess and check. I do this when I don’t understand what I’m doing. It’s unbelievable how much time I’ve wasted throwing different CSS properties at a layout problem on the web, guessing and checking. And 100% of the time, if I stop and actually understand what’s happening, the solution becomes obvious. We’re going to take the second approach to writing Flutter code: understand what’s happening under the hood, and taking a guesswork out of it.

This chapters goal is explore the foundation how Flutter works for us, as developers. This is the plan for building a foundation:

1. Take an in-depth look at the 'counter app', which is the app that’s generated when you start a new Flutter project with the CLI.
2. Make the counter app more robust adding some basic Widgets.
3. Spend some time talking about BuildContext, the widget tree, and elements. Understanding how this works is 90% of debugging Flutter errors.
4. Learn tricks and tools that the Flutter team has built in to the SDK that makes development easier.

>**IS YOUR ENVIRONMENT SET UP?**
If flutter isn’t installed on your machine yet, you can find installation instructions in the appendix. If you’re reading this in MEAP, you can find installation instructions at flutter.io/get-started/

#### 3.1  Intro to the counter app
Let’s fire up that first Flutter app. Navigate in your terminal to the location you want this app to live.
```
$ cd ~/Desktop/flutter_in_action/
$ flutter create counter_app
$ cd counter_app && flutter pub get
```
And run your app. This is what you should see in your simulator:
**Figure 3.1. The Flutter counter app**
![](https://dpzbhybb2pdcj.cloudfront.net/windmill/v-7/Figures/basic_flutter.png)

You can press that button and the counter will increase. It’s a hoot. It’s also worth noting here that this is incredible. Starting a new Flutter project takes about 2 minutes!

### 3.1.1  Flutter project structure
A Flutter project, when first created, is a big directory. The good news is most of it doesn’t matter to you. This is what your directory should look like:
```
counter_app
  |- android
  |  ... a bunch of junk
  |- ios
  |  ... a bunch of junk
  |- lib
    |- main.dart
  |- test
    |- widget_test.dart
  pubspec.lock
  pubspec.yaml
  README.md
```
#### 3.1.2  Anatomy of a Flutter App
Then majority of the counter app lives inside the `main` file. The generated source code is beautifully commented, so I feel like I’m beating a dead horse here, but let’s walk through some of the most important parts of the app, starting at the top of `main.go`, working down.
```
import 'package:flutter/material.dart';
```
```
void main() => runApp(MyApp());
```

At the least, your app will contain a line like this one. In a more robust app, you might do more in your `main` function, but you must call `runApp` with your top-level widget passed as an argument.

#### 3.1.3  Everything is a Widget
In Flutter, (nearly) everything is a widget, and widgets are just Dart classes that know how to describe their view. They’re blueprints that Flutter will use to paint elements on the screen. The widget class is the the only view model that Flutter knows about. There aren’t separate controllers or views.

In most other frameworks, especially on the web, widgets are called components, and the mental model is similar. A widget (or component) is a class that defines a small piece of your UI. To build an app, you make a ton of widgets (or components) and put them together in different ways, to compose gradually larger widgets.

A difference, though, in components and widgets, is that a widget can define any aspect of an application’s view. Some widgets, such as the `Row`, define aspects of layout. Some are less abstract and define structural elements, like `Button` and `TextField`. The theme that defines colors and fonts in your app is a widget. Animations are defined by widgets. In a component-based framework from the web, you can build a component that has a singular job of adding padding to a child widget, but you don’t have to. You could use CSS to add padding to whichever component you want. In Flutter you can only style widgets with other widgets. To add padding, you use a `Padding` widget.

The point is, everything is a widget. Even the root of your app is a widget. There isn’t a special object called 'App'. You define your own widget, such as `MyApp`, which returns yet another widget in it’s 'build' method.

The Flutter library is full to the brim with pre-defined widgets. These are some of the most common widgets:

* Layout - `Row`, `Column`, `Scaffold`, `Stack`
* Structures - `Button`, `Toast`, `MenuDrawer` ,
* Styles - `TextStyle`, `Color`, `Padding`
* Animations - `FadeInPhoto`, transformations
* Positioning and Alignment - `Center`, `Padding`

#### 3.1.4  The build method
Every widget must have a `build` method, and that method must return a widget.

**Figure 3.2. Bare minimum StatelessWidget**
![](https://dpzbhybb2pdcj.cloudfront.net/windmill/v-7/Figures/code_annotation_widget.png)

Back in the app, take a look at this 'top-level' widget, MyApp. MyApp in this counter app example is your top level widget, but it’s not special, it’s a widget like anything else:
```dart
class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Flutter Demo',
      theme: ThemeData(
        primarySwatch: Colors.blue,
      ),
      home: MyHomePage(title: 'Flutter Demo Home Page'),
    );
  }
}
```

#### 3.1.5  `new` and `const` constructors in Flutter
In Flutter, you’ll create many instances of the same widgets. Many built-in widgets have both regular constructors and `const` constructors. Immutable instances of widgets are more performant, so you should always use `const` when you can. Flutter makes this incredibly easy on you by letting you omit the `new` and `const` key words altogether. The framework will infer which one to use, and always use `const` when it can.

#### 3.1.6  Hot Reload
Hot reload is one of Flutters best selling points to native mobile-developers. If this section doesn’t excite you, I don’t know how to help you.

A fun fact about Dart is that is has both an ahead-of-time (AOT) compiler and just-in-time (JIT) compiler. In Flutter, when you’re developing on your machine, it uses the JIT. It’s called 'just in time' because it compiles and runs code as it needs to. When you deploy the app production, Flutter uses the AOT compiler. For us developers that means that you can develop and re-compile code quickly in development, but don’t sacrifice non-native performance in production.

Just how quick is hot reload? In the counter app, on line ~15, and change the text passed in to the MyHomePage title argument.
```dart
// `chapter_3/counter_app/lib/main.dart` -- line ~15
home: new MyHomePage(title: 'Flutter Home PageDemo'); // old

home: new MyHomePage(title: 'Hot Reload Demo'); // updated
```
>TIP
>Depending on your environment, you can trigger a hot reload a number of ways.
>
>* In Intellij, there’s a 'hot reload' button, and the shortcut is CMD + \
>   * I have my Intellij shortcuts setup to hot-reload (and Dart Format) on Save. (CMD + S)
>
>* If you used `flutter run` in your terminal, type `r` in that terminal to hot reload.

Fire that hot reload. Be amazed. Even more amazing: You could’ve added new widgets and changed the theme color and it would’ve reloaded just as quickly. In fact, let’s checkout one more example.

On line ~13, in the `ThemeData` constructor, update the `primarySwatch` argument to a different color.
```
// `chapter_3/counter_app/lib/main.dart` -- line ~12
theme: ThemeData(
    primarySwatch: Colors.blue,       // old
),

theme: ThemeData(
    primarySwatch: Colors.indigo,       // updated
),
```

Hit that hot reload again. If everything went okay, your top app bar and the button should’ve changed colors in sub-second time. Pretty amazing stuff.

#### 3.2  Widgets: The widget tree, widget types and the State object
In general, Flutter is made of a handful of widget types. The two that we care about right now, which are the base of all other widget types, are `StatelessWidget` and `StatefulWidget`.

The general goal when developing UI in a Flutter app is to compose a ton of widgets together to build the widget tree. A Flutter app is represented by a widget tree, similar to the DOM on the browser is a tree-structure.

Here’s a simple visual representation of the widget tree for the counter app:

![](https://dpzbhybb2pdcj.cloudfront.net/windmill/v-7/Figures/counter_app_tree.png)

The process of composing widgets together is done by telling widget’s that their child (or children) are more widgets. A simple example is styling some text:
```dart
return Container(
    child: Padding(
    padding: EdgeInsets.all(8.0),
    child: Text("Padded Text")
    ),
);
```

Other common properties in Flutter that allow you to pass widgets into widgets are `children` and `builder`, both of which we’ll see later (and often) in this book.

#### 3.2.1  Stateless Widget
The difference between a `StatefulWidget` and a `StatelessWidget` is right in the name. A `StatefulWidget` tracks it’s own internal state. A `StatelessWidget` is a "dumb" widget. It doesn’t care about its configuration or what data it’s displaying. It could be passed configuration from its parent, or the configuration could be defined within the widget, but it can not change it’s own configuration. A stateless widget is immutable.

>NOTE
When it comes to widget jargon, you’ll see the word configuration often. It’s kind of vague, but basically encapsulates everything: namely the variables passed in and it’s size constraints (enforced by its parent).

Imagine a button in your app. Perhaps it will always say "Submit":
```dart
class SubmitButton extends StatelessWidget {
  Widget build(context) {
    return Button(
      child: Text('Submit');
    );
  }
}
```
Or, perhaps you want the button to say "Submit" in some cases and "Update" in others.
```dart
Listing 3.1. A widget with configuration

class SubmitButton extends StatelessWidget {
  final String buttonText;
  SubmitButton(this.buttonText);

  Widget build(context) {
    return Button(
      child: Text(buttonText);
    );
  }
}
```

Either way, this widget is 'dumb' because it doesn’t update itself. It doesn’t care what the button says. It’s configuration relies on parent widgets.

Also, importantly, `StatelessWidgets` are destroyed entirely when Flutter removes them from the widget tree. We’ll talk more about the widget tree and context later in this chapter, but it’s important to understand that a `Stateless` widget shouldn’t be responsible for any data you don’t want to lose. Once its gone, its gone.

#### 3.2.2  Stateful Widget
A `StatefulWidget` has internal state and can manage that state. All `StatefulWidgets` have corresponding `State` objects. This is the anatomy of every `StatefulWidget`.

```dart
Listing 3.2. Anatomy of a Stateful Widget

class MyHomePage extends StatefulWidget {

  @override
  _MyHomePageState createState() => new _MyHomePageState();
}

class _MyHomePageState extends State<MyHomePage> {

  @override
  Widget build(BuildContext context) {
    // ..
  }
}
```

If you remember earlier, I said that every widget class must have a build method. As you can see above, the `StatefulWidget` class, in fact, doesn’t have a build method. But, every `StatefulWidget` has an associated State object, which does have a `build` method. You can think of the pair of `StatefulWidget` and `State` as the same entity. In fact, `Stateful` widgets are actually dumb and immutable (just like a `StatelessWidget`), but their associated `State` objects are smart and mutable.

`MyHomePage` is a `StatefulWidget`, because it manages the state of the counter in the center of the app. When you tap that button, it fires a method called `_incrementCounter`.

```dart
void _incrementCounter() {
  setState(() {
    _counter++;
  });
}
```
**Figure 3.3. setState tells flutter to repaint**
![](https://dpzbhybb2pdcj.cloudfront.net/windmill/v-7/Figures/basic_flutter_with_action.png)

#### 3.2.3  setState
`setState` is the third important Flutter method that you have to know, after `build` and `createState`. It only exists on the `State` object. It more or less says "Hey Flutter, execute the code in this callback, (in this case, increase the counter variable by one), and then repaint all the widgets that rely on this state for configuration (in this case, the number on the screen in the middle of the app)." This method takes one argument, a void callback.

In our example, the `State` object lives in the `MyApp` widget, and it’s children interact and rely on the state. When you press the button, it calls a method passed to it from `MyAppState`. That method calls `setState`, which in turn calls the `MyAppState.build` method again, repainting the widgets who’s configurations have changed.
**Figure 3.4. setState visual**
![](https://dpzbhybb2pdcj.cloudfront.net/windmill/v-7/Figures/set_state_diagram.png)

There isn’t much more to setState than that, but its worth noting that setState can’t execute async code. Any async work should be done before calling `setState`, because you don’t want Flutter to repaint something before the data it’s meant to display has resolved. For example, if you’re fetching a gif from a gif API on the internet, you don’t want call `setState` before the image is ready to displayed.

#### 3.2.4  initState
The `State` object also has a method called `initState`, which is called as soon as the widget is mounted in the tree. `State.initState` is the method in which you initialize any data needed before Flutter tries to paint it the screen. For example, you could subscribe to streams or compute some data into a human friendly format.

There are a few other lifecycle methods on the `State` object, which I’ll cover later in the book. `initState` and `setState` are the most important, though.

#### 3.3  BuildContext
When you update the theme in your ThemeData, it updates child widgets way down the widget tree. How does this work? Its tied to something that you’ve seen a few times, but I haven’t talked about: `BuildContext`.

Every `build` method in widgets take one argument, the `BuildContext`. BuildContext is a reference to a widget’s location in the widget tree. In practice, this means that your widget can gather information about it’s place in the tree.

A concrete example is the `Theme.of` method, a static method on the Theme class. When called, `Theme.of` takes a `BuildContext` as an argument and returns information about the theme at that place in the widget tree. This is why, in the counter app, we can call `Theme.of(buildContext).primaryColor` to color widgets. That gets the `Theme` information for this point in the tree and then returns the data saved at the variable `primaryColor` in the Theme class.

Every widget has it’s own `BuildContext`. Which means, if you had multiple themes dispersed throughout your tree, getting the theme of one widget could return different results than another.

In the specific case of the theme, or other `of` methods, you’ll get the nearest parent in the tree of that type.

![](https://dpzbhybb2pdcj.cloudfront.net/windmill/v-7/Figures/theme_of.png)

`BuildContext` is used for a lot more under the hood though. If you want to insert widgets into the tree, or display widgets that are in a different place in the tree, you have to use `BuildContext` to tell the framework where exactly in the tree you want to display widgets.

For example, Flutter uses the build context to display modals and routes. It know where to insert a modal (for example, which page to display it over), based on the current build context.

The important first point about build context is that it contains information about a widget’s place in the widget tree, not about the widget itself.

Widgets, state and context are arguably the three corner stones of developing a basic app in Flutter. Which means we can put them in action now.

#### 3.4  Enhancing the Counter app with the most important widgets
A counter app isn’t useful right now. You can’t even reset your count. In this section, we’ll extend the functionality of the counter app and explore some of the most important Widgets in Flutter. According to the documentation, the absolute basic Widgets are the following:

* Container
* Row
* Column
* Image
* Text
* Icon
* Raised Button
* Scaffold
* AppBar
  
Of these widgets, `Column`, `Text`, `Icon`, `Scaffold` and `AppBar` are already in the counter app. We’ll add the rest to make the counter app a bit more fun.

Your next-level counter app will look like this in the end:

**Figure 3.5. Finished Counter App 2.0**
![](https://dpzbhybb2pdcj.cloudfront.net/windmill/v-7/Figures/finished_counter_app.png)

#### 3.4.1  Raised Button
First, let’s add a button to decrease the counter. This functionality will all live in the `_MyHomePageState` class.

To decrease the counter, we need:

* A button to click
* A function that decrements _counter by one.
  
First, to add the button, let’s start in the `build` method of `_MyHomePageState`.
```dart
Listing 3.3. Add a Raised Button to the _MyHomePageState.build method
// _MyHomePageState
Widget build(BuildContext context) {
    return new Scaffold(
      // ...
      body: new Center(
        child: new Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: <Widget>[
            // ...
            RaisedButton(
              child: Text("Decrement Counter"),
              onPressed: _decrementCounter,
            ),
          ])),
      // ...
```

To finish up that functionality, we need to write the `_decrementCounter` method.
```dart
void _decrementCounter() {
  setState(() => _counter--);
}
```
Interaction is largely handled by callbacks in Flutter, like `onPressed`. In widgets provided by Flutter and that you’ll write, you’ll use callbacks to execute some function when a user interacts with your app. In built-in widgets, you’ll see `onPressed`, `onTapped`, `onHorizontalDrag`, and many more.

> Just in Time: Composition (over inheritance)
>
> "Designing object-oriented software is hard, and designing reusable object-oriented software is even harder."
>
> This is the opening line of 'Design Patterns: Elements of Reusable Object-Oriented Software', published in 1994. In all object-oriented programming, one of the hardest design issues is establishing relationships between your classes. There are two ways to create relationships between classes. The first, inheritance, established an "is a" relationship. Composition establishes a "has a" relationship. For example a `Cowboy` is a `Human`, and a `Cowboy` has a `Voice`. Inheritance tends to have you designing objects around what they are, and composition around what they do.
>
> What if you wanted to make a game where you’re a Cowboy and you have to protect the wild west from Aliens. Maybe you have classes like this:

```dart
Human
  .rideHorse()

  Cowboy
    .chaseOutlaws()
    .fightAliens()

  Rancher
    .herdCattle()

Alien
  .flySpaceship()
  .invadeEarth()
```
> Great. You got to re-use the `rideHorse` method because cowboys and ranchers do both, in fact, ride horses. You’re well into making this killer game when you have the wild idea that the aliens learn the ancient art of horse riding. Well, that’s a problem. An alien isn’t a human, so you shouldn’t have Alien inherit from Human. But you also don’t want to rewrite `rideHorse`.
>
> This could’ve been avoided by using composition from the beginning, rather than inheritance. You could have a HorseRiding class, which could be added as a member to any class. It’d look more like this:
```dart
HorseRiding
  .rideHorse

Cowboy
  HorseRidingInstance.rideHorse()
  .chaseOutlaws()
  .fightAliens()

Rancher
  HorseRidingInstance.rideHorse()
  .herdCattle()

Alien
  HorseRidingInstance.rideHorse()
  .flySpaceship()
  .invadeEarth()
```
> This is great. No matter how many objects need to ride a horse, you have an easy, decoupled way to add that functionality.
>
> The curious among you might be asking, "Why not just make all the actions into their own classes and inherit everything?" Well, that’s not a bad idea. Maybe the Rancher learned how to fly a spaceship. So now how do we think about our objects that have these methods?
>
> Well, a Cowboy is a `HorseRider` and `AlienFighter` and `OutlawChaser`. Similarly, the alien and rancher are combinations of what they can do:
```dart
Alien = HorseRider + SpaceShipFlyer + EarthInvader
Rancher = HorseRider + CattleHerder
Cowboy = HorseRider + OutlawChaser + AlienFighter
```
> If you made classes that represents `HorseRider`, `EarthInvader`, and the like, then you could implement those actions into your classes. This is what `composition` is.
>
> (If you’re thinking, "that’s sounds a lot like the idea behind abstract classes," you’re correct. We’ll explore those deeply in the section 3 of this book.)
>
> In the example above where you added a `RaisedButton`, you used composition. Recall the code:
```dart
//...
RaisedButton(
  child: Text("Decrement Counter"),
  onPressed: () => _resetCounter(),
),
//...
```
> To make a button that says "Decrement Counter", you passed in another widget (`Text`) that handled the responsibility of setting text.
>
> In Flutter, favor composition (over inheritance) to create reusable and decoupled Widgets. Most widgets don’t know their children ahead of time. This is especially true for widgets like Text blocks and Dialogs, which are basically 'containers' for content.
> 
> A more robust example of a button may look like this:
```dart
class PanicButton extends StatelessWidget {
  final Widget _display;
  final VoidCallback _onPressed;

  PanicButton({String this._display, VoidCallback this._onPressed});

  Widget build(BuildContext context) {
    RaisedButton(
      color: Colors.red,
      child: _display,
      onPressed: _onPressed,
    );
  }
}
```
> Here, using composition, I’m saying "this button has text, rather than this "this text is a button." Because, what if you want the button to display an icon instead of text? It’s already set up to do that. All you need to do is pass in an `Icon` instead of `Text`. The button doesn’t care about it’s child, it only knows that it has one.
>
> You could kick that up a notch and pass in the color if you wanted. Now, the button doesn’t even care about that, only that it will be told what color it is.

Back in your app, you should now have a button that you can use decrement the counter by one.

#### 3.5  Intro to Layout in Flutter
Flutters rendering engine is cool because it isn’t based on an sort of specific layout system. Way down on the low-level, it doesn’t consider the screen a cartesian graph (at first). It doesn’t say "you have to use WIHO layout, flex layout, plain ol cartesian coordinates, or anything else. It leaves that up to us. And often, we mix and match those systems to achieve the layout we want.

The most common questions from those working in Flutter for the first time are all about layout. Layout is hard seemingly on every platform. Placing pixels on a screen in a smart way is (inexplicably) much harder than it should be. Lucky for us, Flutter offers a whole host of layout widgets that are ready to go out of the box.

Besides layout widgets, I’ll also talk about constraints in Flutter. Constraints are widgets and properties on widgets that tell child widgets how much space they can take up. There’s a big section in a couple pages about constraints.

#### 3.5.1  Row and Column
The most common layout style in Flutter is known as the flexible layout, just like flexbox on the web. You can use flex layouts with `Column` and `Row` widgets.

The counter app already has a `Column` widget in it:

```dart
Listing 3.4. The Column widget in the counter app

// _MyHomePageState
body: new Center(
  child: new Column( // The column iss aptly named. It layout all its children in a column.
    mainAxisAlignment: MainAxisAlignment.center,
    children: <Widget>[
      new Text(
        'You have pushed the button this many times:',
      ),
      new Text(
        '$_counter',
        style: Theme.of(context).textTheme.display1,
      ),
      RaisedButton(
        child: Text("Decrement Counter"),
        onPressed: _decrementCounter,
      ),
    ],
  ),
),
```

The `Row` widget behaves like the `Column` but on a horizontal axis.

I want to wrap the decrement button in `Row` in the example app, so I can add a second button beside it.

In that same code block, start by adding the `Row` around the `RaisedButton`:

```dart
// _MyHomePageState.build

Row(                    // new
  children: <Widget>[   // new
    RaisedButton(
      color: Colors.red,
      child: Text(
        "Decrement",
        style: TextStyle(color: Colors.white),
      ),
      onPressed: _decrementCounter,
    ),
  ],                    // new
),                      // new
```

When you hot reload your app, the Decrement button is now aligned to the left side of the screen. This is because Flexible widgets try to take up as much space as they can on their *main* axis.

The row widget expands as much as it can horizontally, which in this case is as wide as the whole screen, constrained by it’s parent (the column).

**Figure 3.6. Row widget with single child and no alignment**
![](https://dpzbhybb2pdcj.cloudfront.net/windmill/v-7/Figures/app_annotation_row_width.png)

#### 3.5.2  Layout Constraints in Flutter
Layout and constraints are monumentally important in Flutter. Flutter is, after all, mainly a (brilliant) UI library and a rendering engine. Understanding how widgets determine their sizes will save you headaches in the future. You will certainly, at some point, get some errors when you’re using `Row` and `Column` and other layout widgets. These are layout constraint errors.

I’ve probably seen this error more than any other in Flutter:
```
fluttr layour infinite size error
```
Before we get into fixing that error, I need to take a conceptual aside to discuss how Flutter knows what pixels to paint on the screen.

#### 3.5.3  RenderObject
I’ve said many times that everything in Flutter is a widget. That isn’t quite true. There are other classes that Flutter uses, mainly internally, besides widgets. One of which is the `RenderObject`.

Render objects are responsible for the actual painting to the screen done by Flutter. Render objects are made internally by the framework, and all the render objects make up the render-tree, which is separate from the widget tree. The render tree is made up of classes that implement `RenderObject`. And, render objects have corresponding widgets.

As developers, we write widgets, which provide data (such as constraints) to a render object. The render object has methods on it like `performLayout` and `paint`. These methods are responsible for painting the pixels on the screen. They’re concerned with exact bits of information for controlling pixels on the screen.

These render objects are also kind of dumb though. By design, they know some basic data about their parent render object, and they have the ability to visit their children, but they don’t coordinate with each other on the scale of the whole app. They don’t have the ability to make decisions, they only follow orders.

Importantly, widgets build child widgets in their `build` method, which create more widgets, and so on down the tree until it bottoms out at a `RenderObjectWidget` (or, a collection of `RenderObjectWidgets`). These are the widgets that create render objects that paint to the screen.

Consider a `column` widget, which would not be a leaf `RenderObjectWidget` in a widget tree. A column is an abstract layout idea, it isn’t an actual thing you can see. Text and colors are concrete objects that can be painted. The job of a column is to provide constraints, not to paint anything on he screen.

>NOTE
RenderObject’s aren’t of that much concern to us as developers. But, they’re an important piece of the relationship between your widgets and how Flutter actually works. The render object API is exposed to us, but it’s unlikely you’d need to use it.

#### 3.5.4  RenderObject and Constraints
Render objects are closely tied to layout constraints. While you can set your own constraints on widgets using constraint widgets), render objects are ultimately responsible for telling the framework a widget’s actual, physical size. Constraints are passed to a render object, which eventually says "okay, given these constraints, I will be this size and in this exact location."

In other words, constraints are concerned with `minWidth` `minHeight` `maxWidth` and `maxHeight`. `Size` on the other hand, is concerned with actual `width` and `height`. When a render box is given its constraints, it then decides how much of that alloted space it will actually take up (it’s size).

Different render objects behave differently. The most common render object subclass, by far, is the `RenderBox`, which calculates widget’s size using a cartesian coordinate system. In general, there are three kinds of render boxes:

1. Those that try to take up as much space as possible. For example, the boxes used by a `Center` widget.
2. Those that try to the same size as their children. For example, the boxes used by an `Opacity` widget.
3. Those that try to be a particular size, such as the boxes used by an `Image` widget.

#### 3.5.5  RenderBoxes and layout errors
Back to the original problem: Sometimes the constraints that are given to a box are unbounded. This happens when either the `maxHeight` or `maxWidth` given to a render box is `double.INFINITY`. Unbounded constraints are found in `Row`, `Column`, and widgets that are scrollable. Which makes sense because a row can be infinitely wide (depending on its children).

`Row` and `Column` are special because they’re flexboxes. They behave differently based on the constraints passed by their parents. If they have bounded constraints, they try to be as big as possible within those bounded constraints. If they have unbounded constraints, they try to fit their children in that direction. For example, a column full of images that has unbounded constraints will try to be as tall as all of the images combined height.

If you don’t know to look for it, this can lead to a pesky little error. Consider this code:

```dart
Listing 3.5. Columns within Columns can cause infinite height

child: Column(
  children: <Widget>[
    new Column(
      mainAxisAlignment: MainAxisAlignment.center,
      children: <Widget>[
        Expanded(
          child: new Text(
            'You have pushed the button this many times:',
          ),
        ),
      ],
    ),
  ],
),
```
In this case, the inner column is going to try to be whatever size it’s child tries to be, and is unbounded by its own parent. The `Expanded` is going to say 'Great, I have no height constraint, and it’s in my nature to try and be as big as possible, so I’m going to expand forever.' That’ll thrown an error.

This is a contrived example, to be sure. But because widgets pass constraints down the tree, there can be a some degrees of separation between nested flex boxes and you’ll end up with an infinitely expanding child somewhere.

#### 3.5.6  Multi-child widgets
Back in the app, let’s add this second button to the row, which will increment the counter.

```dart
Listing 3.6. Add a second button to the Row

Row(
  children: <Widget>[
    RaisedButton(
      color: Colors.red,
      child: Text(
        "Decrement",
        style: TextStyle(color: Colors.white),
      ),
      onPressed: _decrementCounter,
    ),
    RaisedButton(
      color: Colors.green,
      child: Text(
        "Increment",
        style: TextStyle(color: Colors.white),
      ),
      onPressed: _incrementCounter,
    ),
  ],
),
```
**Figure 3.7. Row Widget with multiple children and no alignment**
![](https://dpzbhybb2pdcj.cloudfront.net/windmill/v-7/Figures/app_annotation_row_width_2.png)

Now, there are two buttons, both aligned to the left side. To make that a little more pleasing to look at, we need to add an alignment to the Row.

```dart
Row(
  mainAxisAlignment: MainAxisAlignment.spaceAround,
  children: <Widget>[
    RaisedButton(
      color: Colors.red,
  // ...
```
**Figure 3.8. Row Widget with spaceAround alignment**
![](https://dpzbhybb2pdcj.cloudfront.net/windmill/v-7/Figures/app_annotation_space_around.png)

These are all the flexible layout alignments from a real life example.

Listing 3.7. Alignment styles in Flutter

#### 3.5.7  Icons
In Flutter, you have access to all the Material Design icons for free as a constant. Compared to my experience on the web, this is a real blessing. Icons are a core part of building mobile interfaces where space is limited. In Flutter, you don’t have to find an external library or upload images. Very cool.

In the counter app right now, there’s already an Icon being used in the `FloatingActionButton`, the `Add` icon. Since `Icons` is a constant, you can access Icons anywhere in your app (passed into the Icon widget). The Icon widget is, as you probably guessed, a widget. You can use it anywhere.

The `FloatingActionButton` button is a prime example of a widget that Flutter gives you, styled and all, for free. "A floating action button is a circular icon button that hovers over content to promote a primary action in the application. Floating action buttons are most commonly used in the `Scaffold.floatingActionButton` field." (Flutter documentation). When used in a `Scaffold` (as it is in our app), it’ll be placed where it needs to be, no work necessary on your part. You can use it anywhere you’d like, though. It’s just a round button with some nice box-shadow on it.
I say this is a prime example because it’s certainly styled a certain way, and it looks like it belongs in Google apps. But it’s also not "distinct" in any way. That’s what’s fantastic about Material Design. It’s possible to take full advantage of Flutter’s built in widgets without relinquishing your app’s unique feel. (Flutter: lowering the value of designers since 2018)

Back in the app, we want the `FloatingActionButton` (FAB) to reset the counter, not increase it. Easy enough! What are the steps to getting this done?

* Write a new method `resetCounter`, and pass it to the FAB’s `onPressed` argument.
* Change the icon used in the FAB.

First, let’s write the method. All we want to do is set `_counter` back to 0. Also, don’t forget, we need to tell Flutter to repaint!
```dart
void _resetCounter() {
  setState(() => _counter = 0);
}
```

That’s a simple enough method. Now we have to update the `FAB` itself.

Step one is choosing the right Icon. I choose `Icons.refresh`. I think it makes the most sense. But, there are hundreds of material icons available. Here’s the icons I use the most often:

Table 3.1. Common Material Icons

| icon	| const name	| icon	| const name |
|-------|-------------|-------|------------|
| |Icons.add| | Icons.check |
| | Icons.arrow_drop_down | | Icons.arrow_drop_up |
| | Icons.arrow_forward | | Icons.arrow_back |
| | Icons.chevron_left | | Icons.chevron_right |
| | Icons.chevron_left | | Icons.chevron_right |
| | Icons.close | | Icons.menu |
| | Icons.favorite | | Icons.refresh |

In the `FAB`, change the icon and the function passed into the `onPressed` callback.

```dart
floatingActionButton: new FloatingActionButton(
  onPressed: _resetCounter, // Updated Callback
  tooltip: 'Reset Counter', // Updated Tooltip
  child: new Icon(Icons.refresh), // Updated Icon
),
```

#### 3.5.8  Images
As you’d expect, Flutter makes it easy to add images to your app. The `Image` widget has different constructors, depending on the source of your image. The easiest way to add an image is the with `Image.network` constructor. You pass it a url as a String and it takes care of everything for you. An example would be: `Image.network("https://funfreegifs.com/panda-bear")`. Any URL that resolves to an image can be passed.

In real life, though, you probably want some images hosted locally. In this case Flutter gives us `Image.asset`. This constructor works the same, you pass in a path to an image in your project, and it resolves it for you. However, you have to tell Flutter about it in your `pubspec.yaml` first.

In this counter app, let’s put a Flutter logo at the top of the app. Theres a Flutter logo image in the GitHub repository that follows this book. You can download it there. Once you have the image, navigate to your `pubspec`.

I’ll take this opportunity to briefly walk through the `pubspec` of a basic Flutter app. (If you start a new Flutter project, the pubspec file has in-depth comments, which you may find quite helpful.)

```dart
Listing 3.8. Add an image to your Flutter pubspec file

name: counter_app
description: A new Flutter project.
version: 1.0.0+1

environment:
  sdk: ">=2.0.0-dev.68.0 <3.0.0"

dependencies:
  flutter:
    sdk: flutter
  cupertino_icons: ^0.1.2

dev_dependencies:
  flutter_test:
    sdk: flutter

flutter:
  uses-material-design: true
  assets:
    - flutter_logo_1080.png
```

Any assets that you need in your app needs to be listed under the `assets` header in your spec file. It must follow this format. YAML is sensitive to whitespace. The assets themselves should be listed as a path from the `lib` folder in your project. I just happen to put mine directly under lib, but if it was in a folder called `images`, that line would say `images/flutter_logo_1080.png`.

This assets list isn’t for images only. You can include any static files that you want to access in your app. For example, at my job we list some `.yaml` files here that we use for configuration.

Now that you’d added the `flutter_logo_1080.png` to your assets, you’re going to have to re-run. Hot reload doesn’t work with the spec files.

Now, you can access that image in the counter app by adding the `Image` to the `Column` widget’s children.
```dart
Listing 3.9. Adding an image to your Flutter app

children: <Widget>[
  Image.asset(
      'flutter_logo_1080.png',
      width: 100.0,
    ),
  ),
  Text(
    'You have pushed the button this many times:',
  ),
```

Hot reload. You should have an image in your app.

#### 3.5.9   Container Widget
The image doesn’t look great though. It’s just kinda sitting there on top of the text. Let’s clean it up with the `Container` widget. This is what we’re going for, and it can all be done with the `Container`:

Some widgets, for example Container, vary from type to type based on their constructor arguments. In the case of Container, it defaults to trying to be as big as possible, but if you give it a width, for instance, it tries to honor that and be that particular size.

**Figure 3.9. Transform the Flutter Logo with the Container widget**
![](https://dpzbhybb2pdcj.cloudfront.net/windmill/v-7/Figures/app_transform_logo_container.png)


The `Container` widget is a "convenience" widget that provides a whole slew of properties that you would otherwise get from individual widgets. For example, there is a `Padding` widget, that solely adds padding to its child. But the `Container` widget has a padding property (among others).

You will likely get a lot of use out of the `Container` widget. Just look at all these optional properties you can take advantage of (and these aren’t all):

```dart
Listing 3.10. A sample Container widget properties

this.alignment,
  this.padding,
  Color color,
  Decoration decoration,
  this.foregroundDecoration,
  double width,
  double height,
  this.margin,
  this.transform,
```
We’ll explore all of these in time. The point is that if you need to make a widget look good, you should reach for a `Container`.

Wrap your `Image.asset` in a `Container`, and then add these properties to it:

```dart
Listing 3.11. Add a Container widget

Container(
  margin: EdgeInsets.only(bottom: 100.0),
  padding: EdgeInsets.all(8.0),
  decoration: BoxDecoration(
    color: Colors.blue.withOpacity(0.25),
    borderRadius: BorderRadius.circular(4.0),
  ),
  child: Image.asset(
    'flutter_logo_1080.png',
    width: 100.0,
  ),
),
```

Hot reload one more time. Your app should be pretty. More importantly, you’ve now seen all the basic concepts of Flutter.

#### 3.6  The Element Tree

It turns out that there’s one more tree in your Flutter app: the Element Tree. The element tree represnts the structure of your app, much in the same way that the widget tree does. In fact, there’s an element in the element tree for every widget in the widget tree.

Earlier in this chapter, I described widgets as 'blueprints that Flutter will use to paint elements on the screen'. When I said elements in that sentence, I literally meant the Flutter Element class. Widgets are configurations for Elements. Elements are widgets that have been made real and mounted into the tree. Elements are what are actually displaying on your device at any given moment.

Elements are created by widgets. When a new widget is built, the framework calls `Widget.createElement(this)`. This widget is the initial configuration for the element, and the element has a reference to the widget that built it. The elements that are created are their own tree, as well. The element tree is simple. It’s like the skeleton to your app. It holds the structure of the app, but none of the details. It can look up configuration details via those references to it’s corresponding widget.

Elements are different than widgets because they aren’t rebuilt, they’re updated. When a widget is rebuilt, or a different widget is inserted at some place in the tree by an ancestor, an element can change it’s reference to the widget, rather than being re-created. Elements can be created and destoryed, of course, and will be as a user navigates around the app. But, consider an animation. The animation calls build after every frame change — which is a lot! But it’s likely that the widget is the "same", but some display properties have changed. In that case, the element itself doesn’t have to rebuild, because the tree is still structurally the same.

And this is how Flutter gets away with rebuilding widgets constantly. They’re cheap, and can be replaced in the tree without disturbing the tree that’s on the screen, because it’s handled by elements.

There’s one last important detail I’d like to touch on: State objects are actually managed by elements, not widgets. In fact, Flutter renders based on Elements and state objects, and doesn’t care about widgets at all.

This is all necessary (and optimal), because widgets are immutable. Because they’re immutable, they can’t change their relationships with other widgets. They can’t get a new parent. They have to be destroyed and rebuilt. Elements, however, are mutable, but we don’t have to update them ourselves. We get the speed of mutable elements, but the safety of writing immutable code. It’s a win-win.

>NOTE
Elements, like render objects, are rarely of concern to the developer, but you can create your own.

#### .6.1  Exploring the Element Tree with an exmple
To add a bit more flair to the app (and demonstrate how the element tree works), I want to swap the incredment and decrement buttons each time the "reset" button is pressed. To make this easier, I already made a widget called `FancyButton`. This is a stateful widget that manages its own background color, basically. It’ll have extra flair since the buttons will be different colors.

```dart
class FancyButton extends StatefulWidget {
  final VoidCallback onPressed;
  final Widget child;

  const FancyButton({Key key, this.onPressed, this.child}) : super(key: key);

  @override
  _FancyButtonState createState() => _FancyButtonState();
}

class _FancyButtonState extends State<FancyButton> {
  @override
  Widget build(BuildContext context) {
    return Container(
      child: RaisedButton(
        color: _getColors(),
        child: widget.child,
        onPressed: widget.onPressed,
      ),
    );
  }
}

  Color _getColors() {
    return _buttonColors.putIfAbsent(this, () => colors[next(0, 5)]);
  }
```

In order to use this `FancyButton`, you only need to update the `_buttons` property in the `_MyHomePageState.initState` method. Right now, `_buttons` is being assigned to a list of two `RaisedButton` widgets. Replace `RaisedButton` with `FancyButton` in both occassions — but, keep the configuration!

```dart
// _MyHomePageState.build
initState() {
    super.initState();
    _buttons = <Widget>[
      FancyButton(              // updated
        child: Text(
          "Decrement",
          style: TextStyle(color: Colors.white),
        ),
        onPressed: _decrementCounter,
      ),
      FancyButton(              // updated
        child: Text(
          "Increment",
          style: TextStyle(color: Colors.white),
        ),
        onPressed: _incrementCounter,
      ),
    ];
}
```
If you reload your app now, you have the same app, but the buttons should have different background colors. In order to make those colors update and the buttons swap, update the `_MyHomePageState._resetCounter` method to call `_swap`.

```dart
void _resetCounter() {
    setState(() => _counter = 0);
    _swap()                      // updated
  }


  void _swap() {
    setState(() => _buttons.insert(0, _buttons.removeAt(_buttons.length - 1)));
  }
```

With the `_swap` method being called inside `_resetCounter`, you can hot reload your app and swap the buttons whenever you press the floating action button.

But, it isn’t right. If your code is the same as mine, then this is what’s happening when you press the reset button:

The buttons are indeed swapping places, but the button background colors aren’t swapping with the rest of the button. This is the result of elements and state objects and widgets and how they all work together.

#### 3.6.2  The Element Tree and State Objects
A few things to keep in mind as I explain what’s happening: - State objects are actually managed by the element tree - State objects are long-lived - State objects can be re-used - Element’s have references to widgets

The relationship between a single stateful widget, an element and a state object looks like this:

**Figure 3.10. The relationship between an elemnent and widget**
![](https://dpzbhybb2pdcj.cloudfront.net/windmill/v-7/Figures/element_widget.png)

It’s helpful for me if I consider the element as the brains of the operation. Elements are simple in that they only contain meta-information and a reference to a widget, but they also know how to update their own reference to a different widget if the widget changes.

Anytime Flutter is rebuilding, the element’s reference points the new widget in the exact place of the element’s old reference. When Flutter is deciding what to rebuild and rerender after `build` is called, an element is going to look at the widget in the exact same place as the previous widget it referenced. Then, it’ll decide if the widget is the same (in which case it doesn’t need to do anything), or if the widget has changed or it’s a different widget altogether (in which case it needs to re-render).

**Figure 3.11. Each element points to a different widget, and know’s its type**
![](https://dpzbhybb2pdcj.cloudfront.net/windmill/v-7/Figures/element_widget_relationship_step_1.png)

So, when you swap those two buttons, they replace each other in the widget tree, but the element’s reference points to the same location. Each element is going to look at it’s widget and say "has this widget changed or is it a new widget altogether?". So, we’d expect the element to see that the widget’s color property has changed, so it should in-fact update it’s reference to the new widget.

The problem is in what Elements look at to decipher what’s updated. They only look at two properties on the widget:

* the exact type at runtime
* A widget’s key (if there is one)
In this example, The color of these widgets aren’t in the widget configuration, they’re in the state objects. So, the element is pointing to the updated widgets, and displaying the configuration, but still holding onto the original state object. So, the element is seeing the new widget that’s been inserted into this place in the tree and it’s thinking "There’s no key, and the runtime type is still `FancyButton`, so I don’t need to update my reference. This is the correct widget to match my state object."

**Figure 3.12. The elements think they’re the same widget’s because they’re of the same type**
![](https://dpzbhybb2pdcj.cloudfront.net/windmill/v-7/Figures/element_widget_relationshio_step_2.png)

#### 3.6.3  Widget Keys
You may have guessed, at this point, that the simple solution here is to add a key to each of the `FancyButton` widgets. You’d be correct. When working with widgets in collection, giving them key’s is a good idea. The easiest way to do that is give each a `UniqueKey`.

```
_buttons = <Widget>[
  FancyButton(
    key: UniqueKey(),      // new
    child: Text(
      "Decrement",
      style: TextStyle(color: Colors.white),
    ),
    onPressed: _decrementCounter,
  ),
  FancyButton(
    key: UniqueKey(),     // new
    child: Text(
      "Increment",
      style: TextStyle(color: Colors.white),
    ),
    onPressed: _incrementCounter,
  ),
];
```

**Key types: ValueKey, ObjectKey, UniqueKey, GlobalKey, and PageStorageKey. And, when to use them.**

As you can probably see by the title that there are many types of keys. In fact, though, `PageStorageKey` is a subclass of `ValueKey<T>`, which is a subclass of `LocalKey`. And, that is a subclass of `Key`. `ObjectKey` and UniqueKey also implement `LocalKey`. `GlobalKey` is a subclass of `Key`. They’re all related, and they’re all a `Key` type.

That relationship is hard to follow. I had to triple check the docs to make sure I didn’t mess all those relationships up. Luckily, there’s no reason to memorize all those relationships, I’m just making a point. Which is, the familial relationship of all the keys doesn’t matter. They’re all the same in some way, but they’re all used for specific cases. There isn’t much nuance, so they might as well be different objects in our heads.

All that said, you can put all the keys into two camps: Global and Local.

>NOTE
Using keys, especially GlobalKeys, is generally not necessary or recommended. GlobalKeys could almost always be replaced with some sort of global state management. The execptions to that rule are the issue we’ve seen above, and using some specialized key like `PageStorageKey`

**Global Keys**

Global keys are really used to manage state and move widgets around the widget tree. For example, you could have a GlobalKey in a single checkbox widget, but use the widget on multiple pages. That widget is being re-used by the framework under the hood. When you navigate to different pages to see that checkbox, it’s `checked` state will remain the same. If you check it on page A, it’ll be checked on page B.

**Local Keys**
Local keys are all similar in that they’re scoped to the build context that you created the key in. Deciding which one to use comes down to the case.

* `ValueKey<T>` - Value key’s are best when the object that you’re adding a key to has a constant, unique property of some sort. For example, in a todo list app, each widget that displays a todo probably has a `Todo.text` that’s constant and unique.

* `ObjectKey` - Object key’s would be perfect for the catalog in the Farmers Market app. Consider: two products could have the same title. Two different sellers could sell Brussels Sprouts. And one seller could have multiple products. So, what makes a product unique is the comibation of the product name and the seller name. So the key would be a literal object passed into an ObjectKey:

```dart
var key = ObjectKey({
    "seller": product.seller,
    "product": product.title
})
```

* `UniqueKey` - Unique keys can be used if you’re adding keys to children of a collection and the children don’t know their values until they’re created. In the sample app, the product cards don’t know their color until they’re created, so a unique key is a good option.
* `PageStorageKey` - This is a specialized key used to store page information, such as scroll location.

#### 3.7  Summary
* In Flutter, everything is a widget, and Widgets are just Dart classes that know how to describe their view.
* A widget can define any aspect of an application’s view. Some widgets, such as the `Row`, define aspects of layout. Some are less abstract and define structural elements, like `Button` and `TextField`.
* Flutter favors composition over inheritance. Composition defines a 'has a' relationship, inheritance defines a 'is a' relationship.
* Every widget must contain a `build` method, and that method must return a Widget.
* Widgets should be immutable in Flutter, but State objects shouldn’t.
* Widgets have `const` constructors in most cases. You can, and should, omit the `new` and `const` keywords when creating widgets in Flutter.
* A `StatefulWidget` tracks it’s own internal state, via an associated `State` object. A `Stateless` widget is "dumb" and is destroyed entirely when Flutter removes it from the widget tree.
* `setState` is used to tell Flutter to update some state and then repaint. It should not be given any async work to do.
* `initState` and other life cycle methods are powerful tools on the State object.
* `BuildContext` is a reference to a widget’s location in the widget tree. In practice, this means that your widget can gather information about it’s place in the tree.
* The element tree is the smart one. It manages widgets, which are just blueprints for elements that are actually in use.
* In Flutter, widgets are rendered by their associated `RenderBox` objects. These render boxes are responsible for the telling the widget it’s actual, physical size. These objects receive constraints from their parent, and then use that to determine their actual size.
* The `Container` widget is a "convenience" widget that provides a whole slew of properties that you would otherwise get from individual widgets.
* Flutter 'Row' and 'Column' use the concept of 'flex' layouts, much like flexbox in CSS.
