>In this chapter:
>* Setting up named routes
>* Building routes on the fly
>* Using the Navigator
>* Custom page transition animations

When I was planning out this chapter, I was trying to answer the question "Why?", or "Who cares?". This is one of the standard questions the authors at Manning think about before writing a chapter. And, well, this time it was pretty easy to answer: everyone who doesn’t want to make an app with a single page. Thus, a chapter on routing.

Routing can be a real pain (but it shouldn’t have to be!). In the web world, I think the folks behind React Router nailed the solution. It’s easy to use, and it’s flexible. It matches the reactive and composable UI style of React.

According to their docs, they’re in the business of "dynamic routing", rather than static. Historically, most routing was declarative and routes were configured before the app rendered. The creators of React Router explained it well in their docs: "When we say dynamic routing, we mean routing that takes place as your app is rendering. Not in a configuration or convention outside of a running app." (reactrouter.com)

I’m talking about React Router right now because the mental-model needed for routing in Flutter is the same. And, to be candid, I didn’t know how to approach this topic, so I looked to people who are much smarter than me. (Thanks, React Router team).

#### 7.1  Routing in Flutter

The advantage to dynamic routing is that it’s flexible. You can have a super complicated app without ever declaring a route, because you can create new pages on the fly. The Flutter Navigator also gives us the option of declaring routes and pages, if you want to take the static approach. And you can (and probably will) mix and match static routes and "on-the-fly" routes.

NOTE:Routing in Flutter is never really static, but you can declare all your routes up front, so the mental model is the same.

In Flutter, pages are just widgets which we assign to routes. And routes are managed by the Navigator, which is (you guessed it) just a widget! A Navigator widget is an abstraction over a widget that lays its children out in a stack nature. Because they’re widgets, you can nest Navigators up and down your app willy-nilly. Routers in routers in routers.

Before we get into the how-tos, let’s take a look at the app that I’m using as reference for the next couple of chapters.

7.1.1  The Farmers Market app
I live in Portland, Oregon, where people love Farmers Markets. I mean deeply love them. In an unnatural way. So, I thought I’d get rich by breaking into that market. This is the app I made for people to buy veggies and other treats from farmers:

