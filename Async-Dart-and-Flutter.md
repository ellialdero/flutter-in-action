> **In this chapter:**
>* Futures in Dart
>* Streams and Sinks Dart
>* AsyncBuilder in Flutter
>* Slivers and Scrolling
>* Scroll Physics

This chapter could contain the most difficult concepts to wrap your head around. Unless you’re familiar with async UI programming. If you’re okay with the the following code snippits, then you should skip to section 9.2
```dart
// Futures
var val = new Future<bool>();

// Streams and Sinks
StreamController<bool> _controller = new StreamController<bool>();
Stream get onEvent => _controller.stream;

void handleEvent() {
    onEvent.listen((bool val) => print(val));
}

// async / await
void getRequest() async {
    var response = await http.get(url);
    return response;
}
```
If you aren’t comfortable with that, then strap in. Async programming is mandatory in modern UI, so I’d expect that you’ve used it at some point before now. But, Dart programming makes heavy use of some specific async patterns, known in most languages as "observables".

You won’t get very far into any Dart code before you have to start dealing with streams, which is what Dart calls observables. And in fact, I’d say they’re necessary to effective Flutter programming.

Futures, completers, async for, streams, sinks and the `listen` keyword are all part of the same "part" of the Dart language. These features are all similar or related. Some (if not all) of these classes and features are built ontop of eachother, with `Future` being the basic building bloc.

#### 9.1  Async Dart
In this section, I’ll start with `Future`, breifly, and then focus on streams in-depth. I’ll also explain some convenience features, like `async/await` and `async for` towards the end. Starting in the next section (9.2), I’ll shift back to Flutter and show how all this ties in with the Farmers Market app.

#### 9.1.1  Future Recap
`Future` is the foundational class of all async programming in Dart. In Chapter 4 there was a brief section on the `Future` class, but I’d like to give some concrete examples in addition to the explanation.

Imagine you’re visiting a planet inhabited by robots, and you decide to pick up a galaxy-famous robot burger. You order your burger at the counter and get a receipt. Futures are a lot like that receipt. You, the burger order-er, tell the robot-employee that you’d like to buy a burger. The employee gives you a receipt that guarantees you’ll get a buger as soon as one is ready.

So you, the caller, go wait until the robot calls your number, and then delivers on the guarantee of a burger. The receipt is the Future. It’s a guarantee that a value will exist, but it isn’t quite ready. The burger is the value, not the future.

In code, a future is a placeholder for a value that will exist. The most common scenario for using futures is when you’re getting values over the network. In UI specifically, you could pass a `Future<List<String>` into a list of items to display to the user. But, you may need to go fetch that list from an outside API over http. So you say "hey, UI, show a loading sign until this Future completes, but know that a list of strings for you to display is coming eventually."

Futures are *thenable* (that is then-able), so when you call a future, you can always say `myFutureMethod().then((returnValue) ⇒ … do some code … );`

`Future.then` takes a callback, which will be executued when the future value resolves. In the burger restaurant, the callback is what you decide to do with the burger when you get it. The value passed into the callback is whatever the return value of the original future is.
```dart
Listing 9.1. Basic Future Use

void main() {
   print("A");
   futrePrint(Duration(milliseconds: 1), "B").then((status) => print(status));
   print("C");
   futrePrint(Duration(milliseconds: 2), "D").then((status) => print(status));
   print("E");
 }

 Future futrePrint(Duration dur, String msg) {
   return Future.delayed(dur).then((onValue) => msg);
 }

// prints
A
C
E
B
D
```

9.1.2  async/await
The keywords `async` and `await` are the easiest way to wrap your head around async programming (in my opinion). In a nutshell, you can mark any function as `async`, and then tell that function to `await` async processes to finish before moving on. It’s basically like saying "Hey function, if you see the word `await` anywhere, just pause right there and wait for it to finish before moving to the next line.

Look at what happens when we re-use the example above but add a couple `await` keywords in the mix:
```dart
Listing 9.2. Async and await example

bvoid main() async {
   print("A");
   await futrePrint(Duration(milliseconds: 1), "B").then((status) => print(status));
   print("C");
   futrePrint(Duration(milliseconds: 2), "D").then((status) => print(status));
   print("E");
 }

 Future futrePrint(Duration dur, String msg) {
   return Future.delayed(dur).then((onValue) => msg);
 }

// prints
A
C
E
B
D
```

