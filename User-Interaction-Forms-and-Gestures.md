> **In this chapter:**
>* User interaction with GestureDetectors
>* Special interaction widgets like Dismissible
>* Flutter forms
>* Text input, dropdown lists, and more form elements
>* Using Keys to manage Flutter forms

This chapter is going to be the shortest, most succinct chapter. That doesn’t mean it’s not important though. This chapter is about letting users interact with your Flutter app. At the end of the day, it seems like all mobile apps basically have one job: to make it easy for a human to interact with a giant amount of data. And half of that interaction is allowing users not only to look at data, but also add to it and change it. This chapter is about that: letting users add and change data in your app. Specifically, this covers two different kinds of interaction: gestures and forms.

#### 5.1  User interaction and gestures
Gestures are any kind of interaction event: taps, drags, pans, etc. I’ll cover that first. To be honest, though, there isn’t much involved here. Flutter, of course, has a convenience widget that allows you to add gesture detectors to whichever portions of your widget tree that you’d like.

You’ve already used many gestures. All the button widgets that have `onPressed` and `onTap` are just convenient wrappers around gesture detectors.

#### 5.1.1  The GestureDetector widget
The core user-interaction widget is called a `GestureDetector`. You can wrap this widget around any other widget and make that child listen to interaction. The `GestureDetector` only needs two things passed to it: a widget (it’s `child`), and a callback to correspond to a gesture.

“It only needs one gesture callback, but it can be passed many different callbacks and respond differently based on the gesture it detects. This is just a couple of the methods available on it:”
* onTap
* onTapUp
* onTapDown
* onLongPress
* onDoubleTap
* onHorizontalDragStart
* onVerticalDragDown
* onPanDown
* onScaleStart

That’s only a few of nearly 30 gestures that it can work with. All of these arguments will call the callback you pass it, and some will pass back "details". `onTapUp`, for example, passes back an instance of the `TapUpDetails` object, which exposes the `globalPosition`, or location on the screen that was tapped. Some of the more complicated gestures, like the drag related ones, pass back more interesting properties. You can get the details about the time a drag started, the position it started, the position it ended, and the velocity of the drag. This let’s you manipulate drags based on direction, speed, etc.

In the weather app, I used a `GestureDetector` in the Forecast page. If you double-tap anywhere on the screen, you should see the temperature switch between fahrenheit and celsius.