![](https://dpzbhybb2pdcj.cloudfront.net/windmill/v-7/Figures/app_preview.png)

The structure of the app isn’t too complicated. It’s basically four pages. Our job in this chapter is wire up that menu with some routes, and create a few on-the-fly routes. Then, we’re going to make it fancy with some page transition animations.

The interesting thing about the routes in this app, in my very biased opinion, is that all the pages share the same structural elements. The app bar, the cart icon, the menu, some of the functionality and the scaffold are all written once, and I pass in the configuration based on the route. This is possible for two reasons: The way we compose UI in Flutter, and the fact that the Navigator is a widget that doesn’t have to be top level.

#### 7.1.2  The app source code
In the git repository, these are the relavent files for the book:
```
lib
├── blocs
│   ├── app_bloc.dart
│   ├── cart_bloc.dart
│   ├── catalog_bloc.dart
│   └── user_bloc.dart
├── menu
│   └── app_menu_drawer.dart
├── page
│   ├── base
│   │   ├── page_background_image.dart
│   │   ├── page_base.dart
│   │   └── page_container.dart
│   ├── cart_page.dart
│   ├── catalog_page.dart
│   ├── product_detail_page.dart
│   └── user_settings_page.dart
├── utils
│   ├── material_route_transition.dart
│   └── styles.dart
├── widget
│   ├── add_to_cart_bottom_sheet.dart
│   ├── appbar_cart_icon.dart
│   ├── catalog.dart
│   └── product_detail_card.dart
├── app.dart
└── main.dart
```

#### 7.2  Declarative Routing and Named Routes
If you’ve built web apps or mobile apps on nearly any other platform, you’ve likely dealt with declarative routing. On the other application platforms that I’ve used, (such as Ruby on Rails, Django, front-end libraries of the not-very-distant past), routes are defined in their own "routes" file, and what you declare is what you get. In AngularDart, your routes page may look like this:

```dart
Listing 7.2. AngularDart Router route definitions

static final routes = [
    new RouteDefinition(
        routePath: new RoutePath(path: '/user/1');
        component: main.AppMainComponentNgFactory),
    new RouteDefinition(
        routePath:  new RoutePath(path: '/404');
        component: page_not_found.PageNotFoundComponentNgFactory)
    //... etc
  ];
```

Understanding Angular code isn’t important, it’s an example of up-front route declarations written in Dart. The point is that you tell your app explicitly what routes you want to exist, and which views they should route to. Each `RouteDefinition` has a path and a component (which is probably a page). This is generally done at the top level of an app. Pretty standard stuff here.

Flutter supports this. While the routes and pages are still built while the app is running, the mental-model you can approach this with is that they’re static.

Mobile apps often support tens of pages, and it’s perhaps easier to reason about if you define them once and then reference them by name, rather than creating unnamed routes all over the app.

Flutter routes follow the path convention of all programing, such as "/users/1/inbox" or "/login". And as you’d expect, the route of the home page of your app is "/".

#### 7.2.1  Declaring routes
There are two parts to using named routes. The first is defining the routes. In the e-commerce app you’re building in this chapter, the named routes are set up in the `lib/main.dart` file. If you navigate to that file, you’ll see a `MaterialApp` widget with the routes established, and in `utils/e_commerce_routes.dart` file you’ll see the static varibales with the actual route names. (This is just so I can safetly use routes without fearing typos in the strings.)

```dart
Listing 7.3. Define Routes in the MaterialApp widget

// e_commerce/lib/main.dart -- line ~ 51
// ...
return MaterialApp(
  debugShowCheckedModeBanner: false,
  theme: _theme,
  routes: {
    ECommerceRoutes.catalogPage: (context) =>
        PageContainer(pageType: PageType.Catalog),
    ECommerceRoutes.cartPage: (context) =>
        PageContainer(pageType: PageType.Cart),
    ECommerceRoutes.userSettingsPage: (context) =>
        PageContainer(pageType: PageType.Settings),
    ECommerceRoutes.addProductFormPage: (context) =>
        PageContainer(pageType: PageType.AddProductForm),
  },
  navigatorObservers: [routeObserver],
);

// e_commerce/lib/utis/e_commerce_routes.dart
class ECommerceRoutes {
  static final catalogPage = '/';
  static final cartPage = '/cart';
  static final userSettingsPage = '/settings';
  static final cartItemDetailPage = '/itemDetail';
  static final addProductFormPage = '/addProduct';
}
```

**Navigating to Named Routes**
Navigating to named routes is as easy as using the `Navigator.pushNamed` method. The `pushNamed` method requires a `BuildContext` and a route name, so you can use it anywhere that you have access to your `BuildContext`.
```dart
var value = await Navigator.pushNamed(context, "/cart");
```
Pushing and popping routes is the bread and butter of routing in Flutter. Recall that the Navigator lays its children (the pages) out in a 'stack' nature. The stack operates on a "last in, first out" principal, as stacks do in computer science. If you’re looking at the home page of your app, and you navigate to a new page, you "push" that new page on top of the stack (and on top of the home page). If you pushed another route, we’ll call it page three, and wanted to get back to the home page, you’d have to "pop" twice.

**Figure 7.1. Flutter’s navigator is a stack-like structure**
![](https://dpzbhybb2pdcj.cloudfront.net/windmill/v-7/Figures/navigator_push_pop.png)

The `Navigator` class has a bunch of helpful methods to manage the stack. This is only a handful of the methods I find myself using:
* pop
* popUntil
* canPop
* push
* pushNamed
* popAndPushNamed
* replace
* pushAndRemoveUntil

One important note about pushing named routes is that they return a `Future`. If you’re not familiar with the `await` keyword above, it’s used to mark expressions that return an asynchronous value. We’re not going to get into async Dart quite yet. But, the quick version is that when you call `Navigator.pushNamed`, it immediately returns a `Future` object, which says "Hey, I don’t have a value you for you yet, but I will as soon as this process finishes". In the specific context of routing, this means "As soon as they navigate back to here from the page they’re on now, I’ll give you whatever value they pass back from that page." Later in this chapter, we’ll explore passing values between routes more in-depth.

In the e-commerce project repository, we can find an example of using `Navigator.pushNamed` in the `AppBarCartIcon` widget, found in `lib/widget/appbar_cart_icon.dart` file.

![](https://dpzbhybb2pdcj.cloudfront.net/windmill/v-7/Figures/app_bar_icon_annotation.png)

This icon has a little bubble on it that keeps track of how many items are in the users cart. And The widget is actually an `IconButton` widget, and its wired up to navigate to the cart page when it’s tapped and the `onPressed` callback is called:
```dart
onPressed: () {
    return Navigator.of(context).pushNamed("/cartPage");
},
```

That’s all there is to it.

NOTE:You may notice that `Navigator.of(context).pushNamed(String routeName)` function signature isn’t the same as the previously mentioned `Navigator.pushNamed(BuildContext context, String routeName)` signature. These are interchangeable.

#### 7.2.2  MaterialDrawer widget and a the full menu
If you’ve seen a Material Design app, you’re probably familiar with this type of app drawer:

![](https://dpzbhybb2pdcj.cloudfront.net/windmill/v-7/Figures/menu_drawer.png)

Since this chapter is about routing, I think now’s a good time to explore how to build that menu. First, let’s think about what we actually want it to do:

1. The menu should display when a user taps a menu button.
2. There should be a menu item for each page, which navigates to a route on tap.
3. There should be an "About" menu item, which shows a modal with app information.
4. There should be a menu header, which displays user information. When you tap on the user settings, it should route to the user settings page.
5. The menu should highlight which route is currently active.
6. The menu should close when a menu item is selected, or when a user taps the menu overlay to the right of the menu.
7. When the menu is open or closed, it should animate nicely in and out.

This particular menu drawer is a combination of only five required widgets, all of which are built into Flutter.

* Drawer
* ListView
* UserAccountsDrawerHeader
* ListTile
* AboutListTile
  
The `Drawer` is the widget that houses this menu. It takes a single widget in it’s `child` argument. You can pass it whatever you want (because everything is a widget). A `Drawer` will most likely be passed to a `Scaffold` in it’s `drawer` argument.

If you also have an `AppBar` in a scaffold with a drawer, then Flutter automatically sets the right side icon on the app bar to a menu button, which opens the menu on tap. The menu will animate out nicely, and close when you swipe it left or tap the overlay to the right.

NOTE:You can override the automatic menu button by setting `Scaffold.automaticallyImplyLeading` to `false`.

**Menu items: ListView and ListItems**
Next, the `ListView`. A list view is layout widget that arranges widgets in a scrollable container. It arranges it’s children vertically by default, but it’s quite customizable. We’ll talk about scrolling and scrollable widgets in depth in the next chapter. For now though, you can use it just like you’d use a `Column`. You only need to pass it some children, which are all widgets.

A `ListItem` is a widget that has two special characteristics: They’re fixed height. Making them ideal for menus. Also, the `ListItem` widget is opinionated. Unlike other generalized widgets, which expect a argument called `child`, the `Item` has properties like "title", "subtitle", and "leading". It also comes equipped with an `onTap` property.

![](https://dpzbhybb2pdcj.cloudfront.net/windmill/v-7/Figures/material_list_tile.png)

The `UserAccountsDrawerHeader` is a Material widget thats used to display crucial information, and it listens to multiple events. Imagine a Google app like GMail, which let’s you switch between user accounts with the tap of the button. This is built in. We don’t need this for this app. There’s no lesson here. It’s just a great example of how much Flutter gives you for free.

Finally, the `AboutListTile`. This widget can be passed into the `ListView.children` list, and configured in just a few lines of code:

```dart
AboutListTile(
    icon: Icon(Icons.info),
    applicationName: "Farmer's Market",
    aboutBoxChildren: <Widget>[
    Text("Thanks for reading Flutter in Action!"),
]),
```

With that code right there, you get a fully functional menu button that displays a modal on tap, complete with a button to close the modal. It looks like this:

![](https://dpzbhybb2pdcj.cloudfront.net/windmill/v-7/Figures/about_modal.png)

With all that in mind, I think this brings our requirements list to this:

1. The menu should display when a user taps a menu button.
2. There should be a menu item for each page, which navigates to a route on tap.
3. There should be an "About" menu item, which shows a modal with app information.
4. There should be a menu header, which displays user information. When you tap on the user settings, it should route to the user settings page.
5. The menu should highlight which route is currently active.
6. The menu should close when a menu item is selected, or when a user taps the menu overlay to the right of the menu.
7. When the menu is open or closed, it should animate nicely in and out.

This is slightly over-exaggerated. Of course you have to actually write those five lines of code for the `AboutListTile`. But that’s a heck of a lot easier than writing the logic to show a modal, and write the layout for the modal itself.

**Implementing the Menu Drawer**
With all this widget knowledge in mind, most of the work is done for you. The bulk of implementing this menu is in the routing. Which works well, because this is a chapter about routing. Most of this work is done in the `lib/menu/app_menu_drawer.dart` file.

For starters, menu items, when tapped, should route to a new page. In the the `build` method, there is a `ListTile` for each of these menu items. One `ListTile` looks like this:

```dart
Listing 7.4. Menu drawer item in a ListTile widget

// e_commerce/lib/menu/app_menu_drawer.dart -- line ~63
ListTile(
    leading: Icon(Icons.apps),
    title: Text("Catalog"),
    selected: _activeRoute == ECommerceRoutes.catalogPage,
    onTap: () => _navigate(context, ECommerceRoutes.catalogPage),
),

void _navigate(BuildContext context, String route) {
    Navigator.popAndPushNamed(context, route);
}
```

You can see another example of adding a page to the stack in the `UserAccountsDrawerHeader` widget in this same build method.

```dart
UserAccountsDrawerHeader(
  currentAccountPicture: CircleAvatar(
    backgroundImage:
        AssetImage("assets/images/apple-in-hand.jpg"),
  ),
  accountEmail: Text(s.data.contact),
  accountName: Text(s.data.name),
  onDetailsPressed: () {
    Navigator.pushReplacementNamed(
        context, ECommerceRoutes.userSettingsPage);
  },
),
```

#### 7.2.3  Highlight active route with RouteAware
Another interesting aspect of the Flutter router is "Observers". You won’t get far into Dart programming without using observers and streams and emitting events. There’s an entire chapter devoted to that later in this book, but for now I’m going to focus specifically on a `NavigatorObserver`. A navigator observer is an object that tells any widget who’s listening "hey, the navigator is performing some event, if you’re interested." That’s really it’s only job — but it’s an important one.

![](https://dpzbhybb2pdcj.cloudfront.net/windmill/v-7/Figures/nav_observer_flow.png)

A subclass of `NavigatorObserver`, and the one we’re interested in here is `RouteObserver`. This observer specifically notifies all it’s listeners if a route of specific type changes. For instance, `PageRoute`.

This `RouteObserver` is how we’re going to keep track of which page is active, and use it highlight the correct menu item in the menu.

For many cases, you’ll only need one `RouteObserver` per `Navigator`. Which means there will likely be only one in your app. In this app, the route observer is built in the `app` file.
```dart
final RouteObserver<Route> routeObserver = RouteObserver<Route>();
```
I created the observer out in the open, not protected by the safety of any class, because I need it to be visible through out the whole app. So, it’s in the "global" scope.

After this, you need to tell you `MaterialApp` about it. In the same file:
```dart
Listing 7.5. Pass route observers into the MaterialApp widget

// e_commerce/lib/app.dart -- line ~51
return MaterialApp(
    debugShowCheckedModeBanner: false,
    theme: _theme,
    home: PageContainer(pageType: PageType.Catalog,),
    routes: { ... }
    navigatorObservers: [routeObserver],
);
```
That’s all the set up, now you can listen to that observer on any `State` object. It has to be `stateful` because you’ll need to use some state lifecycle method. For our purposes, this is going to happen back in the `AppMenu` widget you’ve been working with in this section.

First, the state object needs to be extended with the mixin `RouteAware`.
```dart
class AppMenuState extends State<AppMenu> with RouteAware { ... }
```
This `RouteAware` mixin is an abstract class that provides an interface to interact with a route observer. Now, your state object has access to methods `didPop`, `didPush` and few others.

In order to update the menu with the correct active route, we basically need to be notified when ever theres a new route on the stack. There are two steps to that: First, listen to changes from the route observer. Second, listen to the observer to be notified whenever the route changes.
```dart
Listing 7.6. Listen to the route observer

// e_commerce/lib/menu/app_menu_drawer.dart -- line ~19
class AppMenuState extends State<AppMenu> with RouteAware {
  String _activeRoute;

    @override
    void didChangeDependencies() {
        super.didChangeDependencies();
        routeObserver.subscribe(this, ModalRoute.of(context));
    }
// ...
```

Now that this widget is aware of route changes, it needs to update it’s active route variable when any navigator activity happens. This is done in the `didPush` method that it inherited from `RouteAware`.
```dart
// e_commerce/lib/menu/app_menu_drawer.dart -- line ~30
@override
void didPush() {
_activeRoute = ModalRoute.of(context).settings.name;
}
```

#### 7.3  Routing on the Fly
"Routing on the fly" is the idea that you can route to a page that doesn’t exist until it’s generated in response to an event. For example, you may want to navigate to a new page when the user taps a list item. You don’t have to establish this route ahead of time, because routes are just widgets. Here’s some example code:

```dart
void _showListItemDetailPage() {
    Navigator.push(
      context,
      MaterialPageRoute(
        builder: (context) => SettingsPage(
              settings: settings,
            ),
        fullscreenDialog: true,
      ),
    );
  }
```

In Flutter, everything that seems like a new widget on the "route stack" is a route. Modals, "bottom sheets", snack bars, and dialogs are all routes, and are perfect candidates for routing on the fly.

#### 7.3.1  MaterialRouteBuilder
The first important place that we route on the fly in this app is whenever we tap on a product in the `catalog` page, and navigate to a `product detail` page.

>NOTE
On this page there are a lot widgets and concepts not yet covered in this book, like `StreamBuilder` and `Slivers` and all kinds of goodies. We will cover those later in the book.

Around line 88 in `lib/widget/catalog.dart`, I build the `ProductDetailCard`, which is listening for a tap to navigate to another page.

![](https://dpzbhybb2pdcj.cloudfront.net/windmill/v-7/Figures/detail_cards.png)

The code for that looks like this:
```dart
// e_commerce/lib/widget/catalog.dart -- line ~88
return ProductDetailCard(
    key: ValueKey(_product.imageTitle.toString()),
    onTap: () => _toProductDetailPage(_product),
    onLongPress: () => _showQuickAddToCart(_product),
    product: _product,
);
```
```dart
Listing 7.7. Navigate to a new route on the fly

// e_commerce/lib/widget/catalog.dart -- line ~37
Future _toProductDetailPage(Product product) async {
    Navigator.push(context,
        MaterialPageRoute(
            builder: (context) => ProductDetailPageContainer(
              product: product,
            )));
}
```

That’s it for navigating to new pages which aren’t established. Routes which don’t cover the whole screen are similar.

#### 7.3.2  showSnackBar, showBottomSheet and the like

Flutter has widgets and logic to make super easy use of routes that aren’t pages, like modals and snackbars. These are technically routes, under the hood, they just don’t render like a whole page would. They’re still widgets that get added on the stack of the navigator, rather than being attached to a page.

In this app, we make use of the "bottom sheet", which is common in iOS apps, and a snackbar. These are similar in the fact that they appear from the bottom of the screen, and only cover a portion on the screen.

![](https://dpzbhybb2pdcj.cloudfront.net/windmill/v-7/Figures/popup_routes.png)

They’re different, though, because a bottom sheet is a `ModalRoute`. Meaning that when it’s in view, you cannot interact with the page beneath it. A snack bar doesn’t obstruct the app, though, you can still interact while it’s in view.

The bottom sheet is implemented in the same `Catalog` widget, and kicked off by pressing and holding on a `ProductDetailCard` via the `ProductDetailCard.onLongPress` method.
```dart
// e_commerce/lib/widget/catalog.dart -- line ~88
return ProductDetailCard(
    key: ValueKey(_product.imageTitle.toString()),
    onTap: () => _toProductDetailPage(_product),
    onLongPress: () => _showQuickAddToCart(_product),
    product: _product,
);
```

This method is in the business of showing the `BottomSheet`, waiting the user to interact with it, and listening to the data that the bottom sheet passes back.

![](https://dpzbhybb2pdcj.cloudfront.net/windmill/v-7/Figures/modal_future_flow.png)

The important code in that method is this:
```dart
// e_commerce/lib/widget/catalog.dart -- line ~48
void _showQuickAddToCart(BuildContext context, Product product) async {
  var _cartBloc = AppState.of(context).blocProvider.cartBloc;

  int qty = await showModalBottomSheet<int>(
    context: context,
    builder: (BuildContext context) {
        return AddToCartBottomSheet(
        key: Key(product.id),
        );
    });

    _addToCart(product, qty, _cartBloc);
}
```

To complete this implementation, we need to look at the `AddToCartBottomSheet` code. Within that widget, there’s a `RaisedButton`, which corresponds to this button on the screen:

![](https://dpzbhybb2pdcj.cloudfront.net/windmill/v-7/Figures/bottom_sheet_modal.png)

```dart
// e_commerce/lib/widget/add_to_cart_bottom_sheet.dart -- line ~88
RaisedButton(
    color: AppColors.primary[500],
    textColor: Colors.white,
    child: Text(
        "Add To Cart".toUpperCase(),
    ),
    onPressed: () => Navigator.of(context).pop(_quantity),
)
```

This is a pretty run of the mill button. The part we’re interested in is its `onPressed` callback. When it’s pressed, it pop’s the top route off the navigator (which is the bottom sheet itself), and then passes that variable (`_quantity`), back to code that added this route onto the stack.

Other popup style widgets work just as easily. For example, to show a snackback, you can call `showSnackbar`, which lives on a scaffold: `Scaffold.of(context).showSnackBar(Widget content)`;.

#### 7.4  Routing Animations
The final piece of the routing puzzle is making it pretty, which is my favorite part about writing apps. Believe it or not, though, there’s almost no work involved in adding custom route transitions. The real work, actually, is in writing the animation, if you want to do it super fancy. But, as you saw in the previous app, Flutter gives you quite a few animations out of the box.

Before we dive in, consider how page transitions work in Flutter by default. The following facts are important:

* Pages are just widgets, so they can be animated like any other widget.
* Pages have default transitions, which differ by platform. (iOS style or Material style)
* All transitions involve two pages. One of which is coming into view, and one that is leaving view.

With that in mind, lets dive in. Transitions are handled by the `PageRoute`, or in our case `MaterialPageRoute`, which extends `PageRoute`, which extends `ModalRoute`, which extends `TransitionRoute`. Somewhere in this mess, there’s method called `buildTransitions`, which, among other things, takes two animations as arguments. One is for itself, as it exits, and the second to coordinate with the route that’s replacing it. `MaterialPageRoute` already implements transitions, which means you can override `MaterialPageRoute.buildTransitions`.

All this method has to do is return a widget. And by overriding it, we don’t have to worry about writing the `AnimationController` or `Tween` — thats all taken care of in the super class. Which means, we can simply return a page that’s wrapped in an animated widget, and it will animate accordingly. Take a look at the code to make that less abstract. This can be found in `lib/util/material_route_transition.dart`.

```dart
Listing 7.8. Writing a custom page transition.

// e_commerce/lib/util/material_route_transition.dart
class FadeInSlideOutRoute<T> extends MaterialPageRoute<T> {
  FadeInSlideOutRoute({WidgetBuilder builder, RouteSettings settings})
      : super(builder: builder, settings: settings);

  @override
  Widget buildTransitions(BuildContext context, Animation<double> animation,
      Animation<double> secondaryAnimation, Widget child) {

    // ...

    return FadeTransition(opacity: animation, child: child);
  }
}
```

To use this, all you need to do is go to `Catalog._toProductDetailPage` and replace `MaterialPageRoute` with `FadeInSlideOutRoute`. Now when you tap a product detail card, the routes will fade. I guess it’s worth saying that you could get as fancy and wild with this as you wanted. You’re limited only by the animation you write.

#### 7.5  Summary

* Flutter uses "dynamic" routing, which makes routing much more flexible and fluid.
* Flutter’s navigator allows you to create routes "on the fly" in your code, just as some user interaction takes place or the app receives new data.
* Flutter supports (practically) static routing to, using named routes. (Although these routes are still technically created as the app is running).
* Define your named routes in your `MaterialApp` widget, or whichever top-level "App" widget you’re using.
* The Navigator manages all the routes in a stack manner.
* Navigating to routes is done by calling variations of `Navigator.push` and `Navigator.pop`.
* "Push" calls return a `Future`, which await a value from that’ll be passed back by the new route.
* Creating a full Material style menu drawer in Flutter involves several incredibly generous widgets: `Drawer`, `ListView`, `ListTile`, `AboutListTile`, and `DrawerHeader`.
* You can anticipate changes in routing by setting up a `RouteObserver` and subscribing to it on any widget’s state object.
* Several UI elements are managed with the Navigator, and are technically routes, though they aren’t pages, such as snackbars, bottom sheets, drawers and menus.
* You can listen for user interactions using the `GestureDetector` widget.
* Implementing custom page transitions is done by extending `Route` classes.