**Futures, asnyc and catching errors**
You can can catch errors with async code in two ways, so you can handle errors or failed network calls gracefully. With plain ol' future types, there’s a method called `catchError`.

```dart
Listing 9.3. Catching errors on furutres

Future futrePrint(Duration dur, String msg) {
   return Future.delayed(dur).then((onValue) => msg);
 }

 main() {
    futrePrint(Duration(milliseconds: 2), "D")
          .then((status) => print(status))
          .catchError((err) => print(err));
 }
```
You can also use `try / catch` blocks, especially useful with `async / await`.

```dart
void main() async {
    try {
      print("A");
      await futrePrint(Duration(milliseconds: 1), "B")
        .then((status) => print(status));
      print("C");
      await futrePrint(Duration(milliseconds: 2), "D")
        .then((status) => print(status)).catchError((err) => print);
      print("E");
    } catch(err) {
    print("Err!! -- $err");
  }
}
```

#### 9.2  Sinks and Streams (and StreamControllers)
Streams are a big part of Dart programming. One of the biggest concerns in building UI’s, it seems, is how to handle data asynchronously. And Streams are the way the Flutter team has decided to handle rendering data from the internet.

Streams are an asynchronous programming pattern, often called observables in other languages. From a high-level, streams provide a way for classes or objects to be notified when events happen.

> NOTE
> `Stream` aj llctuaya c ifccepsi casls, nhs gkfn xnx ieecp el uvr blvseoebar trptaen. Jn egrnale, houhtg, mea"r"ts aj pkr bwtv zbkp xr debesric vyr teccpon cz s eholw.

In real life, the observer pattern is the most relatable architecture pattern around. Throughout the day, how often are you given an update by email, from apps, or from real life conversations, and react to it? (That’s why this is called *reactive* programming.)

In code, you have one object that’s just passively waiting for to be notified by another class. Whenever it is updated, it takes appropriate action.

To expand on the robot burger future example, an observer might be the cook that’s actually making the burgers. That cooks job is completely reactive. She know’s how to make a burger when one is ordered, but she doesn’t actively seek out burger-eaters to make burgers for them. She just sits in the kitchen, waiting to be told what to cook.

The active part of this relationship is handled by the robot behind the register. This robot is given orders from customers, and then he turns around and passes (or emits) that information to the cook.

There are three pieces of the observer pattern:

* Sinks
* Streams
* subscribers
    * Also called: listeners, observers

* *Sinks* are the first stop for events in the observer pattern. A sink is the piece of the puzzle that you feed data into. It’s kind of the 'central source' for the whole process. In dart, a `Sink` is an abstract class that more specific types of sinks implement. The most common of which is the `StreamController`.

* *Streams* are properties on the `Sink`. When the sink needs to notify listeners of new events, it does so via streams.
* subscribers are the external classes or objects that are waiting to be notified. This is done by "listening" to streams.

#### 9.2.1 Implementing streams
This is basically the boiler plate you’d need for any stream (using the burger example):
```dart
Listing 9.4. Stream example

class BurgerStand {
  StreamController _controller = new StreamController();
  Stream get onNewOrder => _controller.stream;
  Cook cook = new Cook();

  void deliverOrderToCook() {
    onNewOrder.listen((newOrder) {
      cook.prepareOrder(newOrder);
    });
  }

  void newOrder(String order) {
    _controller.add(order);
  }
}

class Cook {
  void prepareOrder(newOrder) {
    print("preparing $newOrder");
  }
}

main() {
  var burgerStand = new BurgerStand();
  burgerStand.deliverOrderToCook();

  burgerStand.newOrder("Burger");
  burgerStand.newOrder("Fries");
  burgerStand.newOrder("Fries, Animal Style");
  burgerStand.newOrder("Chicken nugs");
}
```

