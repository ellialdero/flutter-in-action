> **In this chapter:**
>* StatefulWidget and State object
>* Widget tree vs Element tree
>* State object lifecycle
>* InheritedWidget
>* Intro to Streams and Async Dart
>* Bloc state management architecture

This chapter is going to be a ton of fun, for two reasons: First, there isn’t one approach state management (in Flutter or elsewhere). There are many different state management patterns, and all developers have opinions about each of them. And second, developers (including myself) are… passionate… about their opinions.

With that in mind, I won’t be able to do a deep dive on every pattern that’s popular right now. I thought a lot about which patterns I should cover, and I came up with this two-pronged litmus test to decide:

1. Will it help you expand your Flutter and Dart skills specifically?
2. Is it un-opinionated enough that it’s concepts can be applied elsewhere?
This is what I came up with fo

This is what I came up with for this chapter, in general: 1. Deep dive into the StatefulWidget. This is information you need regardless of your state management approach. 2. The `InheritedWidget`. The inherited widget the "third" widget (along with Stateless and Stateful). It’s enough to handle state in a smaller app on it’s own, and all heftier libraries likely use this under the hood. 3. Google’s "BLoC" pattern for app state management. I basically decided that this was the best pattern to spend time on because it’s not verbose and it doesn’t require an outside library. Patterns like Redux have a crazy amount of boiler plate code, you have to pull in an outside library, and they’re extremely opinionated. It’s easy to get in the weeds.

The next section is all about StatefulWidgets and State objects. It’s mostly conceptual. The code-writing will begin in the following section.

#### 8.1  Deep Dive into StatefulWidget
Here’s a small review: A `StatefulWidget` has two jobs: It holds on to immutable variables, just like a `StatelessWidget`. No property on the `StatefulWidget` can’t be updated. Its second job is to create an associated `State` object.
```dart
Listing 8.1. StatefulWidget example
class ItemCounter extends StatefulWidget {
    final String name;

    ItemCounter({this.name});

    @override
    _ItemCounterState createState() => _ItemCounterState();
}
```

The `State` object has many jobs. It’s basic jobs are keeping track of internal, mutable state, and building child widgets with `State.build`. But, the state objects in Flutter can get a bit trickier than that. It’s worth knowing how they’re treated differently in the Widget tree, as well as how to deal with its lifecycle.

#### 8.1.1  The Widget Tree and the Element Tree
Flutter knows how to render widgets to the screen by building the Element Tree. The Widget tree isn’t directly rendered, because Widgets are "blueprints" for renderable elements. We, the developers, build a Widget tree of blueprints, which Flutter internally maps to an element tree. It’s worth reviewing how different types of widgets interact with the element tree. A stateless widget gets mapped, one to one, to an element. As the app is rendering and Flutter is crawling the widget tree, it’ll say: "Hey Element tree, can you make an element that corresponds to this widget?" And it does.

The element tree doesn’t handle stateful widgets the same way. When Flutter asks the Element tree to make the stateful widget, the Element tree says "Sure. Hey new stateful widget, can you create a state object for me?" And so it does.

