> **In this chapter:**
>* Using the AnimatedWidget
>* Using the canvas and CustomPaint class
>* The Paint class
>* AnimationControllers, Tweens, and Tickers
>* TweenSequence class
>* SlideTransition and other convenience widgets

Once again, I’m going to start by gushing about Flutter. The built-in widgets you’ve used so far has shown how easy it is to build an interface with Flutter. But, as I’ve said too many times, one of the best things about Flutter is how much control it gives you if you want it. And that’s what we’re going to talk about in this chapter. Using custom animations and the canvas, we’re going to tell Flutter exactly what we want it to paint. In particular, I’ll focus on two things:

1. Animations. Almost every display widget on the screen will animate in some way. All the text will change color, all the background objects will change color and move around the screen, and the icons will change shape and color. The most interesting note here is that the app will still be buttery smooth, despite changing entirely with almost every interaction from a user.
The bulk of the logic needed to build these animations is in the `forecast_page`, and a couple methods are in the `forecast_controller`. We’ll cover most of that in the next section. That section is going to be about animating the `Sun` class, from start to finish.

2. The canvas. I’m going to make that cloud shape in the background from scratch, using the `Canvas`. The canvas is a widget that allows you to tell Flutter what to draw, pixel by pixel. It requires math, and its fun.

Lastly, before we jump in, this topic requires some foundational explanation, but the coding will come soon enough.

#### 6.1  Introducing Flutter Animations
Using animations is perhaps the best thing you can do to make an app feel polished and slick, not to mention intuitive. Animations are often an afterthought, because they can require a good amount of work. Luckily, many built-in widgets in Flutter, especially Material Design widgets, have motion animations out of the box. And, custom animations aren’t so much harder, to boot.

In general, there are two main types of animations in Flutter: "tween" animations and physics-based animations. Tween animations, which is what we’re looking at in this chapter, are animations that have a defined start and finish. For example, in the weather app, when you choose a different time and the sun and cloud background position animates to a new location on the screen, they know that end position before the animation starts running.

A physics-based animation, on the other hand, rely on user interaction. A good example is a "fling" that you’ve probably seen in many apps. The harder you fling, the faster and longer the scroll. Physics-based animations are out of the scope of this chapter, but they’re similar enough that you’ll be comfortable with them. From here on out, I will be referring specifically to tween animations when I say animations.

An animation in Flutter is built by combining several pieces. You’ll have to implement four aspects of every animation:

* A Tween
* A Curve
* An AnimationController
* A Ticker (via a TickerProvider)

#### 6.1.1  Tweens
A tween is an object that’s given a start value and end value for whichever property you’re animating (for example, color). Tweens tell Flutter how to transition between those properties. Tweens are aptly-named, because it’s short for *in-betweening*.

Tweens, by default, map the value at any given moment in an animation to a number between 0.0 and 1.0. If you wanted to animate something from yellow to red, you can imagine this is happening (but with many more steps than I can diagram).