#### 9.2.2  Broadcast streams
In our above burger code, the cook is probably getting pretty flustered. He’s working the burger station, the fries station and the chicken nugget stand. That’s too many jobs. This burger place probably needs to a hire a second cook, and split the responsibilty.

In the code, that means there will be two cooks listening to the server, and reacting based on the information. But sinks, be default, can only be listened to once. Which is where the broadcast streams come in. `StreamController.broadcast()` is a constructor that returns a controller that can be listened to by multiple subscribers.

I updated the cheese burger example a bit to showcase this. The changes have been annotated.
```dart
Listing 9.5. Broadcast stream controller example

class Cook {
  void prepareOrder(newOrder) {
    print("preparing $newOrder");
  }
}

class Order {}
class Burger extends Order {}
class Fries extends Order {}


class BurgerStand {
  StreamController<Order> _controller = new StreamController.broadcast();
  Cook grillCook = new Cook();
  Cook fryCook = new Cook();


  Stream get onNewBurgerOrder =>
      _controller.stream.where((Order order) => (order is Burger));
  Stream get onNewFryOrder =>
      _controller.stream.where((Order order) => (order is Fries));

  void deliverOrderToCook() {
    onNewBurgerOrder.listen((newOrder) {
      grillCook.prepareOrder(newOrder);
    });

    onNewFryOrder.listen((newOrder) {
      fryCook.prepareOrder(newOrder);
    });
  }

  void newOrder(Order order) {
    _controller.add(order);
  }
}

main() {
  var burgerStand = new BurgerStand();
  burgerStand.deliverOrderToCook();

  burgerStand.newOrder(new Burger());
  burgerStand.newOrder(new Fries());
}
```
The important part of all this is that both of the getters are referring the same stream. So, when the two "listeners" are created in the `deliverOrderToCook` method, they’re being called on two different references to the stream (`onNewBurgerOrder` and `onNewFryOrder`), but they’re listening to the same stream. This wouldn’t be possible on a standard stream, only broadcast streams.

#### 9.2.3  Higher Order Streams
Since streams are so common in Dart and Flutter, it’s not long until you’ll want to get a stream of data, but perform an action on every new piece of data from emitted from the stream. A stream itself returns a stream is called a higher order stream. (Similarly, higher order functions are functions that return new functions. That’s where the name comes from.)

Consider the robot hamburger shop again. You’ve decided to stay on that planet, and you get yourself a job at that burger shop as a cook. Everything is ordered via meal numbers, like this:

+--------------- | 1: Burger, Drink | 2: Cheeseburger, Drink | 3: etc…

But they serve roughly 10,000 items. Oh and, they’re robots. They only understand binary.

#### 9.3  Using streams in blocs
For a less abstract example, consider our Farmer’s Market app. Each time a an item is added to a users cart, the cart icon should update with the new quantity of items. The logic to accomplish this is done in the `CatalogBloc` via streams.

When an item is added to the cart, your code will `add` the additional quantity to the stream controller. The controller will then send a message to every object that’s subscribed to the stream. Each time a listener is notified, it will call the callback function that you provide. For example, the cart icon may call a function that changes the quantity it displays, and then calls set state, triggering a re-render.

The functionality that involves the UI specifically is discussed in the previous chapter, so I’ll focus on the `bloc` code now.

There are three different stream setups in this bloc. The first streams all the products in the catalog. So when a new one is added, it will push out an updated list of products. It’s an *output* of the bloc. This stream has a pretty standard setup.
```dart
// e_commerce/lib/blocs/catalog_bloc.dart

class CatalogBloc {
  StreamController _productStreamController = new StreamController<List<Product>>();
  Stream<List<Product>> get allProducts => _productStreamController.stream;
  // ...
```
The second is another output, and it allows widgets to listen to different product categories. It’s how I’ve been able to split up the the catalog page by category:

This output is more involved. It basically uses the same pattern, but I’ve created a new stream controller for each `ProductCategory` programmatically.