![](https://dpzbhybb2pdcj.cloudfront.net/windmill/v-7/Figures/gesture_detector_double_tap.png)

Because you’ve seen button widgets that use gesture callbacks, implementing this should look familiar. I wrapped the entire body of the `Scaffold` with a gesture detector:

```dart
// weather_app/lib/page/forecast_page.dart -- line ~ 246
body: GestureDetector(
    onDoubleTap: () {
      setState(() {
        widget.settings.selectedTemperature == TemperatureUnit.celsius
            ? widget.settings.selectedTemperature =
                TemperatureUnit.fahrenheit
            : widget.settings.selectedTemperature = TemperatureUnit.celsius;
      });
    },
    onVerticalDragUpdate: (DragUpdateDetails v) => _handleDragEnd(v, context),
    child: ColorTransitionBox(
```

The `GestureDetector.onVerticalDragUpdate` method is called repeatedly as you drag your finger up or down on the screen. When called, it passes you some information about the drag, via the `DragUpdateDetails` class. This class gives several details, but the only that I care about is `DragUpdateDetails.globalPosition`. Anytime `onVerticalDragUpdate` is called, you have the opportunity to perform an action based on the location of the drag. You can see an example of this in the `ForecastPageState._handleDragEnd` method that’s called when `onVerticalDragUpdate` is called.

The purpose of this function is the same as the `TimePickerRow`. While dragging up and down, you’re selecting a new time of day to display. In the following function, I made this happen by "conceptually separating" the screen into 8 rows (one for each time of day choice). Basically, the top 8th of the screen represents "3:00" in the `TimePickerRow`. The second 8th represents "6:00", and so on.

If you’re dragging a pointer vertically on the screen and enter the top 8th segment, then the app will update to display the forecast for that time of today.

```dart
// weather_app/lib/page/forecast_page.dart -- line ~90
void _handleDragEnd(DragUpdateDetails d, BuildContext context) {
    var screenHeight = MediaQuery.of(context).size.height;
    var dragEnd = d.globalPosition.dy;
    var percentage = (dragEnd / screenHeight) * 100.0;

    var scaleToTimesOfDay = (percentage ~/ 12).toInt();
    if (scaleToTimesOfDay > 7) scaleToTimesOfDay = 7;

    _handleStateChange(scaleToTimesOfDay);
}
```

>NOTE
If you’re wondering where these numbers come from, it’s in the generated in the `WeatherDataHelper` class that makes fake data for the app. In a method called `dailyForecastGenerator`, I’m generating 8 "forecasts" per day. In the UI, this is where the `TimerPickerRow` choices come from. (`3:00`, `6:00`, `9:00`, etc).

```dart
Listing 5.2. Generating weather data for the weather app

// weather_app/lib/utils/generate_weather_data.dart -- line ~51

// I've omitted lines that aren't relavent
ForecastDay dailyForecastGenerator(City city, int low, int high) {
    List<Weather> forecasts = [];
    // ...
    for (var i = 0; i < 8; i++) {
      startDateTime = startDateTime.add(new Duration(hours: 3));
      int temp = _random.nextInt(high);
      // ...
      forecasts.add(Weather(
          city: city,
          dateTime: startDateTime,
          description: randomDescription,
          cloudCoveragePercentage: generateCloudCoverageNum(randomDescription),
          temperature: tempBuilder));
     // ...
    }
```

Back in the main thread of this section, we’ll now look at another built-in gesture widget that’s used in basically every app these days.

#### 5.1.2  Dismissible
The `Dismissible` widget is worth highlighting because it’s trickier than some of the other gesture widgets. First, look at how it’s used in the weather app. If you aren’t sure what a dismissible is, you will when you see the picture just below this.

On the settings page, you can remove cities from your list of cities by swiping the list item from right to left.

![](https://dpzbhybb2pdcj.cloudfront.net/windmill/v-7/Figures/dismissible_example.png)

This is implemented by using the `Dismissible` widget, which does require a bit more setup than most widgets.

```dart
Listing 5.3. Using dismissible in a collection

// weather_app/lib/page/settings_page.dart -- line ~81
child: ListView.builder(
  // ...
  itemBuilder: (BuildContext context, int index) {
    var city = allAddedCities[index];
    return Dismissible(
      onDismissed: (DismissDirection dir) => _handleDismiss(dir, city),
      background: Container(
        child: Icon(Icons.delete_forever),
        decoration: BoxDecoration(color: Colors.red[700]),
      ),
      key: ValueKey(city),
      child: CheckboxListTile(
        value: city.active,
        title: Text(city.name),
        onChanged: (bool b) => _handleCityActiveChange(b, city),
      ),
    );
  }),
```

Along with being another great example of what you get out of the box with Flutter, dismissible is important because it’s one of the only places that a `Key` is required. This is another great example of why keys are important. What would happen if you didn’t give dismissibles in a list a key? Imagine you have a list with 5 dismissible list items.

1. Dismissible number 2 is swiped, thus removed from the widget tree.
2. Flutter knows that it’s time to rebuild.
3. All the elements in the element tree start looking at their associated widgets. Because there are no keys, the elements are all looking at run time type only.
4. One element says "hey, my widget is gone, maybe it’s one of these others of the same type in this collection."
5. All the other dismissibles already have elements pointing at them. The elements don’t know who’s supposed to be pointing at what.

The key solves this, because an element know’s by looking at the other dismissible widgets that they all have different keys, so the element know’s that it’s no longer needed.

Other than that the dismissible isn’t "different" or relative to other user interaction widgets. In fact, that’s the point. There are several widgets with built-in interaction, and you’ll likely create some of your own. They all follow the same basic rules though:

* They generally just wrap widgets that aren’t interactive
* They provide callbacks, which pass details of the interaction event, which gives you a chance to handle the data how you’d like.

The interactive widgets that are different are those that take input, such as a text input field. They’re similar, but a bit more involved. Those are covered in the remainder of this chapter when I talk about forms.

#### 5.2  Flutter Forms
One time in an interview I was asked, "Which React forms library do you use?" (Which blew my mind, because… what?) The point is, I suppose, that forms and handling user input can be cumbersome. There’s a lot of state to deal with, events to listen to, and values to massage into something useful.

In the rest of this chapter, I’m going to spend a lot of time walking down a single file, explaining how forms work in Flutter.

In particular, there’s a form in the weather app that allows the user to add new cities to their list. Let’s talk about the requirements to make that happen:

* Create the UI for the form, including form fields for users to add whatever data they’d like.
* Implement a way to grab all the data when the user submits the form. Ideally, this will all happen at once, rather than field by field. (Hint: keys are helpful!)
* Validate the data. Make sure it’s data that we can use.
* Pass the data onto the business logic, which will know how to submit the data to a database (we aren’t concerned with the logic in this chapter).

First, I want to cover how forms work from a high level.

#### 5.2.1  The Form widget
The Form widget is a wrapper of sorts that provides some handy methods and integrates with all the form field widgets in it’s sub-tree. Using a form element is straight forward, but under the hood it’s doing solid work for you.

In the weather app, a form is created in the `weather_app/lib/page/add_city_page.dart`.
![](https://dpzbhybb2pdcj.cloudfront.net/windmill/v-7/Figures/form_annotated.png)

The built-in Flutter (and easiest) way to interact with the `Form` is by passing it a key of type `FormState`. The widget will associate that global key with this particular form’s state object, giving you access to the state object anywhere. This is the only situation in which I recommend using global keys.

The "structure" of all the pieces of this form can be imagined like this.
![](https://dpzbhybb2pdcj.cloudfront.net/windmill/v-7/Figures/element_widget0001.png)

#### 5.2.2  GlobalKey<FormState>
Using a "form key" (which is actually a global key of sub-type form state), is a lot like using a controller on many other widgets. In the previous app, I explained the `TabController`, for example.

All the logic and properties that lives on the `FormState` object are accessible via this key that you’ll make, because it’s literally a reference to a `FormState` object.

When you create a `GlobalKey<FormState>`, you truly are only creating a key. But, then you pass it to the `Form` widget’s `Key` property, so now you have a reference to that Form widget’s state object.

In the `weather_app/lib/page/add_city_page.dart`, building the form starts with the key and creating the form.

```dart
Listing 5.4. Initial form setup in weather app

// weather_app/lib/page/add_city_page.dart -- line ~ 16
class _AddNewCityPageState extends State<AddNewCityPage> {
  // ...
  final GlobalKey<FormState> _formKey = new GlobalKey<FormState>();

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      // ... appbar
      body: Container(
        padding: EdgeInsets.symmetric(horizontal: 8.0),
        child: Form(
          key: _formKey,
          // ...
```

Now that you have reference to your forms state object, you can put it to use.

#### 5.2.3  Using FormState
It’s worth mentioning that you don’t have to use the form widget in Flutter. You can just throw a text input widget into your app wherever, and manage it’s input. In fact, you’ll probably want to do that a lot.

Forms come in handy because of what `FormState` gives you. Rather than having to manage every form element in a long form, FormState will do it for you. When you build a form, you can pass any widgets you want to it’s `child` property. And you can build whatever kind of widget sub-tree you want from here. But, any widget in the sub tree that will take input can be wrapped in a `FormField` widget, and then the form becomes "aware" of it. That’s the main advantage of a form. You’ll see later that when you call `FormState.save` it’ll go ask all the form elements if they have anything to save.

#### 5.3  FormField widgets
As I mentioned, all form input elements should be wrapped with `FormField` widget. For example, you can use a `Checkbox` in your form, but should be wrapped in a `FormField`.

```dart
return FormField(
    child: Checkbox(
        //...
```
Most of the functionality that comes from forms lives on these widgets. There are three different form field widgets:
* FormField the standard field.
* TextFormField - A specialized form field that wraps a TextField
* `DropdownButtonFormField` - A convenience widget that wraps a `DropdownButton` in a `FormField`.
* 
In particular, in this app, there are three form fields, one of each type:
![](https://dpzbhybb2pdcj.cloudfront.net/windmill/v-7/Figures/form_with_field_vars.png)

#### 5.3.1  FormField Properties
Although all three fields are different form field types, they all have similar properties. Look at the code for the `_titleField` widget I’ve made, which is a `TextFormField`.

```dart
Listing 5.5. TextFormField example

// weather_app/lib/page/add_city_page.dart -- line ~112
Widget get _titleField {
  return Padding(
    padding: const EdgeInsets.only(top: 8.0),
    child: TextFormField(
      decoration: InputDecoration(
        border: OutlineInputBorder(),
        helperText: "Required",
        labelText: "City name",
      ),
      autofocus: true,
      autoValidate: true,
      validator: (String val) {
        if (val.isEmpty) {
          return "Field cannot be left blank";
        }},
      onSaved: (String val) => _newCity.name = val,
    ));
}
```

The three properties of that `TextFormField` that we’re interested in are the `validator` callback, the `autoValidate` flag, and the `onSaved` callback.

* `validator` is a convenience method that takes a callback. When the form field calls the callback, it passes the the input of this field as a `String`. Whatever is returned from this callback is added as a error text to the field. If it returns nothing or `null`, then the form field doesn’t show any error text: The validator function is the first of a few that’s handled by the `FormState` on all the form fields in it’s sub-tree. You validate validate all your fields by calling `FormState.validate()`, which turns around and calls all the form field’s validator callbacks. Or, you can `autoValidate` a widget.

* `autoValidate` is a boolean flag on form fields, that when true calls the validator callback straight away and whenever the form field changes. This is my preferred method, simply for the UI value. It gives instant and direct feedback to the user.
* `onSaved` works the same as `validator`. It’s called whenever `FormState.save()` is called. The text field example in the weather app, again, looks like this:

```dart
Listing 5.6. TextFormField example

// weather_app/lib/page/add_city_page.dart -- line ~112
Widget get _titleField {
    // ...
    child: TextFormField(
      decoration: InputDecoration(
     // ...
      autofocus: true,
      autoValidate: true,
      validator: (String val) {
        if (val.isEmpty) {
          return "Field cannot be left blank";
        }},
      onSaved: (String val) => _newCity.name = val,
    ));
}
```
**DropdownFormButton**
The `DropDownFormButton` is also an extension of the `FormField` widget. This widget, though, is a super convenience widget provided by Flutter (in my opinion). It wouldn’t be insignificant to build this widget on your own (the dropdown button, that is).

In the weather app, this is what the `DropdownButtonFormField` looks like:
![](https://dpzbhybb2pdcj.cloudfront.net/windmill/v-7/Figures/dropdown_annotated.png)

>NOTE
In this example I’m using a class called `DropDownExpanded`, which is I wrote myself. It’s a replica of the `DropdownButtonFormField` widget in every way, except for the fact that it can optionally pass a boolean flag called `isExpanded`. If `isExpanded` is true, this replacement drop down widget will tell it’s child, which is a `DropdownButton`.

Starting at this point, this is exactly how the original `DropdownButtonFormField` behaves. But on that child `DropdownButton`, if `isExpanded` is true, it’ll wrap it’s childen in an expanded.

This is the code inside the `DropdownButton` that creates the widget that displays the value of the button.

```dart
Widget result = DefaultTextStyle(
       style: _textStyle,
       child: Container(
        // ...
         child: Row(
           mainAxisAlignment: MainAxisAlignment.spaceBetween,
           mainAxisSize: MainAxisSize.min,
           children: <Widget>[
             widget.isExpanded ? Expanded(child: innerItemsWidget) : innerItemsWidget,
             // ...
```

I don’t know if this is intentional by the Flutter team, or a bug. And, by now, it might be fixed or changed. But this is a cool lesson: If you want to tweak something in Flutter to work for you, it’s pretty easy to do in a variety of ways. Everything is available to you as the developer in the API, and it’s open sourced if you need to just make a copy that works for you.

It’s implemented similarly to the TextF`ormField, with the notable different that you pass it items to display when the button is tapped.

```dart
Listing 5.7. DropdownButtonFormField example

// weather_app/lib/page/add_city_page.dart -- line ~114
Widget get _countryDropdownField {
  return DropDownExpanded<Country>(
    isExpanded: true,
    decoration: InputDecoration(
      border: OutlineInputBorder(),
      labelText: "Country",
    ),
    value: _newCity.country ?? Country.AD,
    onChanged: (Country newSelection) {
      setState(() => _newCity.country = newSelection);
    },
    items: Country.ALL.map((Country country) {
      return DropdownMenuItem(value: country, child: Text(country.name));
    }).toList(),
  );
}
```

I’ve only highlighted the different properties on this method. It accepts a `onSaved` callback and `validator` callback also, but I’ve opted not to use them.

**Generic Form Fields**
If you want to use any other input type, such as a `Checkbox` or date picker, you can wrap any widget in a `FormField` widget. This widget exposes the same methods as the previous two form fields I’ve talked about, and basically just makes the `FormState` aware that these fields are part of the form too.

In the weather app, I used a FormField to wrap this checkbox:

![](https://dpzbhybb2pdcj.cloudfront.net/windmill/v-7/Figures/checkbox_formfield.png)

It’s implemented pretty much the same way:

```dart
bool _isDefaultFlag = false;
  Widget get _isDefaultField {
    return FormField(
      onSaved: (val) => _newCity.active = _isDefaultFlag,
      enabled: _enabled,
      builder: (context) {
        return Row(
          mainAxisAlignment: MainAxisAlignment.spaceBetween,
          children: <Widget>[
            Text("Default city?"),
            Checkbox(
              value: _isDefaultFlag,
              onChanged: (val) {
                setState(() => _isDefaultFlag = val);
              },
            ),
          ],
        );
      });
  }
```

That’s really everything there is to know about form fields functionality. Like all things in Flutter, the learning curve comes from knowing what you can do. When you figure out what that is, you’ll realize that most of the UI work is taken care of by Flutter.

#### 5.3.2  Form styling and labels
I want to briefly explain styling forms. All in all, it follows the same pattern as styling any other widget in Flutter: you wrap fields in `Padding`, `Center` and other layout widgets to work with the position, and style the widgets individually using whichever properties they expose. In particular, all input and form fields take an arguemnt called `decoration`, which you pass an `InputDecoration`.

The `InputDecoration` class accepts a whole slew of perperties that you can use to style your form field. You can set the background color, change the colors based on whether the field has focus or not, change the shape of the field, style all the text - both in the input and the helper labels - and more.

In the weather app, I opted for this outlined look. And, when you focus into the input, the label text animates to the top. It’s pretty slick:

![](https://dpzbhybb2pdcj.cloudfront.net/windmill/v-7/Figures/form_field_outline.png)

This is all done via the `decoration` property. But, incredibly, I did no work. I used another class, provided by the framework, to make all that happen. Here’s the code in the second text field:

```dart
Listing 5.8. InputDecoration example

Widget get _stateName {
  return Padding(
    padding: const EdgeInsets.only(top: 16.0),
    child: TextFormField(
      decoration: InputDecoration(
        border: OutlineInputBorder(),
        helperText: "Optional",
        labelText: "State or Territory name",
      ),
// ...
```
I guess the point is this: wow. That’s some top notch UI for free. I didn’t even have to go pull in a library of widgets. It’s in the widgets that already exist in the framework.

#### 5.3.3  Form methods help manage form state
The last piece of the user input and form puzzle is using the form to manage the form state and behavior. I’ve talked about all the pieces: fields, styling widgets, and keys, and now it’s time to tie them all together.

In particular, we care about how to respond to changes in the form, and what the app should do when the user is finished with the form. This is achieved with a couple methods on the `Form` widget and the `FormState`.

In the weather app, the best place to start is the top of the form’s build method again. Earlier in this chapter, I briefly showed the beginning of the build method, but I ommited some important lines:

```dart
Listing 5.9. Form methods

// weather_app/lib/page/add_city_page.dart -- line ~17
class _AddNewCityPageState extends State<AddNewCityPage> {
  City _newCity = new City.fromUserInput();
  bool _formChanged = false;
  final GlobalKey<FormState> _formKey = new GlobalKey<FormState>();

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
       //...
      body: Form(
          key: _formKey,
          onChanged: _onFormChange,
          onWillPop: _onWillPop,
```

**Form.onChange**
These are the only two methods that the form itself expects. First, take a look at how `onChanged` is used in the weather app:
```dart
void _onFormChange() {
  if (_formChanged) return;
    setState(() {
      _formChanged = true;
    });
}
```
The neat part about that is "Flutter knows to rerender widgets that rely on this flag for configuration." I use that flag for two things: First, telling Flutter not worry about autovalidating if the form is still blank. If `autoValidate` is on, it’s called as soon as the widget renders, which means every field would fail validation annd show an error before the user even has a chance to type anything in.
```dart
Widget get _titleField {
    return Padding(
      padding: const EdgeInsets.only(top: 16.0),
      child: TextFormField(
        decoration: InputDecoration(
          border: OutlineInputBorder(),
          helperText: "Required",
          labelText: "City name",
        ),
        autofocus: true,
        autovalidate: _formChanged,
```

The second use is more interesting to me: Button’s are disabled if their 'onPressed' callback is null. There’s no reason that the user should be able to submit something if they haven’t changed the form, so we can programmatically make the button disabled to start out.

```dart
// weather_app/lib/page/add_city_page.dart -- line ~61
child: RaisedButton(
  color: Colors.blue[400],
  child: Text("Submit"),
  onPressed: _formChanged
      ? () {
          // submit form
        }
      : null,
),
```
image::../../figures/diagrams/chapter_5/button_conditionally_enabled.png

**FormState.save**
The most important part of forms, of course, is submitting the data. The `Form` widget wraps up this process with the `FormState.save` method. In the previous code snippit, I removed the code block that’s executed when the submit button is pressed. This is the full example:
```dart
// weather_app/lib/page/add_city_page.dart -- line ~61
RaisedButton(
  color: Colors.blue[400],
  child: Text("Submit"),
  onPressed: _formChanged
      ? () {
          _formKey.currentState.save();
          _handleAddNewCity();
          Navigator.pop(context);
        }
      : null,
),
```

The magic is really in the `_formKey.currentState.save()` method. I mentioned it earlier, but this method will tell the form to find all the form fields in this portion of the apps widget tree and call `onSaved`. In the weather app particularly, that means that three methods will be called:
```dart
// weather_app/lib/page/add_city_page.dart -- line ~82
Widget get _titleField {
  return Padding(
    padding: const EdgeInsets.only(top: 16.0),
    child: TextFormField(
      onSaved: (String val) => _newCity.name = val,

// weather_app/lib/page/add_city_page.dart -- line ~103
Widget get _stateName {
  return Padding(
    padding: const EdgeInsets.only(top: 16.0),
    child: TextFormField(
      onSaved: (String val) => print(val),

// weather_app/lib/page/add_city_page.dart -- line ~140
Widget get _isDefaultField {
  return FormField(
    onSaved: (val) => _newCity.active = _isDefaultFlag,
```

That’s the core value of the form widget in a nutshell: the `onSaved` methods make it easy to work with forms that could, in theory, be way more complex.

The `_handleAddNewCity()` method isn’t Flutter specific, but in the interest of showing the full functionality, this is the method:
```dart
// weather_app/lib/page/add_city_page.dart -- line ~166
void _handleAddNewCity() {
  var city = City(
    name: _newCity.name,
    country: _newCity.country,
    active: true,
  );

  allAddedCities.add(city);
}
```

The final neat trick with forms is about being clever with the way the app behaves when the user wants to leave the form.

**Form.onWillPop**
If you’ve ever been on any form on any platform, but particularly in a mobile app, you’ve likely filled out long, complicated forms. And, nothing is worse than having to re-fill out the form because you needed to navigate away for some reason.

Flutter has a built-in way to handle situations like that via the` Form.onWillPop` method. This method gives you the chance to execute a function when the user is about to leave a form page for any reason. In the weather app, I decided to make the user confirm they want to leave the page if they started filling out the app and then tried to hit the `back` button.

![](https://dpzbhybb2pdcj.cloudfront.net/windmill/v-7/Figures/display_modal_route.png)

Another good option would be to save the form information in some way, so that you can populate the form with that information if the user came back to the page.

The code for the app is all handled in the `_onWillPop` method. It’s called from the form, here:
```dart
// weather_app/lib/page/add_city_page.dart -- line ~31
body: Container(
  padding: EdgeInsets.symmetric(horizontal: 8.0),
  child: Form(
    key: _formKey,
    onChanged: _onFormChange,
    onWillPop: _onWillPop,
```

And the part that you care about:

```dart
// weather_app/lib/page/add_city_page.dart -- line ~176
Future<bool> _onWillPop() {
    if (!_formChanged) return Future<bool>.value(true);
    return showDialog<bool>(
      context: context,
      builder: (BuildContext context) {
        return AlertDialog(
            content: Text(
                "Are you sure you want to abandon the form? Any changes will be lost."),
            actions: <Widget>[
              FlatButton(
                child: Text("Cancel"),
                onPressed: () => Navigator.pop(context, false),
                textColor: Colors.black,
              ),
              FlatButton(
                child: Text("Abandon"),
                textColor: Colors.red,
                onPressed: () => Navigator.pop(context, true),
              ),
            ],
          );
      });
  }
```

There’s a bit in this section about routing and the `Navigator`. That will be covered in depth in the future, but for now, just know a couple things:

* A dialog is a route, as far as the navigator is concerned.
* Any route can pass value back to the previous route via the `pop` method.
* 
So, the `showDialog` method is technically routing to a new "route" (showing the the dialog). Then, when the user taps the "Cancel" or "Abandon" button, it’s passing back `false` or `true` to the outer function: `_onWillPop`. This function returns a `Future<bool>`, so it’s just waiting for the return value from `showDialog`, which it will return to the `Form.onWillPop` method.

Routing is a big subject, and I don’t think you should "get it" from one paragraph. The point of all this is about the `Form.onWillPop` method. When called by the form, it’ll expect to eventually recieve a boolean. If it receives `false`, then it won’t let the navigator return the previous page. If it receives `true`, it will.

#### 5.4  Summary
User intaction in flutter is handled via two kinds of widgets: inputs and gesture detectors.
Flutter handles gestures and user interaction events via `GestureDetector` widgets.
The gesture detector can listen for many gestures via it’s various callbacks. This is only 5 of ~30:
* onTap
* onLongPress
* onDoubleTap
* onVerticalDragDown
* onPanDown
* Several built in widgets listen these gestures as well: `Dismissible`, `Button`, `FormField`, and many more.
* Flutter forms are convenient wrappers around several input widgets that make managing complex forms easier.
* Form’s state can be managed with a `GlobalKey<FormState>`, which is a reference to a `FormState` object itself.
* Form’s are aware of widgets below them in the widget tree that are wrapped in `FormField` widgets, and can take advantage of this relationship.
* FormFields provide many methods, namely: `onChange`, `onSave`, and `validator`
* Form’s themselves have two methods that are quite valuable: `onChange` and `onWillPop`