**Figure 6.1. Example of tween values**
![](https://dpzbhybb2pdcj.cloudfront.net/windmill/v-7/Figures/red_yellow_tween.png)

In our app, the position of the sun changes depending on the time of the day. That’s achieved by changing it’s position via it’s `Offset` property. An `Offset` describes the location of a widget relative to it’s own original position.

**Figure 6.2. Tweens map a value to the range of 0.0 to 1.0**
![](https://dpzbhybb2pdcj.cloudfront.net/windmill/v-7/Figures/positioned_diagram0001.png)

#### 6.1.2  Animation Curves
A `Curve` is used to adjust the rate of change of that animation over time, allowing them to speed up or slow down at certain points in an animation, rather than move at a constant speed. Flutter comes with a set of common, pre-defined curves in the `Curves` class. The default curve is called linear, because it moves at a constant speed.

“This is a comparison of a second common curve, which is provided by Flutter, is the `Curves.EaseIn`.”

**Figure 6.3. Ease in curve compared to default linear curve**
![](https://dpzbhybb2pdcj.cloudfront.net/windmill/v-7/Figures/curves_diagram.png)

#### 6.1.3  Ticker providers
A `Ticker` houses the logic that lives under the hood and gives the whole animation life. Tickers can be used by any object that wants to be notified every time a frame change triggers. Any time the screen changes in Flutter, it’s actually a series of tiny, subsecond re-renders. Flutter re-renders at 60 frames per second, and animations are really Flutter re-rendering so quickly, all the while painting animated objects at an interval of their final position, that it looks smooth and lifelike.

In practice, tickers are easy to use. You may never have to deal with a ticker directly, because Flutter gives you a nice class called `TickerProvider` that does just that. It provides a ticker to your class.

The most common way to use this is to extend a `State` class with a `TickerProviderStateMixin`. This gives your stateful widget all the internal functionality it needs to be notified of frame changes.
```dart
class _MyAnimationState extends State<MyAnimation> with TickerProviderStateMixin {}
```

#### 6.1.4  AnimationController
Finally, animations rely on an `AnimationController`. The AnimationController does what it’s name implies: controls animations. It’s aware of the `Ticker` object, which gives it life, and it turns around and tells objects what their new value is during an animation, provided by the tweens and curves.

The `AnimationController` API has useful methods to start and stop animations, reset an animation, play an animation in reverse, or repeat it indefinitely, and it has all kinds of properties that give you information about the animation as it’s happening. This gives you fine grain control over how an animation ticks. But as with everything else, you don’t have to do much more than call `AnimationController.start`, to make an animation happen.

In order to use an AnimationController, all you need to do is give it a ticker and a duration (how long the animation should last). Since your state object extends `SingleTickerProviderStateMixin`, it is the ticker itself.

This is a barebones example of making animation controller that animates the color of a widget. It’s meant to show the controller, not the tweens and curves.

```dart
Listing 6.1. Animation controller example

class _AnimatedContainerState extends State<AnimatedContainer>
        with SingleTickerProviderStateMixin {
  AnimationController _controller;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      vsync: this,
      duration: new Duration(milliseconds: 1000),
    );

    startAnimation();
  }

  Future<void> startAnimation() async {
    await _controller.forward();
  }

  @override
  Widget build(BuildContext context) {
    return Container(
      color: _colorTween.animate(_controller).value;
      child: //...
    );
  }
}
```

That’s the bulk of what you need for every animation you’ll ever make. A controller to… er… control everything, a ticker to feed it life, and a tween to map the value of that ticker to a value you can use.

#### 6.1.5  AnimatedWidget
An `AnimatedWidget` is a class that extends `StatefulWidget` and has little more functionality. (In fact, I encourage you to look at the source code for the `AnimatedWidget` class, you may be surprised how simple it is [4]).

An animated widget is "dumb". It doesn’t know anything about an animation controller or ticker. It doesn’t even know about the tween that’s updating it’s value. Rather, an animated widget requires a `Listenable` as an argument. An `Animation` is listenable. And an animation is returned from `Tween.animate(AnimationController)`, so that’s what we have to pass into it. We’ll look at that after we complete the `Sun`.

Because the `AnimatedWidget` requires a `listenable`, we can pass it on through straight to the super class of the `Sun` class. Which looks like this:
```dart
class Sun extends AnimatedWidget {
  Sun({Key key, Animation<Color> animation})
      : super(key: key, listenable: animation);
  // ...
```

Because this `listenable` is defined in the super class, it exists on the `Sun` (which is the subclass). No need to define it in the class.

There are only two other lines of code you need to update in this widget to make it work:
```dart
@override
Widget build(BuildContext context) {
  final Animation<Color> animation = listenable;
```
And finally, the color property in the `build` method.
```dart
decoration: new BoxDecoration(
  shape: BoxShape.circle,
  color: animation.value,
),
```

That’s all it takes to make a widget an `AnimatedWidget`. Most of the animation work is done above the widget in the tree, and passed into it. Which we’ll look at now, but this is the final code for the `Sun` class.

```dart
// weather_app/lib/widget/sun_background.dart
class Sun extends AnimatedWidget {
  Sun({Key key, Animation<Color> animation}) : super(key: key, listenable: animation);

  @override
  Widget build(BuildContext context) {
    final Animation<Color> animation = listenable;
    var maxWidth = MediaQuery.of(context).size.width;
    var margin = (maxWidth * .3) / 2;

    return new AspectRatio(
      aspectRatio: 1.0,
      child: new Container(
        margin: EdgeInsets.symmetric(horizontal: margin),
        constraints: BoxConstraints(
          maxWidth: maxWidth,
        ),
        decoration: new BoxDecoration(
          shape: BoxShape.circle,
          color: animation.value,
        // ...
```

#### 6.1.6  Implementing the AnimationController and Tween for the background
In order to make the animated widget work, you need to build that animation that you’ll pass into it. Continuing to work "backwards", this is how you the `Sun` class is defined with the animation passed through.
```dart
//weather_app/lib/page/forecast_page.dart -- line ~257
Sun(animation: _colorTween.animate(_animationController)),
```
So far, you know this about the process of using animated widget:

1. ??
2. Pass the animation into the animated widget.
3. Extract the animation value from `animated.value`.

In order to build that animation and fill the beginning steps, you have to actually build it. Based on that previous small code snippit, we need a `_colorTween` and an `_animationController`.

```dart
AnimationController _animationController;
ColorTween _colorTween;
```
For this app, we know that we’re going to fire off animations pretty often, every time the state changes. And it needs to be ready every time the state changes. So, I think we should create that `_animationController` as soon as the widget is made, for starters. Which means we should look to `initState`. The `ForecastPage.initState` method just calls `_render()`, because we’re also going to want to do this same logic when the configuration for ForecastPage updates. We’ll come back to that.

`ForecastPage.render` is calling a method called `ForecastPage.handleStateChange`. In that method you’ll, find the beginning of the logic for these animations. But, it also just calls more methods: `ForecastPage.buildAnimationController`, `ForecastPage.buildTweens` and `initAnimation`. The goal for the moment is to only animate the color of the sun, so I’ll focus on these three methods.

```dart
Listing 6.2. Basic animation controller functionality.

void _handleStateChange(int activeIndex) {
    //...
    _buildAnimationController();
    _buildTweens();
    _initAnimation();
    setState(() => activeTabIndex = activeIndex);
    //...
  }

void _buildTweens() {
  _colorTween = new ColorTween(
      begin: currentAnimationState.sunColor,
      end: nextAnimationState.sunColor,
  );
  //...
}

void _buildAnimationController() {
  _animationController?.dispose();
  _animationController =
      AnimationController(duration: Duration(milliseconds: 500), vsync: this);
  //...
}

void _initAnimation() {
  _animationController.forward();
  //...
}
```

Since you’ve already written the code to pass the animation into the sun and provided a value in the `Sun` class itself, this is all there is to making the a basic `AnimatedWidget` work.

“Because so many animations are tied together in this app, I think it makes more sense to finish some other tasks before we dive in deeper to the animations.”
[4] The source code can be found at [](github.com/flutter/flutter/blob/master/packages/flutter/lib/src/widgets/transitions.dart)

6.2  CustomPainter and the Canvas
The "clouds" widget is a bit more involved than the sun. It animates in two ways: it changes color and positions itself on and off the screen when necessary. Also, importantly, it’s not made up completely of widgets. The outer class that we’ve defined is a widget. And it’s child, the `CustomPainter` is also a widget. But the painter’s child isn’t (see figure  6.4 below). The painter is special because it let’s you draw directly on the screen. You can literally control every single pixel by drawing shapes and lines and dots of any color you want. And, because this is Flutter, there’s a nice API that makes it easy.

**Figure 6.4. Canvas in a widget tree**
![](https://dpzbhybb2pdcj.cloudfront.net/windmill/v-7/Figures/custom_paint_widget.png)

From a high level, the pieces of painters that matter are:
* the `Canvas`
* `Paint` class
* a `Size` parameter, which defines the size of the canvas.
* The `shouldRepaint` method
The canvas is the widget that you’re painting on. It’s blank when it’s defined. `Size` is an object that gives you the width and height of the canvas. Both of these are used to tell Flutter where to paint on the screen.

Figure 6.5. A Canvas is placed on the screen by it’s size.
![](https://dpzbhybb2pdcj.cloudfront.net/windmill/v-7/Figures/canvas_widget.png)

Then, the `Paint` class is used to define a "brush" of sorts. This class has an API to define the color you want to use, the `strokeWidth` you want to paint with (in pixels) and other properties such as `strokeCap`, which defines the shape of the end of the line. The methods on the `Canvas` object itself are what you use to tell Flutter what to draw with this paint. It has methods like `drawRect` and `drawLine`. Finally, the `shouldRepaint` method is a lifecycle method called whenever the configuration of this painter changes. It must be defined and return a boolean, and it’s used to Flutter to repaint. It’s expensive to repaint all the time, and this method gives you finer control over the repaint process.

That paragraph was all in the abstract, so this figure shows what the clouds image in the app is really made of:

**Figure 6.6. Canvas draw method examples**
![](https://dpzbhybb2pdcj.cloudfront.net/windmill/v-7/Figures/cloud_paint_diagram.png)

#### 6.2.1  Define the CustomPainter and Paint object
Let’s walk through the `Clouds` class in the weather app to make that less abstract:
```dart
Listing 6.3. The clouds class defines what’s needed for the CustomPainter

// weather_app/lib/widget/clouds_background.dart
class Clouds extends AnimatedWidget {
  final bool isRaining;

  Clouds({Key key, Animation<Color> animation, this.isRaining = false})
      : super(key: key, listenable: animation);

  Widget build(BuildContext context) {
    final Animation<Color> animation = listenable;

    var screenSize = MediaQuery.of(context).size;
    var _paintBrush = Paint()
      ..color = animation.value
      ..strokeWidth = 3.0
      ..strokeCap = StrokeCap.round;

    return Container(
      height: 300.0,
      child: CustomPaint(
        size: screenSize,
        painter: CloudPainter(
          cloudPaint: _paintBrush,
          isRaining: isRaining,
        ),
      ),
    );
  }
}
```
Next look at the actual `CloudPainter` class, which is where we draw on the canvas. This is the minimum you need for every custom painter object: A `paint` method, and a `shouldRepaint` method.
```dart
Listing 6.4. The bare minimum of the custom painter

// weather_app/lib/widget/clouds_background.dart -- line ~31
class CloudPainter extends CustomPainter {
  final bool isRaining;
  final Paint cloudPaint;
  CloudPainter({this.isRaining, this.cloudPaint});

  @override
  void paint(Canvas canvas, Size size) {
    // paint on the canvas
  }

  @override
  bool shouldRepaint(CustomPainter oldDelegate) => false;
}
```

#### 6.2.2  CustomPainter paint method
All the logic in this class is in the `paint` method. Painting on the canvas, in it’s simplest explanation, requires you to give exact instructions of where to paint what on the canvas. If you were instructing a friend what to paint on a canvas, you might say, "Draw a red line from the top-left corner to the center." Then follow that up with another command, "Draw a small blue circle in the center."

There are two things to note: First, order matters. In my human example above, the blue circle would overlap the red line, because it was drawn second. Secondly, I just need to remind you that computers are dumb. And because they’re dumb, we have to use math and give it precise commands. Unlike our smart friend, a computer doesn’t know what the 'top-left corner' is. Precision is key in paint methods.

Before we look at the code, think about what we’re trying to achieve. This cloud:

image::../../figures/diagrams/chapter_4/sun_and_moon.png

These are some of the ideas we need to think about.
* It’s centered.
* The canvas is the entire screen, but the drawing doesn’t take up the whole screen.
* It’s made up of four shapes: three circles and one rectangle with rounded corners.
* Phone screens are all different sizes, so we have to make the measurements malleable.
* With all this in mind, we know we need to use math to get some positions on the screen, then draw 4 shapes to the screen. This is what that ends up looking like:

```dart
Listing 6.5. Half of the CloudPainter.paint method.

// weather_app/lib/widget/clouds_background.dart -- line ~38
@override
void paint(Canvas canvas, Size size) {
  var rectTop = 110.0;
  var rectBottom = rectTop + 40.0;

  var figureLeftEdge = size.width / 4;
  var figureRightEdge = size.width - 90.0;
  var figureCenter = size.width / 2;
  var cloudBaseRect = new Rect.fromPoints(
    Offset(figureLeftEdge, rectTop),
    Offset(figureRightEdge, rectBottom),
  );
  var cloudBase = new RRect.fromRectAndRadius(
    cloudBaseRect,
    Radius.circular(10.0),
  );
  canvas.drawCircle(Offset(figureLeftEdge + 5, 100.0), 50.0, cloudPaint);
  canvas.drawCircle(Offset(figureCenter, 70.0), 60.0, cloudPaint);
  canvas.drawCircle(Offset(figureRightEdge, 70.0), 80.0, cloudPaint);
  cloudPaint.strokeWidth = 3.0;
  canvas.drawRRect(cloudBase, cloudPaint);
  // ...
```

There are three general steps that I use to frame the logic of my paint methods: 1. Define the spacial and location variables (such as `rectTop` from the code snippet. 2. Create the figure instances like `Rect`. 3. Paint the figures to the canvas with `Canvas.draw` methods.

The first step makes the code easier to understand and reason about. In this example, `size.width / 4` will give exactly 1/4 of the screen as padding to the left. `size.width - 90.0` is arbitrary, again, but will give exactly 90 pixels of padding to the right. The clouds are look better slightly offset from center. (I think, but I’m not an artist, so take that with a grain of salt).

The second step is the "preparation". The canvas has built in methods to draw common shapes. `drawRect` draws a rectangle, `drawLine` a line, etc. To draw a rectangle, you need to create a rectangle object to give to the `drawRect` method. The `Rect` class, `Line` class, `Circle` class, and all the other shapes have constructors that use different measurements to define the shape. For example, I used `Rect.fromPoints`, which takes two offsets and draws the smallest rectangle that encloses both points. There’s also `Rect.fromLTWH`, which means "from left, top, width, height". I encourage you explore all the options when you next use a custom painter.

I went with `Rect.fromPoints` because it takes two `Offset` objects as parameters. Offsets are used quite a bit in UI development, and it’s comfortable for me to work with them. That’s the only reason. None of the constructors are necessarily better than any of the others.

Offsets represent points from the origin of the vector, which is the top-left corner of the canvas (the top-left corner of the screen in our case). Provide the constructor with the top-left point of the rectangle and the bottom-right point. It’s important to note that thus far we haven’t drawn anything on the canvas. We’ve just declared an instance of the `Rect` class.

The rectangle we’re building for the cloud base needs to have rounded corners. Luckily, Flutter gives us a class called `RRect`. You can pass the previously built rectangle to `RRect.fromRectAndRadius` and it’ll give us what we want.

Now, for step three, you need to tell the canvas to execute the painting of the figures. `drawCircle` is different than drawing a rectangle because you don’t have to create a circle object. It only needs an `Offset` that represents the center of the circle, a double that represents the radius of the circle, and a `Paint` object.

Now you only need to paint the base of the clouds that give it that nice flat bottom. We already did the hard part of the creating the `RRect`, you just need to call `canvas.drawRRect` and you’re good to go.

That’s the entirety of the cloud portion of the drawing. There’s a whole second part of the `paint` method that draws the raindrops (when appropriate). It’s a lot more of the same: math and drawing lines. It’s already written in the project repository, and I encourage you look at it.

second part of the paint method that draws the raindrops (when appropriate). It’s a lot more of the same: math and drawing lines. It’s already written in the project repository, and I encourage you look at it.

At this point you should see the cloud on your screen. It’s nice, but we need to animate it — both the color and the position. In the next section, we’re going to do some more complicated animations.

#### 6.3  Animations Again: Staggered animations, TweenSequence, and built in animations

The important information in this section is going to come from coordinating multiple animations from the `ForecastPageState` class. Animating a single value is valuable for UI animations who’s purposes are give users feedback when they interact with the app. But style animations, like the background in this app, often contain complex animations that have to be executed in specific order and with correct timing.

In this app, there are seven tweens animating more than ten properties accross many widgets. Flutter has methods and objects to connect these pieces of animations into one big show. Particularly, I’m going to talk about built in `AnimatedWidgets`, like the `SlideTransition`, using a custom class to manage the state of the animation, and my favorite convenience feature: `TweenSequence`.

#### 6.3.1  Creating a custom animation state class
What I really want to do is decide the starting and ending animation colors on the fly, right every time the state changes in the app. Specifically, everytime the user selected a new time of day. My solution to this isn’t Flutter specific, but it is useful, I think, to see how I coordinated all the different animations together.

Let’s think through the logic of making this work. What do we know so far and what do we want to do here?

* We know that every tween in an animation needs a beginning and end.
* We know that in our case, we need to get those values every time the state is updated, which happens when the user taps a new time in the tab bar.
* We know that the values are based off the time that’s selected and the weather forecast at the time that was selected. For example, the color values and sun’s position are based on the time of day, and the visibility of the clouds and rain is based on the type of weather.

With this in mind, what do you need to accomplish?

* On tap, you need to gather all the time and weather for the time selected. (you also need to know what the current weather data is, for the beginning parts of the tweens).
* With that data, we need to decide what values our tweens will have.
* We need to build the tweens, pass them into the widgets, and call `forward` on the animation controller.

In the app, this involves a few different moving parts.
* ForecastAnimationState - This is a helper class I added to make this easier to reason about. All it does in reality is hold reference to all the different values that this animation needs at any given `begin` or `end` state. It also has a factory constructor that picks all the correct values based on the weather and time data. It’s basically a giant, hard-coded switch statement.
* `ForecastController.getDataForNextAnimationState` - This method coordinates the logic needed to create a `ForecastAnimationState`. It says, "Oh, you chose that tab index? Okay well let me grab the associated time and weather data, then build a new `ForecastAnimationState`. In the app, the code looks like this:

```dart
// weather_app/lib/controllers/forecast_controller.dart
ForecastAnimationState getDataForNextAnimationState(int index) {
  var hour = getSelectedHourFromTabIndex(index);                              #1
  var newSelection = ForecastDay.getHourSelection(selectedDay, hour);         #2
  var endAnimationState = new ForecastAnimationState.stateForNextSelection(   #3
      newSelection.dateTime.hour, newSelection.description);

  // update selectedHourlyTemperature to currentChoice
  selectedHourlyTemperature = newSelection;                                   #4
  return endAnimationState;
```

* `ForecastPageState.currentAnimationState `and `ForecastPageState.nextAnimationState` - These two variables are used to set values of all the tweens needed in the `ForecastPageState._buildTweens` method. Whenever a state change happens and the animation kicks off, we’ll grab the values we need to animate to, build the tweens, and then set the `currentAnimationState` to end values of the just-fired animation, because that’s where we want the next animation to begin. This happens in the `ForecastPageState.handleStateChange` method.

```dart
Listing 6.6. ForecastPageState.handleStateChange method

// weather_app/lib/page/forecast_page.dart -- line ~78
void _handleStateChange() {
  nextAnimationState = _bloc.getDataForNextAnimationState(_tabController.index);   #1
  _buildAnimationControllers();
  _buildTweens();
  setState(() {   #2
    activeTabIndex = _tabController.index;
  });
  _initAnimation();
  // for next time the animation fires
  currentAnimationState = nextAnimationState;   #3
}
```

Finally, use the values from the `ForecastAnimationState` objects. This’ll update the tweens, which makes the animations you’ve written thus far work as expected.

```dart
Listing 6.7. ForecastStatePage._buildTweens method

// weather_app/lib/page/forecast_page.dart -- line ~133
void _buildTweens() {
  _colorTween = new ColorTween(
    begin: currentAnimationState.sunColor,   #1
    end: nextAnimationState.sunColor,
  );
  _cloudColorTween = new ColorTween(
    begin: currentAnimationState.cloudColor,
    end: nextAnimationState.cloudColor,
  );
  // ...
}
```

That’s the complete process for one piece of the animation: the cloud and sun colors.

#### 6.3.2  Built in animation widgets: SlideTransition
The cloud and sun background pieces also animated their positions. Flutter gives us a nice widget for just that use case, because they’re so good to us. There’s a widget called `SlideTransition` that does just that: it slides it’s `child` widget across the screen using `Offset` coordinates. Flutter also comes with a `ScaleTransition`, SizeTrans`ition, and `FadeTransition`, among many others.

These widgets all extend `AnimatedWidgets`, so you can use them exactly how we use the `Sun` and `Cloud` animations. You only need to pass it an animation to the `position` property and a `child` widget. Since we’re already using a `Stack`, all you need to do is replace the `Positioned` widgets with `SlideTransition` widgets, passing them the same widgets to the `child` property. Now, we need to build the tweens. We need two different `Tween<Offset>` objects, one for the sun and one for the clouds. The sun is controlled by the `Tween<Offset> _positionOffsetTween` variable. It’s exactly like the tweens we’ve used thus far. You can see this is in the `ForecastPageState._buildTweens`.

```dart
// weather_app/lib/page/forecast_page.dart -- line ~113
void _buildTweens() {
    // ..
    _positionOffsetTween = new Tween<Offset>(
      begin: currentAnimationState.sunOffsetPosition,
      end: nextAnimationState.sunOffsetPosition,
    );
    // ...
```
Now, the sun background widget needs to know about that tween. That’s set up in the build method of this same widget:

```dart
// weather_app/lib/page/forecast_page.dart -- line ~251
 // ...
SlideTransition(
  position: _positionOffsetTween.animate(
    _animationController.drive(
      CurveTween(curve: Curves.bounceOut),
    ),
  ),
  child:
      Sun(animation: _colorTween.animate(_animationController)),
),
 // ...
```
Nothing new here, but this next part will be more interesting: animating the clouds.

The first time I wrote this animation, I used the same animation controller to move the clouds on and off screen, which had unwanted results. It was animating directly (diagonally) to it’s place off screen. because the cloud and sun figures also animate up and down based on the time of day. This is what was happening (Figure 6.10):

**Figure 6.7. Single position slide animation**
![](https://dpzbhybb2pdcj.cloudfront.net/windmill/v-7/Figures/cloud_animation_wrong.png)

This positioning animation should be two steps. It needs to animate up or down, and then horizontally on or off the screen. This should be happening (Figure 6.11):

**Figure 6.8. Staggered position slife animation**
![](https://dpzbhybb2pdcj.cloudfront.net/windmill/v-7/Figures/cloud_paint_diagram.png)

These are two steps to make this happen:

1. Use a second `AnimationController` that has a longer duration. The clouds should animate vertically at the same rate as the `Sun`, then the extra time is used to slide on or off the screen.
Use a `TweenSequence`. This class takes a `List` of tweens, each of which it’ll execute in the order you’ve listed them, distributing over the duration of the animation.

The first step, making a new `AnimationController`, is nothing new. It’s absolutely reasonable to add multiple `AnimationController` classes to a single widget. In fact, Flutter has a different ticker mixin for this exact case: `TickerProviderStateMixin` (rather than `SingleTickerProviderStateMixin`.

I’ve declared the variable in the `ForecastPageState` class: A`nimationController _weatherConditionAnimationController`. To use it, you should assign that variable a new `AnimationController` object in `ForecastPageState._buildAnimationControllers`, then initialize the animation in ForecastPageState._initAnimations. This is all more of the same, so I won’t cover it, but you can see all the code in the relavent methods mentioned in this paragraph.

#### 6.3.3  TweenSequence
The `TweenSequence`, however, is new, so let’s talk about it. `TweenSequences` can be used anywhere a `Tween` could be, because they are just complicated tweens under the hood, with a nice API for the developer.

It’s constructor takes a `List` of `TweenSequenceItems`. `TweenSequenceItems` require two parameters: a tween and this item’s weight. An item’s weight is relative to all the other items, so the number assigned to any given weight it arbitrary. Flutter adds up the total weights and divides them by the number of items.

> TIP
I like my math to be easy to reason about, so I like use weights that add up to 100. You don’t have to do this. The only requirement is that the total weight of all the items is greater than 0.0.

This code belongs in the `ForecastPageState._buildTweens` method:

```dart
 Listing 6.8. In the ForecastPageState._buildTweens method.

// weather_app/lib/page/forecast_page.dart --- line ~113
void buildTweens() {
  // ...
  // line ~135
  var cloudOffsetSequence = new OffsetSequence.fromBeginAndEndPositions(
          currentAnimationState.cloudOffsetPosition,
          nextAnimationState.cloudOffsetPosition);
  _cloudPositionOffsetTween = new TweenSequence<Offset>(
    <TweenSequenceItem<Offset>>[
      TweenSequenceItem<Offset>(
        weight: 50.0,
        tween: Tween<Offset>(
          begin: cloudOffsetSequence.positionA,
          end: cloudOffsetSequence.positionB,
        ),
      ),
      TweenSequenceItem<Offset>(
        weight: 50.0,
        tween: Tween<Offset>(
          begin: cloudOffsetSequence.positionB,
          end: cloudOffsetSequence.positionC,
        ),
      ),
    ],
  );
```
That’s that. This class is extremely useful for staggering animations. I’ve spent hours writing some really ugly code to get the same effect before I knew this class existed. Anytime you want to animate the same property on the same item multiple times in a single animation, this is what should pop in your mind.

The last step of this animation, again, is to wrap the `Cloud` widget with a `SlideTransition` in the build method.
```dart
Listing 6.9. In the ForecastPageState.build method

// weather_app/lib/page/forecast_page.dart -- line ~260
SlideTransition(
  position: _cloudPositionOffsetTween.animate(_weatherConditionAnimationController),
  child: Clouds(
    isRaining: isRaining,
    animation: _cloudColorTween.animate(_animationController),
  ),
),
```
That’s all there is to the most complicated animation in the app. The only thing left to do is to animate some colors, which will be quick and easy after that.

#### 6.4  Reusable color transition custom widgets
In the `lib/widget/` directory, you’ll notice four classes that are similar:

* ColorTransitionText
* ColorTransitionIcon
* ColorTransitionBox
* TransitionAppbar

These four classes are used all over the app to transition all the colors. The `ColorTransitionBox` animates the background, and the `TransitionAppbar` the color of the appbar. The other two do just what they say do. Because the background color of the app changes drastically, the text and icons all over the app must change to be readable.

From here, I’d like to walk through, quickly, the `ColorTransitionBox`, and then leave the rest up to you to explore on your own if you’d like.

```dart
// weather_app/lib/widget/color_transition_box.dart
class ColorTransitionBox extends AnimatedWidget {
  final Widget child;

  ColorTransitionBox(
      {this.child, Key key, Animation<Color> animation})
      : super(key: key, listenable: animation);

  @override
  Widget build(BuildContext context) {
    final Animation<Color> animation = listenable;
    return DecoratedBox(
      decoration: BoxDecoration(
        color: animation.value,
      ),
      child: child,
    );
  }
}
```

That’s all there is to it. The rest of the non-Flutter code to grab the correct values has been implemented.

#### 6.5  Summary
If you take one thing away from this chapter, I hope that it’s this: Flutter make’s animations incredibly simple. Like all things in Flutter (and life), you can dive deep and make things complicated. But why should you? Flutter’s built in animation library covers every use case I’ve ever had. I’ve never had to write custom `lerp` methods or go much deeper than the `AnimatedWidget`. They did a fantastic job at solving this problem for us, so we should take advantage of that. This is especially nice because animations are often "nice to haves", but when they’re this easy theres no reason not to polish the UI up with animations.

* Many Material Design widgets have built in motion
* Animations require the developer to implement three pieces at a minimum: a controller, a tween and a ticker. Flutter will take care of the curve for you, if you don’t want to customize it.
* Tweens map values of an animatable property to a number scale.
* Tickers are what give life to animations, calling their callback on every frame change.
* Classes that have `AnimationController` objects as properties should extend `SingleTickerProviderStateMixin` or `TickerProviderStateMixin`.

* All widgets which extend `AnimatedWidget` require an animation as a parameter, which provides the value to whichever property you’re animating.
* You can paint exactly what you want, pixel by pixel, using the `CustomPaint` widget.
* The `CustomPaint` widget takes a child which extends `CustomPainter` and has a `paint` method.
* Painting to the canvas is generally a series of drawing shapes and lines using the `Canvas` class.
* The `TweenSequence` class is extremely useful in making staggered animations.