```dart
// e_commerce/lib/blocs/catalog_bloc.dart
class CatalogBloc {
  List<StreamController<List<Product>>> _controllersByCategory = [];
  List<Stream<List<Product>>> productStreamsByCategory = [];

  CatalogBloc(this._service) {
    ProductCategory.values.forEach((ProductCategory category) {
      var _controller = new StreamController<List<Product>>();
      _service.streamProductCategory(category).listen((List<Product> data) {
        return _controller.add(data);
      });
      return _controllersByCategory.add(_controller);
    });
   _controllersByCategory.forEach((StreamController<List<Product>> controller) {
      productStreamsByCategory.add(controller.stream);
  });
}
```

If you aren’t familiar with streams before now, that may seem pretty complicated and confusing. That’s okay, you shouldn’t beat yourself up. This is probably the most complicated piece of code in the entire book. (Unless, of course, I happen write some complicated code for the last app, which I haven’t created yet.)

The code certainly is confusing, but it’s easier to think of a simpler version of that code. I could’ve written something like this:
```dart
StreamController _veggieStreamController = new StreamController<List<Product>>();
Stream<List<Product>> get veggieProducts => _productStreamController.stream;

StreamController _fruitStreamController = new StreamController<List<Product>>();
Stream<List<Product>> get fruitProducts => _productStreamController.stream;

StreamController _proteinStreamController = new StreamController<List<Product>>();
Stream<List<Product>> get proteanProducts => _productStreamController.stream;

// etc
```

This code, at the end of the day, is just creating a list of controllers and a list of streams for each controller. Then, in the UI in the `Catalog` widget, I’m iterating through the `CatalogBlog.productStreamsByCategory` and creating widgets based on each stream. I’ll cover that in the next few pages.

The more important point, for our purposes, is that the same pattern is repeated here. The concept of streams may be intuitive to you, or it may be insane. The implementation of streams may be easy to remember, but it may not be. Either way, though, the pattern of implementing streams doesn’t change. You need three pieces:

* A `StreamController` (or `Sink`)
* A `Stream`
* A subscriber
  
#### 9.3.1  Implementing a bloc input
The other use of streams in the `CatalogBloc` is the blocs inputs. Recall from the previous chapter that inputs in the blocs are always sinks. In this bloc, there are sinks to add new products to the catalog, or update existing ones. I’ll walk through creating a new product.

```dart
// lib/e_commerce/blocs/catalog_bloc.dart -- line !28
final _productInputController = new StreamController<ProductEvent>.broadcast();
Sink<ProductEvent> get addNewProduct => _productInputController.sink;

CatalogBloc(this._service) {
  _productInputController.stream
      .where((ProductEvent event) => event is AddProductEvent)
      .listen(_handleAddProduct);
}

// line ~ 67
_handleAddProduct(ProductEvent event) {
  var product = new Product(
    category: event.product.category,
    title: event.product.title,
    cost: event.product.cost,
    imageTitle: ImageTitle.SlicedOranges); // This is faked.
  _service.addNewProduct(product);
}
```

Again, this is a lot. But, all ~20 lines of code in this example accomplish a single goal. This code basically says "Hey app, I’m exposing this sink called `addNewProduct`. You can call `addNewProduct` with some data, and I’ll add that data to the database via the `_productInputController.stream` in the constructor.

In the UI, this is all handled in the `AddProductForm` widget. Theres a method thats called when the form is submitted that talks to the bloc. The relevant line is annotated:


```dart
// e_commerce/lib/page/add_product_form.dart -- line ~272
void _submitForm() {
  _formKey.currentState.save();
  _bloc.addNewProduct.add(AddProductEvent(_newProduct));
  _userBloc.addNewProductToUserProductsSink.add(NewUserProductEvent(_newProduct));
  Navigator.of(context).pop();
  }
```

With all that knowledge of streams, we can now talk about my favorite part of Flutter. `StreamBuilder` is a feature of Flutter that consumes streams of data and turns them into widget with not work on our part. It’s pretty incredible.

#### 9.4  Async Flutter: StreamBuilder
`StreamBuilder` is a class that generates widgets, but does so with async data. If you wanted to display a list of, for example, products in an ecommerce app, but you knew the list of available products was always changing, a stream builder is what you’d want to use. This class automatically listens to a stream you pass it, and will update and re-render the widget it produces whenever the stream gets new information. You could, of course, write a widget or widget builder that does this. But the beautiful thing about Flutter is that you don’t have to. They did it for you, so you could focus on the more interesting problems of writing an app.