![](https://dpzbhybb2pdcj.cloudfront.net/windmill/v-7/Figures/element_widget.png)

If this handled internally, why do we care? It’s useful to understand because state objects and elements are long-lived. If the StatefulWidget is replaced in the tree, but the new widget is the same type (and has the same key), the corresponding element just keeps pointing to the same spot in the tree, and references the new stateful widget, but the associated `State` object stays right where it is, and is reused.

Flutter provides methods on the state object that allow you to respond to changes in the element tree. These methods are called in a specific sequence. This sequence is generally referred to as the widget’s lifecycle.

#### 8.1.2  The StatefulWidget Lifecycle and When to Do What
I like to think of the lifecycle in two pieces: The main thread, which is a sequence of methods that will be called in order, and will be called at least once in the state object’s lifecycle no matter what. The second part is three methods that may be called depending on different events. They all trigger rebuilds.

>Note
After this section, we’ll start writing code. In the code, you’ll see these lifecycle methods used and I’ll explain what’s happening with them. Before that, it’ll be helpful to give you a quick overview of the whole process. Don’t get bogged down in the details quite yet.

This is the lifecycle, in order:

![](https://dpzbhybb2pdcj.cloudfront.net/windmill/v-7/Figures/stateful_lifecycle.png)

1. The state class constructor is called (as it is for every class in Dart). The widget isn’t in the tree yet, so most state-specific initialization shouldn’t be done here.
2. The state object is associated with a `BuildContext`, or location in the tree. The widget is now considered `mounted`. You can check if a widget is mounted with `Widget.mounted`.
3. `State.initState` is called. This method is called exactly once. This method should be used to initialize properties on the `State` object that depend on it’s associated stateful widget or build context.
4. `State.didChangeDependencies` is called. This method is special because it’s called once immediately after `initState`, but can also be called later in the lifecycle (which I’ll cover in a minute). This method is where you should do initialization that involves an `InheritedWidget`.
5. At this point, the state is considered "dirty", which is how Flutter tracks which widgets need to be rebuilt. Any time a state object needs to be built, including the first time, it marks itself as "dirty".
6. The state object is fully initialized and the `State.build` method is called.
7. After a new build, the state is marked "clean". Up to this point, the lifecycle has been on a single track. When the state is "clean", nothing is happening. The state object is displayed as it’s intended, and it’s waiting for the framework to give it further directions. Several things can happen now.
    1. `state.setState` is called from your code, which always marks the state as "dirty".
    2. An ancestor widget may request that this location in the tree be rebuilt. If the location is to be rebuilt with the same widget type and key, then the framework will call `didUpdateWidget` with the previous widget as an argument. This also marks the state as "dirty", and thus rebuilds the state.
    3. If your widget depends on an `InheritedWidget`, and that inherited widget changes, then the framework calls `didChangeDependencies`. At this point, the widget will be rebuilt.
    4. Finally, there is one action that is guaranteed to occur. The state object is going to be removed from the tree. Flutter will try to reuse state objects and it’s subtree in another location, but if can’t `State.disposed` is called. This method is where you should clean up any resources used by the widget, such as stopping active animations or closing streams. Once `disposed` is called, the widget can never build again. It is an error to call `setState` at this point.

That list is a primer for the rest of this chapter. Again, no need to get too bogged down, yet.

Next, you’re going to see different state management patterns. State management is a combination of passing data around the app, but also re-rendering pieces of your app at the right time. All the re-rendering in Flutter is dependent on the `State` object and it’s lifecycle.

#### 8.2  Pure Flutter state management: The InheritedWidget
The most basic state management in Flutter is just passing state around the tree from widget to widget. This can get cumbersome, and I can’t recommend you do that. Next, you might try a slightly better version known as *lifting state* up.

Lifting state up is a pattern in which mutable state lives high in the widget tree, and is managed by passing properties way down the tree, as well as passing methods that call `setState` way down the tree. This can make your state hard to reason about, and it requires a lot of extra code to pass a property down from widget to widget.
Figure 8.2.
![](https://dpzbhybb2pdcj.cloudfront.net/windmill/v-7/Figures/naive_tree.png)

More importantly, it makes development more painful than it needs to be. If you had a tree that looked like Figure 8.2, “Figure 8. 1” , and you decided you’d rather the `AppBarCartIcon` to be a child of the `CartPage`, you’ll have to remove all the code that passes properties down to the icon, and then add all the code to pass that information down through the `Navigator` and to the cart page.

![](https://dpzbhybb2pdcj.cloudfront.net/windmill/v-7/Figures/moving_button.png)

Luckily, flutter gives us a better way: the `InheritedWidget`.

You’ve likely seen inherited widgets before: `Theme`, `MediaQuery` and `Scaffold` are all inherited. These widgets are special because any widget in the inherited widget’s subtree can access the inherited widget. You don’t have to pass properties from widget to widget, because you can just grab the properties directly from the InheritedWidget.

Then, if you want to move widgets around the tree in development, you don’t have to change the code of any other widgets. You can just move it.

#### 8.2.1  Creating a Central Store wth an InheritedWidget + StatefulWidget team
The boiler plate for an inherited widget as a state management device can be verbose. If you look at the documentation of the `InheritedWidget` class, you’ll see that it’s a third type of widget, and importantly, not an extension of the stateful widget class. According to the documentation this widget is a "base class for widgets that efficiently propagate information down the tree." [1] The point is, inherited widgets are meant to send information, not be sent information. This means we have to combine a stateful widget with an inherited widget to make it work as a central storage.

In the code, start by looking in `lib/main.dart` file, where runApp is being called:
```dart
// e_commerce/lib/main.dart -- line ~36
runApp(
    AppState(
        blocProvider: blocProvider,
        child: ECommerceApp(),
    ),
);
```

Sometimes when I think about how this all works together, it confuses me. It’s a bit tricky. But, The trick of the whole thing is in the `AppState` class. This is just like any other stateful widget, with one notable exception. The static `of` method that’s been defined on it.

#### 8.2.2  The 'inheritFromWidgetOfExactType' and 'of' methods
If you want to get information from an inherited widget elsewhere in your widget tree, you have to call `BuildContext.inheritFromWidgetOfExactType`. This method looks up the tree and finds the closest parent inherited widget of that type. For example, when you call `Theme.of(BuildContext).primaryColor`, Flutter is looking up the tree for the nearest `Theme`, and grabbing the `primaryColor` property from it.
![](https://dpzbhybb2pdcj.cloudfront.net/windmill/v-7/Figures/theme_of.png)

The `of` method is a Flutter convention, not something you get out of the box. Most `of` methods are defined on the inherited widgets themselves, and all they do is call `BuildContext.inheritFromWidgetOfExactType` with the `BuildContext` you provide. They should be static methods.

The `of` method is really the secret to the inherited widget.

```dart
Listing 8.2. Custom 'of' method

// e_commerce/lib/blocs/app_state.dart -- line ~25
static _AppState of(BuildContext context) {
    return (context.inheritFromWidgetOfExactType(_AppStoreContainer)
            as _AppStoreContainer).appData;
}
```
There are three important pieces of that method:
1. AppState - the widget that you want to handle.
2. BuildContext - You don’t want a new instance of AppState. You specifically want the one that’s already managing state. In other words, the one that is associated with the current build context.
3. of - The cleanest way to say that you want a specific instance of AppState.

The `of` method is the secret sauce to using the stateful and inherited combo as a central store. In the code, add this method to the `AppState` class in `lib/blocs/app_state.dart` file:

That method is really cool. It get’s me excited. It’ll be more exciting when you look at the `AppStoreContainer` inherited widget itself.

```dart
Listing 8.3. The inherited widget, which handles a state object.

// e_commerce/lib/blocs/app_state.dart -- line ~49
class _AppStoreContainer extends InheritedWidget {
  final _AppState appData;
  final BlocProvider blocProvider;

  _AppStoreContainer({
    Key key,
    @required this.appData,
    @required Widget child,
    @required this.blocProvider,
  }) : super(key: key, child: child);
  ...
```
So now, we have a method called `of` that gives us reference to the `InheritedWidget.appData` property anywhere in the widget subtree. And, the `InheritedWidget.appData` is just a state object! Which means you can access the same state object anywhere in your app. If that doesn’t excite you, then I don’t know how to please you. This is one of my favorite built-in Flutter features.

Finally, I just want to take a look at the `_AppState.build` method.

```dart
Listing 8.4. _AppState.build method

// e_commerce/lib/blocs/app_state.dart -- line ~36
@override
  Widget build(BuildContext context) {
    return _AppStoreContainer(
      appData: this,
      blocProvider: widget.blocProvider,
      child: widget.child,
    );
  }
```
![](https://dpzbhybb2pdcj.cloudfront.net/windmill/v-7/Figures/inherited_process.png)

It’s also important to note that the `InheritedWidget` class requires a child, but doesn’t have a `build` method. The inherited widget just takes the child given to it and passes it through via it’s super class, a widget. It doesn’t concern itself with displaying an UI.

**The updateShouldNotify**

As a quick aside, I need to talk about the single required method on the `InheritedWidget` class: `updateShouldNotify`. When an inherited widget rebuilds, it may need to tell all the widgets that depend on its data to rebuild as well. This method is called after rebuild, and always passes in the old widget as an argument. This gives you a chance to check if Flutter should rebuild or not. For example, if your new widget is rebuilt with the same data, then there’s no need to make Flutter do the expensive work.

```dart
Listing 8.5. InheritedWidget.updateDidNotify method

// e_commerce/lib/blocs/app_state.dart -- line ~61
bool updateShouldNotify(_AppStoreContainer oldWidget) =>
      oldWidget.appData != this.appData;
```

#### 8.2.3  Use the of method to lift state up
Using an inherited widget at the top of the tree (as in inheritedTree) is another, cleaner style of "lifting state up", in a way. Your state still all lives in a widget at the top of the tree, but it’s easier to manage and reason about. Using this method, all of your setState methods can live in the `_AppState` state object, and you can call them using `AppState.of(context).callMyMethod()`.

In the code, you can see the `of` method in action with some methods in `lib/blocs/app_state.dart`. In the `_AppState` class, find the comment that says `LIFTING STATE UP REGION`. Here, you’ll find a couple class members that use the 'lifting state up' method. This is what the code looks like:

```dart
Listing 8.6. Functionality to control quantity with InheritedWidget
// e_commerce/lib/blocs/app_state.dart -- line ~49
// ...
int cartCount = 0;
void updateCartCount(int count) {
  setState(() => cartCount += count);
}
// ...
```

When `_AppState.setState` is called, is rebuilds itself. This causes its inherited widget child to rebuild, and then it calls `updateShouldNotify`. If that returns true, then `didChangeDependencies` is called on stateful widgets that depend on this inherited widget, and then they’ll get rebuilt as well.

I use that `updateCartCount` method in the `AddToCartBottomSheet`. Notice that this class has overridden `didChanceDependencies`. This is important if you’re using inherited widgets, because if those widgets change for any reason, they widgets will need to reassign their reference to the updated inherited widget.

```dart
Listing 8.7. Override the didChangeDependencies method
// e_commerce/lib/widget/add_to_cart_bottom_sheet.dart -- line ~49
class _AddToCartBottomSheetState extends State<AddToCartBottomSheet> {
int _quantity;
AppState state;

@override
void didChangeDependencies() {
  super.didChangeDependencies();
  state = AppStateManager.of(context);
}
// ...
```

Second, you need to interact with that `AppState` object. This is done in the `build` method, in the RaisedButton widget. The relevant code looks like this:

```dart
Listing 8.8. Call method from inherited widget from the bottom sheet.
// e_commerce/lib/blocs/add_to_cart_bottom_sheet.dart -- line ~75
//...
RaisedButton(
  color: AppColors.primary[500],
  textColor: Colors.white,
  child: Text(
    "Add To Cart".toUpperCase(),
  ),
  onPressed: () => state.updateCartTotal(_quantity)
//    onPressed: () => Navigator.of(context).pop(_quantity),
)
//...
```

That’s the entire 'lifting state up' pattern. If you’re using the inherited widget as your store, then all changes to app-wide state should be done in that `AppState` class. This makes it way less likely to get in sticky situations with state management (compared to just passing state around willy-nilly).

#### 8.2.4  State management patterns beyond Flutter
In the beginning chapter I expressed the anxiety around the options and opinions of state management. To touch on that a bit more, I want to talk about all the options in Flutter.

It’s important to note that Flutter is just the rendering layer of your app. (I mean, it’s so much more, and it gives us so much, but as far as writing code goes, its a UI library.) That in mind, you can use what ever state management patterns you want. Flutter doesn’t care about how it gets data, it only cares about painting that data on the screen.

There are fantastic libraries made by the community like `Redux`, `MobX`, and `ScopedModel`. [2] Of course, no option is better than another. And everyone has opinions. But don’t listen to those opinions, use the patterns that work for you. Just this morning on Twitter I saw a big name from the JavaScript world tweet something about how awful event emitters are. This guy is undoubtedly brilliant, but I love event based architecture. We have different opinions, and that’s great.

That in mind, I can’t cover everything, so I’m going to cover what I like the most. But more importantly, I think it’s the pattern that makes the most sense to start with. Redux is great because it abstracts away so much logic. As long as you follow the pattern, it’s likely going to work. But I don’t want to abstract that much away. Under the hood, Redux uses inherited widgets and event emitters. The BLoC pattern deals with these directly. It hard to get very far in Dart programming or Flutter without streams and the `InheritedWidget`, so I’d like you to learn how those fit into the whole situation. Then, when you want to switch to Redux, the whole thing will make more sense because you have a nice foundation.

[1] Documentation can be found at https://docs.flutter.io/flutter/widgets/InheritedWidget-class.html

[2] You can see excellent code examples of different state management and architecture styles at fluttersamples.com/, a helpful site by community leader Brian Egan. Find Brian here: github.com/brianegan.

#### 8.3  Blocs: Business Logic Components

"B.Lo.C." stands for "Business Logic Components" (or "blocs" from here on out). This pattern was first revealed at DartConf2018, and it’s purpose is to make UI business logic highly reusable. Specifically at DartConf, it was presented as a nice way to share all UI logic between Flutter and AngularDart.

The bloc pattern’s mantra is that widgets should be as dumb as possible, and the business logic should live in separate components.This in itself isn’t unique compared to other approaches, of course, but the devil is in the details. In general, blocs are what they are for two main reasons:

1. Their public API consists of simple inputs and outputs only.
2. Blocs should be injectable, which means platform agnostic, which means you can use the same blocs for Flutter and Web.

Those are broad ideas, of course, but they’re made clearer by the following "non-negociable" rules. These rules were described in the original talk at DartConf2018 and live in two categories: application design and UI rules:

Application design: 1. Inputs and Outputs are sinks and streams only! No functions, no constants, no variables! If you aren’t familiar with streams, put a pin in your questions for a couple more paragraphs. 2. Dependencies must be injectable. If you’re importing any Flutter libraries into the bloc, then that’s UI work, not business logic, and should be moved into the UI. 3. Platform branching is not allowed. If you find yourself in a bloc writing `if (device == browser)…`, then you need to reconsider. 4. Do whatever else you want, so long as you follow those rules

UI functionality:
1. In general, blocs and top-level Flutter pages have a 1-1 relationship. In reality, the point is that each logical state subject has its own bloc. For example, in the Farmers Market app, I have a CartBloc and a Catalog bloc.
2. Components should send inputs “as is”, because there shouldn’t be business logic in the widget! If you need to format text or serialize a model, it should be done in the bloc.
3. Outputs should be passed to widgets ready to use. For example, if you have a number that needs to be converted into displayable currency, that should be done in the bloc.
4. Any branching should be based on simple bloc bolean logic. You should limit yourself to a single boolean stream in the bloc. For example, in Flutter, it is acceptable to write color: `bloc.isDestructive ? Colors.red : Colors.blue`. It is considered wrong if you have to use complex boolean clauses like `if (bloc.buttonIsDestructive && bloc.buttonIsEnabled && bloc.userIsAdmin) { …`

Of course, who am I (or the speaker at DartConf) to attach hard rules to how you design your app? The rules are intended to ensure that your Flutter app is as simple and dumb as possible, but it’s merely a suggestion.

With all that information, this is what the basic view layer architecture of a Flutter app that uses blocs looks like:

Figure 8.3. View layer design with blocs
![](https://dpzbhybb2pdcj.cloudfront.net/windmill/v-7/Figures/view_layer_app_structure.png)

**Note**
>This chart doesn’t represent how data flows from widgets to blocs, it shows that multiple widgets communicate with the same, few blocs that represent a logical piece of state, in this case the cart bloc and catalog bloc.
>   These blocs are the middleman between services, backends, and even between two widgets. If you’re using blocs, there aren’t many cases when you should be using a `StatefulWidget`. (The general exception being widgets that are in forms that haven’t been submitted yet and animation state.) Wherever possible, your widgets should be dumb, stateless widgets who’s only job is rebuilding when it’s told to. Flutter provides a built in solution to avoiding `StatefulWidgets` with blocs, which we’ll cover in a future chapter on Async Flutter.

#### 8.3.1  Blocs. How do they work?
In general, blocs have two jobs. They should expose methods that allow widgets to update state (data flows in) and they should tell widgets when there’s new information and they need to re-render.

The following diagram shows how a user might checkout in an e-commerce app. It also shows how an outside source might give the bloc new information, and the bloc would know how to update the relevant widgets. Importantly, though, that is two different processes. Perhaps the one of the processes is the result of the first, but they’re still separate.

![](https://dpzbhybb2pdcj.cloudfront.net/windmill/v-7/Figures/bloc_diagram.png)

In this diagram, let’s start at the top-left number one step and talk about the steps.

1. A user might tap a "submit" button, which calls `CartBloc.checkOut`.
2. This method does handles whatever business logic it needs to, like serializing some models into JSON perhaps, and then calls out to a service.
3. This service calls `hypotheticalBackend.submitPayment`.

That’s the end of that process. The process that starts in the bottom right corner is triggered not by the previous process, but by the backend replying with new information, which this bloc is listening for.

1. Perhaps the information that comes back is that this item is that this transaction was successful. When the bloc gets that information from a backend, it now says "great, I’ll update some internal state for the user".
2. Because the app is made of dumb widgets that just do what they’re told, they’re waiting to be told to rebuilt when said internal state updates.
3. The widgets rebuild to show the new information.

Those processes, again, are related, because one relied on the first, but they’re independent and one could happen without the other.

That’s the basic tenant of blocs. They take in information or data, manipulate it, and pass it back out to the appropriate party.

#### 8.3.2  Implementing the Bloc architecture
The architecture of the bloc pattern isn’t much different than the `InheritedWidget` example. The only difference is that the `AppState` passed around via the inherited widget is going to expose the blocs to the rest of your widgets. Let’s talk about what needs to happen for this to work.

* Revert or update some of the code changed for the `InheritedWidget`
* Feed the blocs to an inherited widget
* Connect the blocs to whatever external APIs they need
We’ll talk about what goes inside the blocs in the next section, but first let’s wire it all up.

**AppStateManager and BlocProvider**
The first thing you have to do to make this work is uncomment some code! In order to stop using the lifting state up methods from the previous section, you should go to the `AppBarCartIcon` widget and comment out the `Text` at line ~43, and uncomment the `StreamBuilder` below that. After you’re done, it should look like this:

```dart
// ecommerce/lib/widget/appbar_cart_icon -- line ~43
//child: Text(
//  AppStateContainer.of(context).cartCount.toString(),
//  style: new TextStyle(fontSize: 8.0, color: Colors.white),
//  textAlign: TextAlign.center,
//),
  child: StreamBuilder(
    initialData: 0,
    stream: _bloc.cartItemCount,
    builder: (BuildContext context, AsyncSnapshot snapshot) => Text(
          snapshot.data.toString(),
          style: new TextStyle(fontSize: 8.0, color: Colors.white),
          textAlign: TextAlign.center,
        ),
  ),
```

And also, switch out the `onPressed` callbacks in the RaisedButton in the `AddToCartBottomSheet`.
```dart
// ecommerce/lib/widget/add_to_cart_bottomsheet.dart -- line ~75
RaisedButton(
  color: AppColors.primary[500],
  textColor: Colors.white,
  child: Text(
    "Add To Cart".toUpperCase(),
  ),
//  onPressed: () => state.updateCartTotal(_quantity)
  onPressed: () => Navigator.of(context).pop(_quantity),
```

The solution I’ve chosen to work with the blocs involves the InheritedWidget class we’ve already written, but it’s worth noting that blocs and inherited widgets don’t care about each other. You could also use blocs by passing them down the widget tree.

To start, navigate to `lib/bloc/app_state.dart`. The following pieces of code matter to us in this file right now:
```dart
Listing 8.9. The BlocProvider class, used for cleaner code

class BlocProvider {
  CartBloc cartBloc;
  CatalogBloc catalogBloc;

  BlocProvider({@required this.cartBloc, @required this.catalogBloc});
}
```

This class is used solely to make it easier and cleaner to pass the blocs around. This is defined at the bottom of the file, but it’s used at the top, where I want to start explaining the functionality. Recall that this is the same class that has defined the `of` method.

```dart
Listing 8.10. AppStateContainer stateful widget

class AppStateContainer extends StatefulWidget {
  final Widget child;
  final BlocProvider blocProvider;
  const AppStateContainer({
    Key key,
    @required this.child,
    @required this.blocProvider,
  }) : super(key: key);
  //...
```
Next, the associated state class, `AppState` has two relevant pieces of code, a simple getter, and the `build` method.

```dart
Listing 8.11. Relevant pieces of the AppState class

class AppState extends State<AppStateManager> {
  BlocProvider get blocProvider => widget.blocProvider;

//...

@override
  Widget build(BuildContext context) {
    return _AppStateContainer(
      appData: this,
      blocProvider: widget.blocProvider,
      child: widget.child,
    );
  }
}
```

That’s all it takes. Now those blocs will be exposed to the entire app.

**Referencing the blocs in widgets**
Referencing the blocs in the widgets is the same as referencing other `InheritedWidget` properties from earlier in this chapter. I’ll show an example from the `lib/widget/appbar_cart_icon.dart` file.

First, at the top of the `AppBarCartIcon.build` method, I’m grabbing a reference to the `CartBloc`.

```dart
// ecommerce/lib/widget/appbar_cart_icon.dart -- line ~14
var _bloc = AppStateContainer.of(context).blocProvider.cartBloc;
```

Then, in the same file, head down to line ~49, the `StreamBuilder`.

**Warning**
Don’t get bogged down in this snippit. There’s an entire chapter coming up on async Flutter, including `StreamBuilders`. I’ve annotated the important part:

```dart
Listing 8.12. The Flutter stream builder example

child: StreamBuilder(
    initialData: 0,
    stream: _bloc.cartItemCount,
    builder: (BuildContext context, AsyncSnapshot snapshot) => Text(
        snapshot.data.toString(),
        style: new TextStyle(fontSize: 8.0, color: Colors.white),
        textAlign: TextAlign.center,
    ),
),
```

At this point, the blocs are wired up and working with InheritedWidgets, which expose the blocs to the rest of the app. Now, we can implement the bloc itself, but only after we touch on streams in Dart.

#### 8.3.3  Intro to Streams and Async Dart
Streams are big part of Dart programming. A stream is an object in Dart, but it’s also an asynchronous programming concept. In many languages, streams are called observables. From a high-level, streams just provide a way to emit events and have other classes listen for and respond to these events.

**Note**
`Stream` is actually a specific class, and only one piece of the observable pattern. In general, though, "stream" is the word used to describe the concept as a whole.

The word "stream" is apt for this class, because streams emit new values repeatedly, as often and for as long as it needs to. Information is emitted by a `StreamController`, and then flow’s "down the stream". Elsewhere, objects can "listen" to the stream and grab the values that are flowing down the stream.

It might be helpful to think about streams as a collection of 3 pieces all of which have specific jobs.

1. The `StreamController` is the object that you pass new values to, and it turns around emits the event.
 
>NOTE
A `Sink` is a specific subtype of stream controller — the standard type. When discussing the concept, "sink" refers to a stream controller.

2. The stream itself, which is all the new events emitted from the controller.
3. The listener, which is the object that is notified when new information is emitted down the stream.

For a less abstract example, consider our Farmer’s Market app. Each time a an item is added to a users cart, the cart icon should update with the new quantity of items. You could achieve this with a stream.

When an item is added to the cart, your code would also give the updated quantity to the stream controller. The controller "adds this number to a stream", which means effectively means that it sends a message to everyone who’s listening to the stream. Each time a listener is notified, it will call a callback that you provide. For example, the cart icon may call a function that changes the quantity and then calls set state, triggering a re-render.

I don’t want to belabor this concept, but I’m going to give one more example, because it’s super important for the rest of this chapter, but it isn’t easy.

Imagine a stream as an actual stream of water. The `StreamController`, when handed a new value, would toss the new value into the stream. The value would flow down the stream, and any listener of that stream would see the new value and then perform whichever action you’d like it to with the new information.

You’ve actually already worked with streams once in this book. RouteObservers in Flutter function the same way as streams. When a the route changes, the navigator sends a message to all route observers and says "hey, just to let you know, the route changed", the route observer then takes the appropriate action.

The most important single idea to grasp with streams is that you don’t know when or if you’re going to get new information. Rather than manually determining when to update code in response to an event, stream listeners just sit there, waiting patiently for new information.

#### 8.3.4  Implementing Streams in the CartBloc
That’s how streams work conceptually, but implementing them is different, of course. As I’m writing this, I’m realizing that blocs are the perfect example to teach streams, because all the stream logic lives side by side in one class. For the rest of this chapter, you’ll implement the `CartBloc` class and I’ll show the code for streams along the way.

The base of all blocs is inputs and outputs. Like I mentioned earlier, blocs are all about taking in some data, handling manipulating the data in the bloc itself, and then out putting data, if necessary. Inputs are basically whatever we want to tell the bloc, and outputs are basically whatever we want to ask the bloc.

The input/output API the CartBloc will be simple. It will look like this:

* Inputs
    * Add an item to the cart
    * Remove an item from the cart
* Outputs
    * Get all items in cart
    * Get number of items in the cart

**Add an item to the cart**
To add an item from the cart from widgets in the app, it’s best if you can make it as simple and dumb as possible from the widgets point of view. Ideally, you could add an item from the cart by calling `CartBloc.addProductSink.add(Product item, int qty)`. Following that model, which I am, this is the steps we need to take to implement this bloc:

* Implement CartBloc.addProductSink. A sink is a type of StreamController. Calling add on a sink well cause a sink to stream whatever data is "added" to it.
* Listen to the stream from the addProductSink
* When the stream is notified, call the service that adds an item to the user cart in the database.

That’s the whole input process. Remember, inputs and outputs are separate. Perhaps the service call will update an output, but the input doesn’t concern itself with that.

Implementing the input is done in lib/blocs/cart_bloc.dart in three steps:

First: Define a StreamController

```dart
// ecommerce/lib/blocs/cart_bloc.dart
class CartBloc {
//...
StreamController<AddToCartEvent> addProductSink = new StreamController<AddToCartEvent>();
```

Second: Listen to the Stream
```dart
CartBloc(this._service) {
    addProductSink.stream.listen((_handleAddItemsToCart));
```

Third: Write _handleAddItemsToCart
```dart
void _handleAddItemsToCart(AddToCartEvent e) {
  _service.addToCart(e.product, e.qty);
}
```

Finally, to actually add a product to that stream, look in `lib/widget/catalog.dart` at the method called `_addToCart`.

```dart
// ecommerce/lib/widget/catalog.dart -- line ~43
void _addToCart(Product product, int quantity, CartBloc _bloc) {
  _bloc.addProductSink.add(AddToCartEvent(product, quantity)); #1
}
```

That’s the whole process. In general, thats' how all streams will work in Dart (with some variations that we’ll cover in the Async Flutter chapter).

I want to stop here and say that observables and streams are hard to wrap your head around if you aren’t used to using them. If you don’t understand the first time, don’t get upset. Go take a nap and eat some pizza, then come back and try again.

**Remove an item from the cart**
The other side of blocs is the outputs. Outputs are streams, so the set up is similar. One of the outputs in the Farmers Market app is the cart item quantity. I’ll use that as an example.

Output events are generally kicked off from an external source, like an API. The output we’re working on depends on an in app event (a user adding or removing an item from their cart), but that data is coming from our "back-end". (I made a mock back-end that imitates Firebase’s Firestore. Firestore is real-time, and reactive, so you can tell it to notify you anytime a value you’re interested in updates. In the Farmers Market app, I’ve asked the mock-firestore to notify me whenever the number of items in the cart changes. The output event is a result of that notification, not directly kicked off in the client-side code.

With that explanation out the way, I’ll show you how to implement an output. All of this code is in the `lib/bloc/cart_bloc.dart` file.

First: instantiate the objects you need.
```dart
// e_commerce/lib/bloc/cart_bloc.dart -- line 26
StreamController _cartItemCountStreamController = new BehaviorSubject<int>(seedValue: 0);
Stream<int> get cartItemCount => _cartItemCountStreamController.stream;
```

Second: Add data to the stream controller from an external source. I set this up in the constructor of the CartBloc class.
```dart
// e_commerce/lib/bloc/cart_bloc.dart -- line 30
CartBloc(this._service) {
//...
    _service
        .streamCartCount()
        .listen((int count) => _cartItemCountStreamController.add(count));
```
`_service.streamCartCount` is a method on an external service. It’s also implemented with streams, but it has nothing to do with the bloc, so don’t get confused by it. The Flutter firestore implementation uses streams because it’s real-time. It could just as easily be be a method that’s called whenever you get a success response from hitting a REST API.

**Complete the blocs**
That’s technically a small part of the bloc functionality in this app, but all the rest of the functionality would be repeating these steps. Setting up blocs could be simplified to three aspects: the architecture, implementing inputs and implementing outputs.

With that in mind, I encourage you to try and understand the other bloc functionality in this app for extra practice. This is the functionality that wasn’t covered in this chapter (but using the same concepts).

* Cart Bloc
    * Remove from cart input
    * Cart items output
* Catalog Bloc
    * Add new product input
    * Update existing product input
    * All products output
    * Products by category output

The catalog bloc is interesting at this point. The products by category output is tricky because it’s actually a List of Streams. If this is your first time using streams, it may be tough, but I’ll cover it in detail in the async chapter (the next one).

#### 8.4  Summary
* Stateful widget lifecycle gives fine grain control over rebuilding widgets in Flutter, using the methods:
    * initState
    * didChangeDependencies
    * build
    * widgetDidUpdate
    * setState
    * dispose

* State objects are long lived, and can even be reused.
* Using only stateful widgets, you can implement a management pattern called "lifting state up".
* InheritedWidgets are special widgets optimized to pass data down the tree.
* Access InheritedWidgets anywhere in it’s subtree with an of method, which calls `inheritFromWidgetOfExactType`.
* Combining an InheritedWidget and a StatefulWidget gives you a cleaner way to lift state up.
* The bloc pattern a state management pattern that encourages a simple api and reusable business logic components.
* Blocs inputs and outputs should only be sinks and streams.
* Streams, also know as observables, are first class citizens in Dart, used for reactive, asynchronous programming.
* Streams emit events to listeners, who are always waiting patiently for an update from streams.
