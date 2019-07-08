> **In this chapter:**
>* Start your first Flutter app
>* User-interface in Flutter
>* Layout widgets
>* Themes and Styling
>* Custom Form elements
>* Builder patterns

Flutter isn’t just a framework. It’s a complete SDK. And, perhaps the most exciting piece of this SDK to me, as a web developer, is the massive library of built-in widgets that make building the 'front-end' of your mobile app easy. Flutter gives you everything you need (and more) to build a beautiful, buttery smooth, performant mobile app user interface.

This chapter, and the following two, are going to be all about user-interface and making an app beautiful. This chapter includes exploring some of the widgets built into Flutter, layout, styling and delight, and more. In the following chapters, I’ll go a bit further into UI and talk about forms and user input, as well animations.

**Figure 4.1. Screenshots of the weather app**
![](https://dpzbhybb2pdcj.cloudfront.net/windmill/v-7/Figures/weather_app_screenshots.png)

This is the app you’re creating. It’s fairly simply in functionality, with only 3 total pages. (But a lot of functionality.)

In this chapter, in particular, we’ll look at these high-level categories:
1. Structural widgets that will outline the app.
2. Theme and Styling, which this app is heavy on. Here we’ll set the custom color scheme and look at the `MediaQuery` class to help styling.
3. Next, we’ll look in the broad category of widgets that help with layout. This includes building block widgets like `Table`, `Stack`, and `BoxConstraint`, as well some fantastic convenience widgets like `TabBar`, which provides entire features for free.
4. Finally, we’ll move onto another section about layout widgets, specifically `ListView`. This widget can be scrollable, and uses something in Flutter called the `builder` patter.

Before we get started, there are a couple caveats and disclaimers that I’d like to mention:

* There’s no way that a single book or app could (or should) cover every (or even most) of Flutter’s built in widgets (or features). The intention of this book is learning, and although you won’t learn about every single widget, you will learn how to find and use what you’re looking for when the time comes. Flutter’s documentation is among the best I’ve ever seen, and all the widgets' descriptions are robust. And as if the Flutter library wasn’t robust enough, the Flutter team is hard at work adding more widgets and plugins everyday. By the time this book comes out, I’d expect the iOS themed widget library to be fully realized.

* Finally, I’ll be glossing over (or not mentioning at all) some of the code in the projects from here on out. This is a book about Flutter, and a lot of code doesn’t have anything to do with Flutter. For example, models are just models, regardless of the language and framework you’re using. I won’t leave you wondering, though. I’ll point out where the relevant code is when the time is right, I just won’t walk through it line by line.

#### 4.1  Setting up and configuring a Flutter app
In the `flutter_in_action` repository, [3] there’s a directory called `weather_app`. This is where the book will begin. The important beginning files look like this:
```
Listing 4.1. Weather app file structure

weather_app
├── README.md
├── lib
│   ├── blocs
│   │   └── forecast_bloc.dart
│   ├── main.dart
│   ├── models
│   │   └── // models...
│   ├── page
│   │   ├── // pages...
│   ├── styles.dart
│   ├── utils
│   │   ├── // many utils files...
│   └── widget
│       ├── // all the custom widget's for this app
├── pubspec.lock
├── pubspec.yaml
```

#### 4.1.1  Configuration: pubspec and main.dart
This is the new configuration items from the pubspec file that you haven’t seen before (in this book).

```
//...
flutter:
  uses-material-design: true
  fonts:
  - family: Cabin
    fonts:
    - asset: assets/fonts/Cabin-Regular.otf
    - asset: assets/fonts/Cabin-Bold.otf
    // ...
```

Along with declaring assets and importing libraries (as discussed in chapter 3), this is pretty much the only information you’ll ever need for your Flutter pubspec file. If you we’re writing an app that’s heavy on iOS style widgets, you can include an additional flag to import iOS icons.

After the pubspec file, move into `weather_app/main.dart`. The place where you’ll always begin. The `main()` function runs the app, as it always must, but it’s also setting up some configuration for the app.

First, it creates an instance of `AppSettings`, which is a class I made to fake persisting user settings in the weather app. I’ll get into this more in a bit.

The main function also talks to the `SystemChrome` class.

#### 4.1.2  SystemChrome
`SystemChrome` is a Flutter class that exposes some easy methods to control how your app displays on the native platform. This is, perhaps, the only class you’ll ever use to manipulate the phone itself. (Unless you’re building plugins, but that’s for much later in the book.)

In this app, I’m using `SystemChrome.setPreferredOrientations` to restrict the app to portrait mode. It also exposes methods to control what the phone’s overlays look like. For example, if you have a lightly colored app, you can ensure that the time and battery icon on your phone’s status bar are dark (and vice versa).
```dart
void main() {
  AppSettings settings = new AppSettings();

  // Don't allow landscape mode
  SystemChrome.setPreferredOrientations(
          [DeviceOrientation.portraitUp, DeviceOrientation.portraitDown])
      .then((_) => runApp(MyApp(settings: settings)));   #1
}
```

> JUST IN TIME: DART FUTURES
> There’s a whole chapter later on async Dart, but you won’t get very far into Dart without seeing some async methods here and there. A Future is the foundational class of all async programming in Dart.
>
> Futures are a lot like getting a receipt at a burger restaurant. You, the burger orderer, tell the employee that you want a burger. The employee says "okay, here’s a receipt. This receipt guarantees that sometime in the future I will give you a burger. Just as soon as it’s ready."
>
> So you, the caller, go wait until the employee calls your number, and then delivers on the guarantee of a burger. The receipt is the Future. It’s a guarantee that a value will exists, but it isn’t quite ready.

Futures are thenable (that is then-able), so when you call a future, you can always say `myFutureMethod().then((returnValue) ⇒ … do some code … );`

`Future.then` takes a callback, which will be executued when the future value resolves. In the burger restaurant, the callback is what you decide to do with the burger when you get it. The value passed into the callback is whatever the return value of the original future is.

That’s it for app configuration. From here, I’ll be talking about widgets the whole time. Starting with the top-level widget: `MyApp` in the `weather_app/main.dart` file.

#### 4.1.3  MaterialApp widget
This widget basically does two things: * sets up the app’s `Theme` * wraps the app in a `MaterialApp` widget

The generic top-level widget provided by Flutter is the `WidgetsApp`. It’s a convenience widget that provides a number of features required for most mobile apps. Most importantly, it helps to provide a `Navigator` to the application, necessary for routing to pages. The `WidgetsApp` requires a good amount of work to set up, because it’s completely customizable. That’s where `MaterialApp` comes in.

`MaterialApp` is an even more convenient widget that extends `WidgetsApp`. It adds Material Design specific functionality and styling options to your app. It doesn’t just help set up the `Navigator`, it does it for you. In short, it provides a ton of functionality to set up your app’s navigation and `Theme`.

“It's called a Material app because it leans on Material style functionality (for example, page animations from route to another are the same as apps on an Android devive). If you have a highly custom app, this could be a concern.”

I don’t see a drawback to using `MaterialApp`, even if you don’t want to use Material design guidelines. Your theme is still fully customizable. (In fact, in this app, you’ll build an app that doesn’t look very "material".) You can overwrite any routes you want. It set’s up quite a bit of convenience, but everything is reversible.

For now, let’s pass by the theme code. I’ll cover `Theme` in-depth in a few pages. For now, look at the bottom of the `build` method, where a `MaterialApp` widget is returned.

```dart
return MaterialApp(
      title: 'Weather App',
      debugShowCheckedModeBanner: false,
      theme: theme,
      home: PageContainer(settings: settings),
    );
```

[3] The repository is at github.com/ericwindmill/flutter_in_action

#### 4.2  Structural Widgets

Along with the `MaterialApp` widget, there are a few other convenience widgets that you’ll likely use in every Flutter app you ever build. Some of the widgets I’ll cover in this section are the `Scaffold`, the `AppBar` and `Theme`.

#### 4.2.1  Scaffold
Like the MaterialApp widget, a `Scaffold` is a convenience widget that’s designed to make applications (which follow Material guidelines) as easy as possible to build. The `MaterialApp` widget provides configuration and functionality to your app. The Scaffold is the widget that gives your app structure. A material app widget is the plumbing of a house. The scaffold is the house’s walls and beams.

Also like the `MaterialApp`, a Scaffold provides a ton of benefit, but doesn’t pigeon-hole you into Material design. Again, even if you have highly custom design style, I don’t see why you wouldn’t use a Scaffold.

Per the Flutter docs, a Scaffold defines the 'basic material design visual layout'. Which means it can make your app look like this pretty easily:

**Figure 4.2. Diagram of the most important Scaffold widget properties**
![](https://dpzbhybb2pdcj.cloudfront.net/windmill/v-7/Figures/scaffold_diagram.png)

And it provides functionality to add a "drawer", (which is mobile UI speak for an element that animates in from one of the sides, and is used for menus), and a 'bottom sheet', which is an element that animates into view from the bottom of the screen. It’s common in iOS style apps. And, unless you configure it otherwise, the AppBar in a Scaffold is automatically set up to display and handle a menu button and back buttons. You get that for free, which is pretty incredible.

Importantly, though,you can pick and choose which features you want and which you don’t. Look at the constructor for a Scaffold widget class:
```dart
Listing 4.3. Scaffold full property list

// From Flutter source code. Scaffold constructor.
const Scaffold({
    Key key,
    this.appBar,
    this.body,
    this.floatingActionButton,
    this.floatingActionButtonLocation,
    this.floatingActionButtonAnimator,
    this.persistentFooterButtons,
    this.drawer,
    this.endDrawer,
    this.bottomNavigationBar,
    this.bottomSheet,
    this.backgroundColor,
    this.resizeToAvoidBottomPadding = true,
    this.primary = true,
  }) : assert(primary != null), super(key: key);
```

I wanted to show this so you can see that none of these properties are marked as `@required`. You can use an `AppBar`, but you don’t have to. Same with drawers, navigation bars, etc. For this app, I only used the AppBar.

You can see the Scaffold in the `ForecastPage` widget of your app. (`weather_app/lib/page/forecast_page.dart`). The part you care about right now is at the very bottom of the file — the return statement of the `ForecastPageState.build` method:
```dart
// weather_app/lib/page/forecast_page.dart
return Scaffold(
  // TODO: Implement AppBar, Preferred Size
  appBar: AppBar(),
  // TODO: Implement Stack, etc
  body: Container(),
);
```

#### 4.2.2  AppBar and PreferredSizeWidget
The AppBar is yet another convenience widget that gives you all kinds of features for free. The AppBar is typically used in the `Scaffold.appBar` property, which fixes it to the top of the screen at a certain height.

The most notable feature the AppBar is that it provides base app navigation actions for free. The AppBar automatically inserts a menu button if the AppBar’s parent is a Scaffold with a drawer. And, if the Navigator of your app detects that you’re on a page that can navigate "back" (like a browsers back button), it automatically inserts a back button. This button is called the `leading` action, and can be configured with the `AppBar.leading` property and `AppBar.automaticallyImplyLeading` property.

**Figure 4.3. Most important properties of the AppBar widget**
![](https://dpzbhybb2pdcj.cloudfront.net/windmill/v-7/Figures/appbar_widget.png)

**Preferred Size**
In Flutter, widgets' sizes are constrained, generally, by the parent. Once a widget knows it’s constraints, it chooses its own final size. The constraints say how big it can be, but isn’t concerned with it’s final size. The advantage to this system (opposed to HTML, for example, where elements control their own constraints), is flexibility. There are some cases where flexibility isn’t desirable, though. For example: the AppBar.

The `AppBar` class extends `PreferredSizeWidget`. The `Scaffold.appBar` property expects a `PreferredSizeWidget` class because it wants to know the size of the AppBar before it sets constraints. In this app, I use a `PreferredSize` directly. The practical application of this is that you can wrap any arbitrary widget in a `PreferredSize` and use it anywhere you’d put an AppBar. (Which is what we’ll do in the next chapter.)
```dart
Listing 4.4. Using PreferredSize in a Scaffold

// weather_app/lib/page/forecast_page.dart -- line ~217
return Scaffold(
  appBar: PreferredSize(
    preferredSize: Size.fromHeight(ui.appBarHeight(context)),
    child: ...
    ),
  ),
```

The point of using a PreferredSize for this app is that it’ll give you the ability to add animations to the app bar later on, which I did by passing in a custom widget `TransitionAppBar`, which isn’t itself an `AppBar`, so it has to be passed into `PreferredSize`.

“Now that the app's structure is taken care of, I want to discuss the last bit of the app's configuration-type-work.”

> NAMED IMPORTS IN DART
Above, I’m calling `ui.appBarHeight`. `ui` refers to the utils file with a name: import `'package:weather_app/utils/flutter_ui_utils.dart' as ui;`

#### 4.3  Styling in Flutter and Theme
Styling your app in Flutter is simpler than you might expect. Perhaps it’s the nature of a mobile app (compared to web apps), but there’s less to worry about. If you’re diligent about setting up a `Theme` when you start your Flutter app, you shouldn’t have to do much work to keep the app looking great.

Along with Themes, the most important pieces of styling in Flutter are Media Queries, setting fonts, animations, and Flutter’s Color class. All of which we’ll explore in this section, save for animations.

4.3.1 Theme
We’ve seen `Theme` before, briefly. It’s a widget that lets you set quite a few color and font that’ll be used by Flutter throughout your app. To give you an idea, these are some (but not all) properties you can set that will drastically change the appearence of your app:

* `brightness` (which sets a dark or light theme)
* `primarySwatch`
* `primaryColor`
* `accentColor`

These are some properties that control specific features:

* canvasColor
* scaffoldBackgroundColor
* dividerColor
* cardColor
* buttonColor
* errorColor

That’s only 6 of about 20 that are available. And if that wasn’t enough for you,there are almost 20 more properties that set defaults for fonts, page animations, icon styles, etc. Some of those properties are robust classes themselves, offering more customizations you can make on your customizations!

That’s pretty cool stuff, but can be overwhelming. Flutter thought about that though, and made it easy. If you’re using the `MaterialApp` widget at the root of your app, every property has a default value, and much of the "down ballot", more specific properties you can override (like button color), are smart enough to change when you only change something like `primaryColor`. In other words, you can be as granular or hands off as you’d like to be. I’ve said it many times, but one of the great aspects of Flutter is how much it does for you until you decide you need more control.

The class you use to configure your Theme is called `ThemeData`, and to add a theme to your app you’d pass a `ThemeData` object to the `MaterialApp`.theme property of your app. You can also create a your own `Theme` widget and pass it a `ThemeData` object. `Theme` is just a widget, which means you can use it anywhere you can use any widget! The theme properties that any given widget uses are passed down from the closest theme widget up the tree. So, you can create multiple `Theme` widgets across your app, which will override the top level theme for everything in that subtree.

If that sounds familiar, it’s because you use `BuildContext` to get your theme information. Recall that getting widgets from `BuildContext` always gets the closest widget up the tree of that type.

```dart
Listing 4.5. ThemeData in the weather app

// weather_app/lib/main.dart
var theme = ThemeData(
  fontFamily: "Cabin",
  primaryColor: AppColor.midnightSky,
  accentColor: AppColor.midnightCloud,
  primaryTextTheme: Theme.of(context).textTheme.apply(
        bodyColor: AppColor.textColorDark,
        displayColor: AppColor.textColorDark,
      ),
  textTheme: Theme.of(context).textTheme.apply(
        bodyColor: AppColor.textColorDark,
        displayColor: AppColor.textColorDark,
```

#### 4.3.2  MediaQuery and the of method
If you came from the web like me, you may (at first) find writing styles in Flutter cumbersome - particularly spacing and layout. There’s only one unit of measurement - the 'logical pixel'. In CSS there are pixels, inches, percentages, viewport based percentage units, and things that are basically made up, like `rem`. The consequence of this is most of the layout and sizing problems are solved with math. (I know, it’s a hard pill to swallow.) Much of this math you’ll want to do is based on screen size, which you can get using the `MediaQuery` widget.

The `MediaQuery` is a widget similar to `Theme` in that you can use the BuildContext to access it anywhere in the app, via its of method. for example, `MediaQuery.of (context).size` gives you a `Size` object with the phones width and height. You can use that information to make dynamic sizing settings. For example, to get 80% of the width of the phone you could write:
```
var width = MediaQuery.of(context).size.width * 0.8;
```
Remember that a widgets build context gives Flutter a reference to that widget’s place in the tree. So, the special `of` method, which will always take a context, regardless of which object its defined on, basically says "Hey Flutter, give me a reference to the nearest widget of this type in the tree, above myself."

MediaQuery is the first place you should look if you’re trying to get specific information about the physical device your app is running on, or if you want to manipulate the device. You can use it to:

* Ask if the phone is currently in portrait or landscape orientation.
* Disable animations and invert colors for accessibility reasons.
* Ask the phone is the user has their text size factor scaled up.
* Set the padding for your entire app. Which you may want to do if a big company releases a new phone with a notch in the top screen that fundamentally changes the way we have to think about mobile app layout.

You can use MediaQuery to set size parameters based on the screen size. That’s what I usually find myself using it for. Let’s look at an example from the weather app.

#### 4.3.3  ScreenAwareSize method
Recall this code that’s using the `Scaffold` in the `ForecastPage`:

```dart
Listing 4.6. Using PreferredSize in a Scaffold

// weather_app/lib/page/forecast_page.dart -- line ~217
return Scaffold(
  appBar: PreferredSize(
    preferredSize: Size.fromHeight(ui.appBarHeight(context)),
    child: ...
    ),
  ),
```

In the file found at `weather_app/lib/utils/flutter_ui_utils.dart`, you’ll find a some code that defines `ui.appBarHeight(context)` from the snippit above:

```
Listing 4.7. Screen aware sizing methods

// weather_app/lib/wutils/flutter_ui_utils.dart

final double kToolbarHeight = 56.0;
double appBarHeight(BuildContext context) {
  return screenAwareSize(kToolbarHeight, context);
}

const double kBaseHeight = 1200.0;
double screenAwareSize(double size, BuildContext context) {
  double drawingHeight =
      MediaQuery.of(context).size.height - MediaQuery.of(context).padding.top;
  return size * drawingHeight / kBaseHeight;
}
```

This is a simple couple of methods which will give us more control over the `PreferredSize` widget, and show off the MediaQuery class.

Back in the `Scaffold` declaration:
```dart
appBar: PreferredSize(
  preferredSize: Size.fromHeight(ui.appBarHeight(context)),
```
This line of code is going to tell the scaffold (the preferred size’s parent), how big it wants to be. Specifically, it tells Flutter to create a `Size` instance from only a height, leaving the work to determine the width up to Flutter. (This makes sense because an AppBar is a full-screen-width UI element.) And you’re getting that height from the method you wrote called `appBarHeight`.

That method simply makes the code more readable, since its called `appBarHeight`, but all it does is turn around and call `screenAwareSize` with an extra parameter, which is what we want to focus on.
```dart
const double kBaseHeight = 700.0;
double screenAwareSize(double size, BuildContext context) {
  double drawingHeight =
      MediaQuery.of(context).size.height - MediaQuery.of(context).padding.top;
  return size * drawingHeight / kBaseHeight;
}
```

That’s it for `MediaQuery`. You should be aware of it, but there’s no need to memorize it’s functionality

#### 4.4  Common Layout and UI widgets
This is going to be the last big section devoted to individual layout widgets and widgets that represent physical UI elements. Of course, in Flutter, everything is a widget, so we’ll never stop talking about widgets. But after this section, we’ll be talking about complex widgets that do *stuff* rather than show *stuff*. In particular in this chapter, I will cover the `Stack`, `Table`, and `TabBar`.

#### 4.4.1  Stack
A `Stack` is just what it sounds like. It’s used to layer widgets on-top of each other. It’s API can be used to tell Flutter exactly where to position widgets relative to the stack’s border on the screen. (Much like `position: fixed` in CSS). In this case, I’ll use it to make a fancy background that reflects the time of day and current weather:

**Figure 4.4. The background of the weather app.**
![](https://dpzbhybb2pdcj.cloudfront.net/windmill/v-7/Figures/sun_and_moon.png)
The sun, the clouds and the content are all different widgets stacked on top each other . All the children of a stack are either positioned or (by default) non-positioned. The Stack is extra nice because it treats it non-positioned children like a column or row treats their children. It aligns it’s children by their top-left corners, and you can tell it which direction to align in with it’s `alignment` property.

In other words, a stack could be used to work exactly as a column, laying it’s children out vertically, unless you explicitly make a child positioned, in which case it’s removed the flow and placed where you tell it to be.

To make a widget positioned you wrap it in a `Positioned` widget. The positioned widget has properties `top`, `left`, `right`, `bottom`, `width`, and `height`. You don’t have to set any of them, but you can only set at most two horizontal properties (left, right, width) and two vertical properties (top, bottom, height). These properties tell Flutter where to paint the widget. The children are painted by the `RenderStack` algorithm:

* First, it lays out all its non-positioned children in the same way a row or column would.
    * This is what tells the stack it’s final size. If there are no non-positioned children, then the stack tries to be as big as possible.

* Second, it lays out all its positioned children relative to the Stack’s render box, using its properties: `top`, `left`, (and so on)
    * The positioned properties tell Flutter where to place the Stack’s children in relation to the Stack’s parallel edge. `top: 10.0` will place the positioned widget 10.0 pixels inset from the top edge of the stack’s box.

* Once everything is laid out, Flutter paints the widgets in order, with the first child being on the 'bottom' of the stack.

**Figure 4.5. An example of using positioned**
![](https://dpzbhybb2pdcj.cloudfront.net/windmill/v-7/Figures/positioned_diagram.png}

In the weather app, I use a `Stack` in the `Forecast` page. In the `Scaffold.body` property, which has three children. It’s children are basically all the content of the forecast page. It looks like this:

```dart
Listing 4.8. Stack code in Forecast Page

Stack(
  children: <Widget>[
    SlideTransition(
        position:
      // ...
      child: Sun(...)),
    ),
    SlideTransition(
      // ...
      child: Clouds(...),
    ),
    Column(
      children: <Widget>[
        forecastContent,
        mainContent,
        Flexible(child: timePickerRow),
      ]
    ),
  ],
),
```

For the sake of example, this is what the app could look like if it wasn’t animated:

```dart
Listing 4.9. Example of non-animated Stack code

Stack(
  children: <Widget>[
    Position(
        left: 100.0,
        top: 100.0,
      child: Sun(...)),
    ),
    Positioned(
      left: 110.0,
      top: 110.0,
      child: Clouds(...),
    ),
    Column(
      children: <Widget>[
        forecastContent,
        mainContent,
        Flexible(child: timePickerRow),
      ]
    ),
  ],
),
```

#### 4.4.2  Table
The final static multi-child widget that I want to show off is the `Table`, which uses a table layout algorithm to make a table of widgets. Along with stacks, rows, and columns, tables are one of the building blocks of layout, as far as non-scrolling widgets.

In the weather app, you’ll use a table to layout some weather data on the lower half of the screen.

**Figure 4.6. Screenshot showing the table widget in the context of the weather app.**
![](https://dpzbhybb2pdcj.cloudfront.net/windmill/v-7/Figures/table_screenshot_annotation.png)

Tables are more strict than other layout widgets we’ve seen, because they have (theoretically) one purpose: to display data. Tables wouldn’t be much use if the columns didn’t line up correctly to display the data (theoretically). Flutter tables require explicit column widths in advance, and no table cell can be empty. Otherwise, we use it similarly to other multi-child widgets. The simple version API looks like this:

```dart
Listing 4.10. The API for the Table widget

Table(
  columnWidths: Map<int, TableColumnWidth>{},
  border: Border(),
  defaultColumnWidth: TableColumnWidth(),
  defaultVerticalAlignment: TableCellVerticalAlignment,
  children: List<TableRow>[]
);
```
There are a few things worth mentioning:
* You don’t have to pass in `columnWidths`, but `defaultColumnWidth` cannot be null.
* `defaultColumnWidth` has a default argument: `FlexColumnWidth(1.0)`, so you don’t have to pass in anything. But, it can’t be `null`. This effectively means you can’t explicitly pass in null: `defaulColumnWidth: null` would throw an error.
* You define column widths by passing a map to the `columnWidths`. The map takes the index of the column (starting at 0) as the key, and how much space you want it to take up as the value. (More in a bit about `TableColumnWidth`.
* The `children` argument expects `List<TableRow>`, so you can’t just pass it any widget! This is a rare occasion in Flutter, but it does happen.
* Border is optional.
* TableCellVerticalAlignment only works if your row’s children are TableCells, which we’ll see in a bit.

With all that in mind, if you only pass in some children, then all the columns will have the same width, because they’re all flexed, which means they size themselves in relationship to eachother. The elements in a row work together to take up the full width. To achieve the look of the table in the weather app, I configure it like this:

**Figure 4.7. Table diagram with borders to show rows and columns.**
![](https://dpzbhybb2pdcj.cloudfront.net/windmill/v-7/Figures/table_diagram.png)

```dart
Listing 4.11. Using FixedColumnWidth on rows 0,2,3, which makes row 1 flexed.

// weather_app/lib/widget/forecast_table.dart -- line ~39
Table(
  columnWidths: {
    0: FixedColumnWidth(100.0),
    2: FixedColumnWidth(20.0),
    3: FixedColumnWidth(20.0),
  },
  defaultVerticalAlignment: TableCellVerticalAlignment.middle,
  children: <TableRow>[...],
);
```

The next piece of the puzzle is a `TableRow`. A table row is quite a bit simpler than a normal row. There are only two things worth mentioning:

1. Every row in a table must have an equal number of children.
2. You can, but don’t have to, use `TableCell` in the children’s sub-widget trees. The `TableCell` doesn’t have to be a direct child of the `TableRow`, either.

In this app, we’re going to use `TableCell`, because it makes alignment super easy on you. They know how to control their children’s alignment in the context of the table.

In the app, the table has four columns and seven rows. It would be quite cumbersome to write 28 widgets, so I’m going generate each row. Later in this chapter, we’ll explore what Flutter calls the builder pattern, and this is a great precursor to that.

**Generate Widgets from Dart’s List.generate() constructor**

This is the first of many times that I can gush about how cool it is that everything in Flutter is Dart code. Rather than pass a list to the `children` property of the table, we can give it a function that returns a list.

```dart
Listing 4.12. Table code from weather app

Table(
  columnWidths: {
    0: FixedColumnWidth(100.0),
    2: FixedColumnWidth(20.0),
    3: FixedColumnWidth(20.0),
  },
  defaultVerticalAlignment: TableCellVerticalAlignment.middle,
  children: List.generate(7, (int index) {
    ForecastDay day = forecast.days[index];
    Weather dailyWeather = forecast.days[index].hourlyWeather[0];
    var weatherIcon = _getWeatherIcon(dailyWeather);
    return TableRow(
      children: [
      // ....
```

This `List.generate` constructor function is going to run as the page is building. It’s basically a loop, so it’s going to run 7 times. (The index at each loop iteration will actually be 0-6, though, importantly.) At each iteration, we have an opportunity to do some logic, we know that in each iteration you have access to an `index`, which is different in each iteration. This means that you can fetch the data for this widget on the fly. Basically this just makes for much cleaner code then writing something like:

```dart
Listing 4.13. Verbose code without using a function

Table (
  children: [
    TableRow(
       children: [
         TableCell(),
         TableCell(),
         TableCell(),
         TableCell(),
       ]
    ),
    TableRow(
      children: [
        TableCell(),
        TableCell(),
        TableCell(),
        TableCell(),
      ]
    ),
    //... etc, 5 more times,
  ]
)
```

ACK! Even having stripped out all the actul content of each `TableCell`, you can see how that’d be too verbose. Especially because each group of rows and cells is the same as each other one. Using a function to programmatically build the rows is nice. The caveat in this example is that this only works because the array of our data is specifically ordered.

Anyways, I get that all I’m doing is creating a list of widgets. I realize that this isn’t an Earth shattering moment for many people. Coming from the web with HTML And CSS, this was an exciting realization for me. I love many web frameworks, like React and Vue, and you can definitely functionally create components like this in JavaScript frameworks, but it’s just so satisfying to me that there isn’t any markup language or styling involved. It’s pure Dart. Just like we like it.

The remaining code to implement is the table rows themselves, which solely display basic widgets.`TableCell`, `Text`, `Icon`, and `Padding` are all used. For the sake of familiarizing your self with Flutter code, here’s a snippet of the rows:

```dart
Listing 4.14. Table cell examples from the weather app

// weather_app/lib/widget/forecast_table.dart -- line ~52
TableCell(
  child: Padding(
    padding: const EdgeInsets.all(4.0),
    child: Text(
      text: DateUtils.weekdays[dailyWeather.dateTime.weekday],
      style: textStyle,
    ),
  ),
),
TableCell(
 child: Icon(
   icon: weatherIcon,
   size: 16.0,
 ),
),
// ...
```

The code that actually adds this table widget to the tree is back in the ForecastPage:

```dart
Listing 4.15. ForecastPage.build method

// weather_app/lib/page/forecast_page.dart
return Scaffold(
      appBar: // ...
      body: new Stack(
        children: <Widget>[
          // ... sun and clouds positioned widgets
          Column(
            verticalDirection: VerticalDirection.up,
            children: <Widget>[
              forecastContent,
              mainContent,
              // Flexible(child: timePickerRow),
            ],
          ),
        ],
      ),
    );
```

#### 4.4.3  TabBar
Tabs are common in mobile apps. Like all UI, Flutter makes it easy on use to work with tabs. In the weather app, I used the `TabBar` widget, which displays it’s children in a scrollable, horizontal view, and makes them "clickable". If you’re using a tab bar, you probably have content on the page that changes as you select different tabs. This is what make’s the TabBar widget extra useful, it provides a simple API to manage keeping the selected tab and corresponding content sections in sync.

**Figure 4.8. Diagram of tab related widgets in Flutter**
![](https://dpzbhybb2pdcj.cloudfront.net/windmill/v-7/Figures/tabs_diagram.png)

**TabController**
Many interaction widgets have corresponding controllers to manage events. In this case, we’re using a `TabController`. The controller is responsible for notifying your app when a new tab is selected, so your app can update when the controller tells it to. The controller will be created higher in the tree than the TabBar itself, and then passed into the tab bar widget. In the weather app all the tab bar code is in the `weather_app/lib/widget/time_picker_row.dart` file.

The `TimePickerRow` object is stateful widget who’s main purpose is to display the tabs, and also tell it’s parent when a tab change event happens, using the `TabController`.

```dart
Listing 4.16. TabController and TabBar widget setup

// weather_app/lib/widget/time_picker_row.dart
class TimePickerRow extends StatefulWidget {
  final List<String> tabItems;
  final ForecastController forecastController;
  final Function onTabChange;
  final int startIndex;
// ...
```
```dart
Listing 4.17. Flutter tabs implementation in weather app

// weather_app/lib/widget/time_picker_row.dart
class _TimePickerRowState extends State<TimePickerRow> with SingleTickerProviderStateMixin {
  TabController _tabController;
  int activeTabIndex;

  @override
  void initState() {
    _tabController = TabController(
      length: utils.hours.length,
      vsync: this,
      initialIndex: widget.startIndex,
    );
    _tabController.addListener(handleTabChange);
    super.initState();
  }

  void handleTabChange() {
    if (_tabController.indexIsChanging) return;
    widget.onTabChange(_tabController.index);
    setState(() {
      activeTabIndex = _tabController.index;
    });
  }
```

Listing 4.18. Just in time: Listeners

This is the first time, I think, that I’ve come accross a `listener` in this book. Listenters aren’t a specific type, but it is a naming convention that’s used for a bunch of async Dart functions and functionality.

There are many places in the Flutter library that you’ll see the word "listener", "change notifier", and "stream". They’re all different flavors and pieces of the same kind of programming concept: observables.

Observables (known as Sinks in Dart) are covered thoroghly later in the book. A `listener` is an aptly named piece of the "observable" ecosystem. Again, this is just a function called `addListener`. There is no class called `Listener`.

A `listener` generally refers to a function that’s called in response to some event. The function is just sitting around, waiting, listening for someone to say "okay, now’s your time to execute."

The tab controller’s `addListener` function is called whenever a user changes the tabs. This gives you a chance to update some values or state whenever a user changes tabs.

The controller also has getters that help you manage your tabs and corresponding content. Inside the `_handleTabChange` method, you could, and probably will, end up doing something like this:

```dart
Listing 4.19. TabController index getter

int activeTab;
void _handleTabChange() {
  setState(() => this.activeTab = _tabController.index);
}
```

**TabBar Widget**
Most of the `TabBar` functionality lives in the controller, but the widget itself is the `TabBar` widget.

```dart
Listing 4.20. TabBar widget in build method

// weather_app/lib/widget/time_picker_row.dart
@override
Widget build(BuildContext context) {
  return TabBar(
    labelColor: Colors.black,
    unselectedLabelColor: Colors.black38,
    unselectedLabelStyle:
        Theme.of(context).textTheme.caption.copyWith(fontSize: 10.0),
    labelStyle: Theme.of(context).textTheme.caption.copyWith(fontSize: 12.0),
    indicatorColor: Colors.transparent,
    labelPadding: EdgeInsets.symmetric(horizontal: 48.0, vertical: 8.0),
    controller: _tabController,
    tabs: widget.tabItems.map((t) => Text(t)).toList(),
    isScrollable: true,
  );
}
```

That’s all there is to it. You have to tell it what it’s controller is and what it’s children are. Other than that, it’s just optional styling properties.

#### 4.5  ListView and Builders
The `ListView` is arguably the most important widget to learn thus far in the book. Which is apparent by the sheer length of it’s documentation page on the Flutter website. It’s not only used very frequently, but it introduces some patterns and ideas that are common in Flutter.

In our app, we’re using a ListView in the `SettingsPage` widget in `lib/page/settings_page.dart`. We’re using (fake, generated) data to build a scrollable list that lets us select which cities the user of the weather app cares about.

**Figure 4.9. Weather App settings page**
![](https://dpzbhybb2pdcj.cloudfront.net/windmill/v-7/Figures/settings_page.png)

According to the docs, a `ListView` is "scrollable list of widgets arranged linearly". In human English, it’s a scrollable row or column, depending on what axis you tell it to lay out on. The power of the ListView is how flexible it is. It has a couple different constructors which let you make choices based on the content of the list. If you have a static, small number of items to display, you can create a ListView with the default constructor, and it’s created with code very similar to a row or column. This is the most performant option, but maybe not ideal if you have tens or hundreds of items to put in the list.

What I want to focus on for now, though, is the "builder" pattern in Flutter. The builder pattern is found all over Flutter, and it essentially tells Flutter to create widgets as needed. In the default ListView constructor, Flutter builds all the children at once, and then renders. The `ListView.builder` constructor takes a callback in the `itemBuilder` property, and that callback should return a widget, in this case a `ListTile` widget. This builder makes Flutter super smart about rendering if you have a huge (or infinite) number of list items to render. Flutter will only render the items that are visible on the screen. Imagine a social media app like Twitter, that’s essentially an infinite list of Tweets. It wouldn’t be possible to render all the Tweets in that list every time some state changes, so it renders them as needed.

In the app, `ListView` is used on the `SettingsPage`.

```dart
Listing 4.21. ListView builder code in the settings page

// weather_app/lib/page/settings_page.dart -- line ~58
Expanded(
  child: ListView.builder(
      shrinkWrap: true,
      itemCount: allCities.length,
      itemBuilder: (BuildContext context, int index) {
        var city = allCities[index];
        return CheckboxListTile(
          value: widget.settings.selectedCities[city],
          title: Text(city),
          onChanged: (bool b) => _handleCityActiveChange(b, city),
        );
      }),
)
```

The list view builder is the simplest way to create a scrolling list with potentially infinite items, but there are a couple other constructors:

* `ListView.separated` is similar to `ListView.builder`, but it takes two builders. One to create the list items, and a second to create a separator that will be placed between every list item. Pretty neat.

* `ListView.custom` is the final constructor. As the name suggests, it allows you to create a list view with custom children. This isn’t quite as simple as updating the `builder`. Suppose you have a list view in which some list items should be a certain widget, and other list items an entirely different widget. This is where the custom list view comes into play. It gives you fine-grain control on all aspects of how the list view renders its children. We’ll see more custom widgets later in this book.

Lastly, The ListView can optionally be given a `ScrollController`, which can be used to set the initial scroll offset, if you don’t want to start the beginning. You can also use `PageStorageKeys` here to make Flutter remember the users scroll position when they navigate off a page and back.

Sincerly, the `ListView` is one of the widgets that just beautifully represents Flutter as a whole. It’s clean and functional, but gives you something that would be quite a bit of code to write yourself.

#### 4.6  Summary
Flutter includes a host of convenience, structural widgets, like `MaterialApp`, `Scaffold`, and `AppBar`.
These widgets give you an incredible amount for free: navigation, menu drawers, and more.
Use the `SystemChrome` class to manipulate features of the device itself.
Use `MediaQuery` to get information about the screen size.
Use `Theme` to create a custom look and feel in your app.
Use the `Stack` widget to overlap widgets anywhere on the screen, and the `Table` widget to lay widgets out in a table.
The `ListView` and it’s builder constructor give you a fast, performant way to create lists with infinite items.