The stream builder is used all over this app, but I’ll once again use the `Catalog` widget as the main example. That widget is comprised of a single `CustomScrollView` widget, which calls a method I wrote `_buildSlivers` and passes the return value of that method into the custom scroll view, creating the list items for the scroll view. The code that matters right now is in the `_buildSlivers` method.

> NOTE
> The word "slivers" does refer to a specific widget type. Don’t worry about it in this code example. I talk about slivers in depth at the end of this chapter.

```dart
// e_commerce/lib/widget/catalog.dart -- line ~66
List<Widget> _buildSlivers(BuildContext context) {
    // ...
  _bloc.productStreamsByCategory.forEach((Stream<List<Product>> dataStream) {
    slivers.add(StreamBuilder(
        stream: dataStream,
        builder: (context, AsyncSnapshot<List<Product>> snapshot) {
          return CustomSliverHeader(
            headerText:
                snapshot?.data?.first?.category.toString() ?? "header",
          );
        }));
    slivers.add(StreamBuilder(
        stream: dataStream,
        builder: (context, AsyncSnapshot<List<Product>> snapshot) {
          return SliverGrid(
            gridDelegate: new SliverGridDelegateWithFixedCrossAxisCount(
              crossAxisCount: 2,
              mainAxisSpacing: 8.0,
              crossAxisSpacing: 8.0,
            ),
            delegate: new SliverChildBuilderDelegate(
              (BuildContext context, int index) {
                var _product = snapshot.data[index];
                return ProductDetailCard(
                  key: ValueKey(_product.imageTitle.toString()),
                  onTap: () => _toProductDetailPage(_product),
                  onLongPress: () => _showQuickAddToCart(context, _product),
                  product: _product,
                );
              },
              childCount: snapshot.data?.length ?? 0,
            ),
          );
        }));
  });
  return slivers;
}
```

Again, I know that was alot. Unfortunately, we’ve come to a place in the book where features all rely on each other to explain, in a circular manner. That in mind, this section is about `StreamBuilder` widgets. The take away here is that Flutter has built in widgets that handle streams of data. If you have a list of widgets that could change anytime the app becomes aware of new data, Flutter has you covered.

This has finally brought us the final major chunk of Flutter functionality that you’ll probably need for every app ever: first class scroll behavior.

#### 9.5  Infinite and Custom Scrollable widgets
The catalog in the Farmers Market widget is special because, in theory, it could be infinitely long. It’s as long as the data in the "database" tells it to be. There are 20 products or so, so it’s that long, but it could 5,000 products long.

This is pretty standard functionality in modern UI. Instagram, Facebook, and Twitter all use infinite scrolling in their core features. Those services probably use different techniques to handle rendering a list with an unknown, potentially infinite number of list items, but it’s undoubtedly a common feature to have to handle. This is a book about creating UI, so I’m going to focus on the UI aspects of scrolling. In real life, it’s likely that you’d want to fetch items incrementally from a service, but this isn’t real life. This is about assuming you have an unknown number of list items, and probably more than can fit on a single screen.

#### 9.5.1  CustomScrollView and Slivers
The base class for infinite scrolling widgets in Flutter is the `CustomScrollView`. On top of this widget, the `ListView` is built. The list view is the most commonly used scrollable widget, as it just arranges widgets linearly, (like a column or row) but can scroll. A list view’s `children` are all passed to its children property, just like a row or column.

`CustomScrollView` is slightly different, it works directly with slivers. Slivers are just portions of a scrollable views. In fact, they’re basically just widgets. But, they lazily build when they scoll into view, so they’re performant. If you want to build a scrollable list that combines grids and columns, using slivers is a better option. You could achieve a complicated scroll view with the `ListView`, but it would likely be janky if the list was customized in any way.

In the `Catalog` widget, I used the `CustomScrollView` because I have a custom header for each category, and the headers "pin" to the top. In general, this non-standard behavior is smoother in custom scroll views.

> NOTE
> All that said, all scrollables work similarly. In fact, they aren’t much different than working with any other multi-child widget, like rows and columns. Plus, this is a good opportunity to talk about slivers, because people seem to be intimidated by them. I used to be afraid of slivers, until I realized they’re basically just "lower-level" widgets.

The official definition of slivers from the docs makes them seem "easy": "A sliver is a portion of a scrollable area. You can use slivers to achieve custom scrolling effects." I want to say more about them, but there really isn’t much more to say. They’re widgets that exist specifically in custom scroll views.

#### 9.5.2  CatalogWidget scroll view
In the app, I implement the custom scroll view in the Cat`alog widget, using two methods (`CatalogState.build` and `CatalogState._buildSlivers`) and a handful of widgets. First, look at the `build` method:

```dart
// e_commerce/lib/widget/catalog.dart -- line ~107
@override
Widget build(BuildContext context) {
  return CustomScrollView(slivers: _buildSlivers(context),
  physics: BouncingScrollPhysics(),
  );
}
```

This is a pretty "standard" implementation for this scroll view. But, it’s worth noting that this custom scrollview has quite a few configuration options that give you more control. You can make it horizonal rather than vertical. You can use scroll controllers to manage saving and detecting scroll position. The name of the game with this widget is "custom" (and it’s second name would be "performant").

The real action takes place in the `_buildSlivers` method.
```dart
// e_commerce/lib/widget/catalog.dart -- line ~107
List<Widget> _buildSlivers(BuildContext context) {
  _bloc.productStreamsByCategory.forEach((Stream<List<Product>> dataStream) {
    slivers.add(StreamBuilder(
        stream: dataStream,
        builder: (context, AsyncSnapshot<List<Product>> snapshot) {
          return CustomSliverHeader(
            headerText:
                snapshot?.data?.first?.category.toString() ?? "header",
          );
        }));
    slivers.add(StreamBuilder(
        stream: dataStream,
        builder: (context, AsyncSnapshot<List<Product>> snapshot) {
          return SliverGrid(
            gridDelegate: new SliverGridDelegateWithFixedCrossAxisCount(
              crossAxisCount: 2,
              mainAxisSpacing: 8.0,
              crossAxisSpacing: 8.0,
            ),
            delegate: new SliverChildBuilderDelegate(
              (BuildContext context, int index) {
                var _product = snapshot.data[index];
                return ProductDetailCard(
                  key: ValueKey(_product.imageTitle.toString()),
                  onTap: () => _toProductDetailPage(_product),
                  onLongPress: () => _showQuickAddToCart(context, _product),
                  product: _product,
                );
              },
              childCount: snapshot.data?.length ?? 0,
            ),
          );
        }));
  });
  return slivers;
}
```

This is probably the most complicated widget building I’ve done in this app, so let me walk through it from a high level, before I jump into the details of sliver grids and delegates.

First, recall the list of streams from the `CatalogBloc` that delivers a list of streams separated by category.

```dart
Listing 9.6. Catalog bloc output for products by category

// e_commerce/lib/blocs/catalog_bloc.dart
List<StreamController<List<Product>>> _controllersByCategory = [];
List<Stream<List<Product>>> productStreamsByCategory = [];
// ...
ProductCategory.values.forEach((ProductCategory category) {
   var _controller = new BehaviorSubject<List<Product>>();
   _service.streamProductCategory(category).listen((List<Product> data) {
     return _controller.add(data);
   });
   return _controllersByCategory.add(_controller);
 });
 _controllersByCategory
     .forEach((StreamController<List<Product>> controller) {
   productStreamsByCategory.add(controller.stream);
 });
```

We’re making use of this output in the `CatalogState._buildSlivers` method. We’re grabbing that list of streams via the `_bloc.productStreamsByCategory` output, and looping through it to create a heading and grid for each category.

In that loop, there are two steps, really. First, create the header with this loop and add it to the `slivers` list that will be returned by this method.

```dart
Listing 9.7. Adding a header sliver for the current category.

slivers.add(StreamBuilder(
  stream: dataStream,
  builder: (context, AsyncSnapshot<List<Product>> snapshot) {
    return CustomSliverHeader(
      headerText:
          snapshot?.data?.first?.category.toString() ?? "header",
    );
  }));
```

Then, add the "body" of this section, which is the product cards for that same category.

```dart
  slivers.add(StreamBuilder(
  stream: dataStream,
  builder: (context, AsyncSnapshot<List<Product>> snapshot) {
    return SliverGrid(
      gridDelegate: new SliverGridDelegateWithFixedCrossAxisCount(
        //...
      ),
      delegate: new SliverChildBuilderDelegate(
        (BuildContext context, int index) {
          var _product = snapshot.data[index];
          return ProductDetailCard(
            key: ValueKey(_product.imageTitle.toString()),
            // ...
            product: _product,
          );
        },
        // ...
      ),
    );
  }));
```

Here, the stream builder is taking that `dataStream` (a single category stream), and turning it into a `snapshot` in the builder function. The delegate can then grab the specific product by indexing into the `snapshot.data`.

The rest of this method is about new Flutter specific terms and widgets, like SliverGrid and delegates.

The `SliverGrid` widget places it’s widgets in a two-dimensional arrangement. You tell the sliver grid how many columns there are, and it’ll lay them out, in their order from left-to-right first, and when that row is full it’ll begin on the left side of the next row.

The `SliverGrid` itself is a pretty simple widget, accepting only two properties: `SliverGrid.gridDeletate` and `SliverGrid.delegate`.

9.5.3  Delegates
A `delegate` is a class that provides children for slivers. Some delegates, as you’ll see in a bit, are wrappers around builder functions, while some provide layout information.

They’re specifically for slivers, which usually construct their children lazily. At any given time, the a delegate has only created widgets that are visible through the viewport.

This purpose of this is performance. You don’t want Flutter to have to render 500 list items everytime a user scrolls, and slivers handle that problem for you. Not only do they lazily build the children, but they also efficiently destroy the elements and states when they’re scrolled out of view, and replace it with a new sliver in the same position.

Delegates all have a bit in common, so I’ll cover two that’ll give a good idea of how they work.

`SliverChildBuilderDelegate (The basic builder delegate)`
The `SliverChildBuilderDelegate` is a class that wraps a builder functions, and exposes some semantic scrolling behavior. What we care about is the builder. This builder function looks exactly like builders we’ve seen else where. In fact, this class basically is just a builder function that lazily builds its children for the sliver.

```dart
Listing 9.8. SliverChildBuilderDelegate usage

// e_commerce/lib/widget/catalog.dart -- line ~88
delegate: new SliverChildBuilderDelegate(
  (BuildContext context, int index) {
    var _product = snapshot.data[index];
    return ProductDetailCard(
      key: ValueKey(_product.imageTitle.toString()),
      onTap: () => _toProductDetailPage(_product),
      onLongPress: () => _showQuickAddToCart(context, _product),
      product: _product,
    );
  },
  childCount: snapshot.data?.length ?? 0,
),
```
That’s all that you need to create children for slivers. Under the hood, slivers might seem complicated, but just think of them as widgets and they’re nothing new.

**SliverGridDelegateWithFixedCrossAxisCount (A delegate with long name and therefor intimidating but actually simple)**
As I mentioned, there are also delegates that define layout and structure. This widget, with a very long name, `SliverGridDelegateWithFixedCrossAxisCount` is one of those. (There’s also `SliverGridDelegateWithMaxCrossAxisExtent`.)

This delegate is basically responsible for defining the number of columns in the grid in the catalog.

```dart
// e_commerce/lib/widget/catalog.dart -- line ~83
 return SliverGrid(
   gridDelegate: new SliverGridDelegateWithFixedCrossAxisCount(
     crossAxisCount: 2,
     mainAxisSpacing: 8.0,
     crossAxisSpacing: 8.0,
   ),
```

There isn’t a whole lot to this delegate (or other layout delegates). They’re required, but straightforward.

#### 9.5.4  Custom sliver
The final piece of Slivers worth discussing is that you can (like everything else in Flutter), create your own sliver classes. I’ve done this in the `Catalog` by creating the custom sliver that acts as each category header. On line ~74 in the catalog, this is being returned from the first `StreamBuilder`:

```dart
return CustomSliverHeader(
  headerText:
      snapshot?.data?.first?.category.toString() ?? "header",
);
```
That class, `CustomSliverHeader`, `is a custom widget`. `This code is all in the `sliver_header.dart` file. All that’s really going on in that file is that I’m creating a widget CustomSliverHeader, which returns Flutter’s built in `SliverPersistentHeader`, which itself returning a custom sliver delegate. That sounded confusing just typing it out, so as you’re looking at the code, this is the point: slivers are just widgets for infinite-scollables that are smarter about being rendered.

First, the widget class: `CustomSliverHeader`.
```dart
Listing 9.9. The setup for the custom header

// e_commerce/lib/widget/scrollables/sliver_header.dart
class CustomSliverHeader extends StatelessWidget {
  final String headerText;
  final GestureTapCallback onTap;

  const  CustomSliverHeader(
      {Key key, this.scrollPosition, this.headerText, this.onTap})
      : super(key: key);

@override
  Widget build(BuildContext context) {
    return SliverPersistentHeader(
      pinned: true,
      delegate: SliverAppBarDelegate(
        minHeight: Spacing.matGridUnit(scale: 4),
        maxHeight: Spacing.matGridUnit(scale: 8),
        child: Container(
          color: Theme.of(context).backgroundColor,
          child: GestureDetector(
            onTap: onTap,
// ... standard widget tree
```
So, the point, I suppose, is again that there isn’t much to using slivers. The big difference in widgets and sliver widgets is that sliver widgets have a `shouldRebuild` method. Which you can see in this snippit for the `SliverAppBarDelegate`:
```dart
// e_commerce/lib/widget/scrollables/sliver_header.dart -- line ~58
class SliverAppBarDelegate extends SliverPersistentHeaderDelegate {
  final double minHeight;
  final double maxHeight;
  final Widget child;
  SliverAppBarDelegate({
    @required this.minHeight,
    @required this.maxHeight,
    @required this.child,
  });

  @override
  double get minExtent => minHeight;
  @override
  double get maxExtent => math.max(maxHeight, minHeight);
  @override
  Widget build(
      BuildContext context, double shrinkOffset, bool overlapsContent) {
    return new SizedBox.expand(child: child);
  }

  @override
  bool shouldRebuild(SliverAppBarDelegate oldDelegate) {
    return maxHeight != oldDelegate.maxHeight ||
        minHeight != oldDelegate.minHeight ||
        child != oldDelegate.child;
  }
}
```
The `shouldRebuild` method is the new method here (but doesn’t it kind of explain itself?). This is what matters to us, as the developers, that makes slivers efficient. Basically what you want to say is "if this is the same widget and of the same size, don’t rebuild it as you scroll. The min and max extents are important, becuase slivers can change if they’re configured to, based on the other slivers in the viewport. Our sliver is always the same size though.

#### 9.6  Summary
In general, this chapter is about two important concepts that are separate from eachother, but likely will be used together in your app often: stream builders and custom scroll views. Infinite lists (and lists that have different widgets in them) often come from different lists that change, and they get that data from streams.

* Asynchronous programming is difficult, but hugely important in UI.
* Futures provide values that don’t yet exist, but will soon.
* `async` and `await` make async programming easier.
* `StreamController` objects are used to define streams and sinks.
* A `Sink` is the entry point of data for a stream.
* A `Stream` is what other pieces of code `listen` to in order to get data from streams as it becomes available.
* Streams can be transformed, and functions that take in streams and output a stream of transformed data is called a higher-order stream.
* A BLoC is a business logic component that relies on streams to build inputs and outputs your widgets can interact with as a the state-management logic in your widget.
* Flutter provides async widgets via the `StreamBuilder` class, which make async UI much cleaner.
* Stream builders are often, (but not necessarily), used to build inifinite and dynamic scroll views. These are usually built with `CustomScrollView` widgets.
* A `Sliver` is a special widget that’s smart about rebuilding, making scrollables more efficient.
* A `Sliver` builds its children with functions called delegates.