> In this chapter:
>* Dart’s Hello, World!
>* Basic Dart syntax
>* Lexical Scope
>* Using Dart libraries

This book is about building mobile apps with Flutter. But you can’t write a Flutter app with learning a bit about the Dart programming language, first. Aside from being my favorite language, Dart is also quite easy to pick up. If you’re comfortable enough with Dart already that you understand the following code block, you can skip this chapter.

```dart
class CompanyContext extends StateObject {
  final bool isLoading;
  String _companyId;

  CompanyContext({
    this.isLoading = false,
  });

  String get companyId => _companyId;
  void set companyId(String id) {
    _companyId = id;
    _initApp();
  }

  factory CompanyContext.loading() => new CompanyContext(isLoading: true);

  @override
  String toString() => 'CompanyContext{isLoading: $isLoading, currentUser: $currentUser}';
}
```

> **THE COMMAND LINE**
> Many instructions in this book will involve running commands in your machines terminal. I’m a big fan of GUI’s, and don’t use the command line much. You don’t need to be a command line wizard to use this book. Just know that anytime you see a line of code that starts with a `$`, it’s a command for your terminal. The following `⇒` shows the return value (if any). For example, the command `which dart` in the OSX terminal returns the file path to your Dart SDK.
```batch
$ which dart
=> /usr/local/bin/dart
```
#### 2.1  Hello, Dart!
Like all good, cliche programming books, we’re going to start with a program that prints 'Hello, World' (kind of) to your console. It doesn’t matter where your Dart projects live in your computer, but I recommend that you create a directory on your desktop for this book.

Mac and Linux:
```
$ cd ~/Desktop
$ mkdir flutter_in_action
$ cd flutter_in_action
$ mkdir hello_world
$ cd hello_world
```
Windows CMD:
```
> mkdir "%USERPROFILE%\flutter_in_action"
> cd /d "%USERPROFILE%\flutter_in_action"
> mkdir hello_world
> cd hello_world
```

>NOTE
For the rest of this book, I’ll only provide the OSX command. They’re similar enough, and the `$` is ubiquitous.

In your favorite text editor, create a new file in the `hello_world` directory called `hello_world.dart`. Write this code in the file:

```dart
void main() {
  print('Hello, Dart!');
}
```
Now, back in your terminal, you can execute Dart code with the CLI command `dart`. To follow along with these instructions, make sure that you’re in the right project directory where your `hello_world.dart` file lives.

```
$ dart hello_world.dart
// => Hello, Dart!
```

If `Hello, Dart!` did in fact print, congrats! You wrote your first Dart program, and you’re officially a Dart programmer!

If the program didn’t run, double check that you have Dart installed on your machine. Installation instructions are in the appendix.

#### 2.1.1  Anatomy of a Dart Program
Dart programs of all shapes and sizes have a few things in common that must exist. Most importantly, a main function in the entry file of your program.

This is the first piece of the puzzle. This is a Dart function definition:

**Figure 2.1. The main function in Dart**
![](https://dpzbhybb2pdcj.cloudfront.net/windmill/v-7/Figures/anatomy_of_dart_function.png)

> “ll Dart functions look like this, but `main` is special. It's always the first code that's executed in a Dart program. You program must contain a `main` function that the Dart compiler can find.”

Notice the word 'void' in the example. `void` is the return type of this function. If you haven’t used typed languages, this may look strange. Types are a core part of what makes Dart great. Every variable should have a type, and every function should return a type (or `void`). `void` is a keyword that means 'this function doesn’t return anything'. We’ll dive deeper into the type system when we have more robust examples to walk through. But for now, just remember: All functions return a type (or 'void').

Next in this example is the line that contains the print function.
```dart
print('Hello, Dart!');
```

`print` is a special function in Dart that prints text to the console. (JavaScript developers who don’t believe in debuggers should write that down.)

On that same line, you have a `String` ('Hello, Dart!'), and you have the semicolon (;) at the end of the line. This is necessary at the end of every expression in Dart.

That’s all you need to call your program a Dart program.

#### 2.2  Programming a Greeter: Hello, _!
This section will introduce the basics of setting up and running a Dart program. It will also introduce some common Dart syntax. Finally, we’ll use the dart:io package to learn about importing libraries, and basic input and output.

#### 2.2.1  Add more greetings
I want to expand on the greeter example so I can talk about some of Dart’s basic syntax. In general, a lot of Dart syntax is similar to many languages. Control flow, loops and primitive types are as you’d expect if you come from almost any language.

Let’s write a program that will output this to the console:
```
Hello, World!
Hello, Mars!
Hello, Oregon!
Hello, Barry!
Hello, David Bowie!
```

This may feel contrived, but there’s a lot to learn in simple examples. To start, refactor the old example to create a separate function that prints. Your file will look like this:
```dart
void main() {
  helloWorld();
}

void helloWorld() {
  print("Hello, World!");
}
```

Next, the `helloWorld` function needs to be told what to print, because you don’t want it to print "Hello, World" forever. You’ll do this by passing in a name to replace "World" with. You pass in arguments to functions by putting a type and variable name in the `()` in the function signature.
```dart
void helloWorld(String name) {
  print("Hello, World!");
}
```

I don’t just want to print one name though, I want to print a sequence of names. The names will come from a hard-coded `List`, Dart’s basic array-like data structure. A `List` manages it’s own size and provides all the functional programming methods that you expect on an array, like `map` and `forEach`.

For now, you can create a list with the list-literal constructor, using the square brackets: `var myList = [a,b,c]`. Add a list of names to your `main` function:
```dart
void main() {
  List<String> greetings = ['World', 'Mars', 'Oregon', 'Barry', 'David Bowie'];   #1
  helloWorld();
}
```
Right now, there’s an error in the program. This sample calling `helloWorld()` without a string as an argument. We need to be passing each of those individual greetings into the call to `helloWorld()`. To do so, loop over the `greetings` variable and call the `helloWorld()` function inside every iteration of the loop.
```dart
void main() {
  List<String> greetings = ['World', 'Mars', 'Oregon', 'Barry', 'David Bowie'];   #1
  for (var name in greetings) {
    helloWorld(name);
  }
}
```
Finally, update the `helloWorld` method to print "Hello," followed by the specific greeting. This is done with interpolation. String interpolation in Dart is done with the `${}` syntax, or `$` if the symbol is a single value, rather than a chained call. Here is the full example:
```dart
void main() {
  List<String> greetings = ['World', 'Mars', 'Oregon', 'Barry', 'David Bowie'];   #1
  for (var name in greetings) {
    helloWorld(name);
  }
}

void helloWorld(String name) {
  print("Hello, $name");
}
```
That’s all it takes. This should work, now.

#### 2.2.2  I/O and Dart Libraries
The final feature in this example is interacting with a user. The user will be able to say who they want to greet.

The first step is importing libraries. The Dart SDK includes many libraries, but the only one that’s loaded in your program by default is `dart:core`. Some common libraries in the Dart SDK are `dart:html`, `dart:async`, and `dart:math`. Different libraries are available in different environments. For example, `dart:html` isn’t included in the Flutter SDK, because there’s no concept of HTML in a Flutter app. When writing a server-side or command line application, you’ll probably use `dart:io`. Let’s start there.

To import any library, you only need to add an `import` statement at the top of a dart file.
```dart
import `dart:io`;
```
We won’t use standard input and outputs in this book much, if at all after this, so don’t get bogged down in the details of the io library right now. The program will look like this now:
```dart
import 'dart:io';

void main() {
  stdout.writeln('Greet somebody');
  String input = stdin.readLineSync();
  return helloWorld(input);
}

void helloWorld(String name) {
  print('Hello, $name');
}
```

This program asks for a name in the command line from the user, and then greets that person. You could improve this program by looping over everything and repeatedly asking for a name, or make it a number guessing game that exits when you guess the write number. I’ll leave that for you to do on your own.

#### 2.3  Common Programming Concepts In Dart
If all programming languages can be described in terms of the Beatles catalogue, Dart is like The Beatles greatest hits. Everyone loves the Beatles. Because the Beatles are great. And everyone knows 'Hey Jude'. But, when listening to Beatle’s greatest hits at a fun, upbeat party, you’re never worried that you’re going to suddenly be listening to the song 'Within Without You'. I love that song, but it’s not for everyone. And it’s certainly not a party song. When writing Dart code, you’re never scared that you’re going to run into some whacky, unexplainable syntax or behavior. It’s all expected, and easy to decipher. (Not to mention a real crowd pleaser.)

There are a few important concepts you should keep in mind while writing Dart code:

* Dart is and object oriented language, and supports single inheritance.
* In Dart, everything is an object. And every object is an instance of a class. Every object inherits from the `Object` class.
* Dart is typed. You cannot return a number from a function that declares it returns a string.
* Dart supports top level functions and variables, often referred to as *library members*.
* Dart is *lexically* scoped.

And, Dart is quite opinionated. (Which is great, in my opinion.) In Dart, as in all programming languages, there are different ways of getting things done. But some ways are right and some are wrong. This quote from the Dart website sums it up: "Guidelines describe practices that should always be followed. There will almost never be a valid reason to stray from them."

>Just in Time: Typed programming languages
A language is 'typed' if every variable’s type is known (or inferred) at compile-time. In human English, a language is 'typed' if you, as the developer, can (or must) explicitly assign types to variables in your code. A language is 'dynamic' if the types are inferred at runtime. JavaScript, Python, and Ruby are dynamic languages. Under the hood, though, all languages are typed to some degree.
>
>Types are used because they make your code safer. Your compiler won’t let you pass a string to a function that expects a number. Importantly, this type check is done at compile-time. This means that you’ll never ship code that crashes because a function doesn’t know what to do with a different type of data than it expects. Type systems reduce bugs.
>
>One of the best one-sentence lessons I got from a boss when I started using a type system: "Don’t fight the compiler".

#### 2.3.1  Intro to Dart’s Type System
The type system in Dart will be something I’ll discuss throughout the book. The type system is straightforward (as far as type systems go). That said, it has to be briefly examined before I can talk about anything else.

Before I became a Dart developer, I wrote Ruby, Python and JavaScript, which are dynamic. They have no concept of types (to the developer). When I started writing Dart, I found using types to be the biggest hurdle. (But now, I don’t want to live in a world without them.)

There are a few key places that you need to know about types for now, and the rest will be covered in time. First, when declaring variables, you give them a type:
```dart
String name;
int age;
```

The type always comes before the value that it describes.
```dart
int greeting = 'hello';
```
If you try to compile that file by running it in your terminal, you’ll get this error:
```
Error: A value of type 'dart.core::String' can't be assigned to a variable of type 'dart.core::int'.
Try changing the type of the left hand side, or casting the right hand side to 'dart.core::int'.
```
First, that’s a pretty darn good error message, as error messages tend to be in Dart. (Thanks, Dart team.) But also, this is what’s called type safe. Types ensure at compile time that your functions are all getting the right kind of data. This greatly reduces the number of bugs you’ll get at run time.

>TIP
If you’re using one of the IDE’s suggested in the appendix and have installed the the Dart plugin, you won’t even get that far. The linter will tell you you’re using the wrong type straight away. This is, in a nutshell, the value of type systems.

**Complex data types**
When using data structures like a `List` or `Map`, you use `<` and `>` to define the types of the values within the List.

```dart
List<String> names;
List<int> ages;
Map<String, int> people;
```
**Types in functions**
Recall from earlier that the `main` function has a return type of `void`. Any function that’s being used for it side-effects should have this return type. It means that the function returns nothing.

Other functions, though, should declare what they’re going to return.
```dart
int addNums() {
  // return an int
}
```
The second place you use types in functions is when you define your arguments:
```dart
int addNums(int x, int y) {
  return x + y;
}
```

**Dynamic Types**
Dart supports dynamic types as well. When you set a variable as `dynamic`, you’re telling the compiler to accept any type for that variable.
```dart
dynamic myNumber = "Hello";
```

Technically, you could just mark everything as `dynamic` and it’d be the wild west. And to that, I’d say 'Good luck!' `dynamic` comes in handy though. It’s pretty common to use `dynamic` in maps. Perhaps you’re working with JSON:
```dart
Map<String, dynamic> json;
```

If you’re converting some JSON to a Dart object, you know the 'keys' of the Map are going to be Strings, but the values could be strings, numbers, Lists, or another Map.

#### 2.3.2  Comments
Dart supports three kinds of comments:
```
// In-line comments

/*
Blocks of comments. It's not convention to use block comments in Dart.
*/

/// Documentation
///
/// This is what you should use to document your classes.
```
**2.3.3  Variables and Assignment**
Variables are used in Dart to tell an object or class to hold onto some local state. Establishing a variable in Dart is as you’d expect. This is a variable definition:
```dart
String name;
```
That line simply tells your program to make some space in memory for some to-be-determined value (in this case, a `String`).

At this point, `name` hasn’t been assigned to a value, so it’s value `null`. All un-assigned variables in Dart are `null`. `null` is a special value that means 'nothing'. In Dart, `null` is an object, like everything else. Thats why `int` s `String` s `List` s and everything else can be assigned to `null`. Technically, you could do this:
```dart
int three = null;
```

**final and const**
These two key words 'extend' the type of the variable, in a way. You should use these two keywords if you never intend to change a variable. The difference in the two is subtle.

`final` variables can only be assigned once. However, they can be declared before they’re set at the class level. Or, in human English, a `final` variable is almost always a variable on a class that will assigned in the constructor. If that seemed like a bunch of jargon, don’t worry, I’m really going to bore you to death about classes in a bit. You’ll be begging to be confused.

`const` variables, on the other hand, wouldn’t be declared before they’re assigned. Constants are variables that are always the same, no matter what, starting at compile time.

Acceptable:
```dart
const String name = 'Nora';
```
Not Acceptable:
```dart
const String name = 'Nora $lastName';
```

That variable in the second example means that the variable could change after compile-time, and therefor is not allowed.

#### 2.3.4  Operators
There aren’t any big surprises in Dart operators.

Table 2.1. Dart Operators

| Description              | Operators |
|--------------------------|-----------|
| arithmetic               |* / % ~/ + -|
| relational and type test | >= > <= < as is is! |
| equality                 | == !=  |
| logical and / or         | && \|\| |
| assignment               | = *= /= ~/= %= += -= <<= >>= &= ^= |= ??= |
| unary                    | expr expr-- . ?. -expr !expr ~expr expr --expr |

#### 2.3.5  Null Aware Operators
Null-aware operators are one of my favorite features in Dart. In any language, having variables and values fly around that are `null` is bad. It can crash your program. Programmers often have to write `if (response == null)` return in the top of a function that makes asynchronous calls. That’s not the worst thing ever, but it’s not concise. At my job we use Go-lang quite a bit, which isn’t a robust language. (That’s not a judgement, it’s a statement of fact.) About once every 10 lines of code there’s an `if` statement checking for `nil`. This makes for some robust code.

Null-aware operators in Dart help resolve this issue. They’re basically ways to say "If this object or value is `null`, then forget about it."

The number one rule of writing Dart code is to be concise, but not pithy. Anytime you can write less code without sacrificing readability, you probably should. You can use these three operators to achieve that.

**?.**
Suppose you want to call an API and get some information about a `User`. And maybe you’re not sure if the users information you want to fetch even exists. You may do something like this:
```dart
void getUserAge(String username) async {
  var request = await new UserRequest(username);
  var response = parseResponse(request);
  var user = new User.fromResponse(response);
  if (user != null) {
    this.userAge = user.age;
  }
  // etc
  // etc
}
```

That’s fine. And that works. But the null-aware operators make it much easier. This operator will basically say 'Hey, do the operation on this object, unless the object is null. If it is, don’t worry about, just move on.'
```dart
void getUserAge(String username) async {
  var request = await new UserRequest(username);
  var response = parseResponse(request);
  var user = new User.fromResponse(response);
  this.userAge = user?.age;
  // etc
  // etc
}
```
If `user` is indeed null, then your program won’t try to make that assignment, and everything will be fine. Your code is more concise and still readable. That’s the key. These operators (and Dart in general) isn’t about doing some whacky, single-line code golf. Just clean, concise code.

**??**
The second null-aware operator is perhaps even more useful. Suppose you want the same `User` information, but many fields on the user aren’t required in your database. There’s no guarantee that there will be an age on that user. Then you can use the double-question-mark (`??`).

This operator says 'Hey program, do this operation with this value or variable. But if that value or variable is null, then use this backup value.' It allows you to assign a default value at any given point in your process, and it’s super handy.
```dart
void getUserAge(String username) async {
  var request = await new UserRequest(username);
  var response = parseResponse(request);
  var user = new User.fromResponse(response);
  this.userAge = user.age ?? 18;
  // etc
  // etc
}
```
**??=**
This last one accomplishes a pretty similar goal as the last one, but opposite. While writing this, (right now for me, but the past for you), I was thinking about how I never use this operator in real life. So I decided to do a little research. And, wouldn’t you know it? I should be using it. It’s great.

This operator basically says 'Hey, if this object is null, then assign it to this value. If it’s not, just return the object as is.'
```dart
int x = 5
x ??= 3;
```
In the second line there, x will not be assigned 3, because it already has a value. But like the other null-aware operators, this one seeks to make your code more concise.

#### 2.4  Control Flow
The strangest thing about technology is that we treat computers like they’re smart but they’re actually real dumb. They only know how to do roughly 2 or 3 things. You can expect a human, or maybe even a dog, to react appropriately to any given number of situations. Dogs know that if they’re hungry, they need to eat to survive. And they know what’s food and what isn’t. They aren’t going to accidentally eat a rock and hope it works out.

Computers aren’t as nice to work with. You have to tell them everything. They’re quite needy, actually. So we have to take great pains and measures to ensure that no matter what situation arises, they know how to handle it. This is basically why we have control flow, which is the basis for pretty much all logic.

Control flow in Dart is roughly exactly like it is in pretty much all the high level languages. You get if statements, ternary operations, and switch statements.

#### 2.4.1  if and else
Dart supports `if`, `else if` and `else`, as you’d expect.
```dart
if (inPortland) {
  print('Bring an umbrella!');
} else {
  print('Check the weather, first!');
}
```
Inside your conditions, you can use `&&` as and, and `||` as or.
```dart
if (inPortland && isSummer) {
  print('The weather is amazing!');
} else if(inPortland && isAnyOtherSeason) {
  print('Torrential downpour.');
} else {
  print ('Check the weather!');
}
```

Finally, Dart is sane and conditions must evaluate to a boolean. There is only one way to say true: `true` and one way to say false: `false`.

#### 2.4.2  switch and case
Switch statements are great when there are many possible conditions. Switch statements conditions compare ints, Strings, and compile-time constants using `==`. In other words, you must compare a value to a value of the same type that cannot change at run-time. This used to confuse me. Here’s a simple example:
```dart
var number = 1;
switch(number) {
  case 0:
    print('one!');
    break;
  case 1:
    print('one!');
    break;
  case 2:
    print('two!;);
    break
  default:
    print('choose a different number!');
}
```

That’s perfectly valid. However, this wouldn’t be:
```dart
var five = 5;
  switch(five) {
      case(five < 10):
      // do things...
  }
```
At compile time, you can’t guarantee the value of `five`. It could change in the code. It isn’t constant. `five < 10` returns a boolean.

In switch statements, you can fall through multiple cases by not adding a `break` or `return` statement at the end of a case:
```dart
var number = 1;
switch(number) {
  case -1:
  case -2:
  case -3:
  case -4:
  case -5:
    print('negative!');
    break;
  case 1:
  case 2:
  case 3:
  case 4:
  case 5:
    print('positive!;);
    break
  case 0:
  default:
    print('zero!');
    break;
}
```

Each case in a switch statement must end with a keyword that exits the switch. If you don’t, you’ll get an error.
```dart
switch(number) {
  case 1:
    print(number)
    // ERROR!
  case 2:
  //...
```
Most commonly you’ll use `break` or `return`. `break` simply breaks out of the switch, but doesn’t have any other effect.

In addition to those, you can use the `throw` keyword, which throws an error. (More on throw in a bit.) Finally, you can use a `continue` statement and a label if you want to fall-through, but still have logic in every case.
```dart
var animal = 'tiger';
switch(animal) {
  case 'tiger':
    print('it's a tiger');
    continue alsoCat;
  case 'lion':
    print('it's a lion');
    continue alsoCat;
  alsoCat:
  case 'cat':
    print('it's a cat');
    break;
  // ...
}
```

That switch statement would print `it’s a tiger` and `it’s a cat` to the console. I’ve literally never used that, but I’m going to start now. I’m into it.

**Ternary Operator**
The ternary operator is technically that: an operator. But, it’s also kind of an `if/else` substitute. It’s also kind of a `??=` alternative, 
depending on the situation. I use ternaries in Flutter widgets quite a bit. The ternary expression is used to conditionally assign a value. And, it’s called a ternary because it has three portions: the condition, the value if the condition is true, and the value if the condition is false.

**Figure 2.2. Dart ternary operator**
![](https://dpzbhybb2pdcj.cloudfront.net/windmill/v-7/Figures/ternary_op_hedgehog.png)

#### 2.4.3  Loops
You can repeat expressions in loops, just like you’d expect. There a couple different kinds of loops in Dart:

* standard `for`
* `for-in`
* `forEach`
* `while`
* `do while`

Each of these work exactly how they do in literally every programming language that I’ve come across. So I’ll just provide some quick examples.

**For Loops**
If you need to know the index, your best bet is the standard for loop:
```dart
Listing 2.1. Standard for loop

for (var i = 0; i < 5; i++) {
  print(i);
}
```

If you don’t care about the index, the `for-in` loop is great option.
```dart
Listing 2.2. for-in loop

var pets = ['Odyn', 'Buck', 'Yeti'];
for (var pet in pets) {
  print(pet);
}
```
An alternative, and probably the preferred way to loop if you don’t care about the index is using the method on iterables called forEach.

```dart
Listing 2.3. forEach

var pets = ['Abe', 'Buck', 'Yeti'];
pets.forEach((pet) => pet.bark());
```

**While Loops**
Again, while loops behave exactly how you’d expect. While loops will evaluate the condition before the loop runs. Meaning it may never run at all.
```dart
Listing 2.4. while loop

while(someConditionIsTrue) {
  // do some things
}
```
`do-while` loops, on the other hand, evaluate the condition after the loop runs. So it will always execute the code in the block at least once.
```dart
Listing 2.5. do-while loop

do {
  // do somethings at least once
} while(thisConditionIsTrue());
```

**Break and Continue**
These two key words help you manipulate the flow of the loop. Use `continue` in a loop to immediately jump to the next iteration. Use break to `break` out of the loop completely.

```dart
for (var i = 0; i < 55; i++) {
  if (i == 5) {
    continue;
  }
  if (i == 10) {
    break;
  }
  print(i);
}
```

This loop would print:
```
0
1
2
3
4
6
7
8
9
```

#### 2.5  Functions
Functions look familiar in Dart if you’re coming from any C-like language. We’ve already seen a couple examples of this, the `main` function:
```dart
Listing 2.6. A Basic Function

void main() {

}
```
#### 2.5.1  Anatomy of a Dart Function
The anatomy of a function is pretty straight forward:
```dart
Listing 2.7. Anatomy of a Dart Function

String makeGreeting(String name) {
  return 'Hello, $name`;
}
```

It’s important to note that Dart is a true Object-Oriented language. Even functions are objects, with the type Function. You can pass functions around and assign them to variables and all sorts of fun things. Languages that support passing functions as arguments and returning functions from functions usually refer to these as higher-order functions.

We’ll explore higher-order functions in depth when we start writing Flutter apps.

Dart also supports a nice shorthand syntax for any function that has only one expression. In other words, is the code inside the function block only one line? It’s probably one expression and you can use this syntax to be concise:
```dart
String makeGreeting(String name) => 'Hello, $name';
```

In this book, we’ll call this an 'arrow function'.

Arrow functions implicitly return the result of the expression. `⇒ expression`; is essentially the same as `{ return expression; }`. Theres no need (and you can’t) include the `return` keyword.

#### 2.5.2  Parameters
Dart functions allow positional parameters, named parameters, and optional positional and named parameters. Or, a combination of all of them. Positional parameters are simply what we’ve seen so far.
```dart
Listing 2.8. Two Positional parameters

void debugger(String message, int lineNum) {
  // ...
}
```
To call that function, you must pass in a string and int, in that order.
```dart
debugger('A bug!', 55);
```
**Named Parameters**
Dart supports named parameters. Named means that when you call the function, you attach the argument to label:
```dart
Listing 2.9. Calling a function with two named parameters

debugger(message: 'A bug!', lineNum: 44);
```
Named parameters are written a bit differently. You wrap any named parameters in curly braces (`{ }`).
```dart
Listing 2.10. Defining a function with named parameters

void debugger({String message, int lineNum}) {
```

Named parameters are, by default, are optional. But, you can annotate them and make them required.
```dart
Listing 2.11. Required named parameters

Widget build({@required Widget child}) {
  //...
}
```
The pattern you see above is going to become very familiar to you when we start writing Flutter apps.

**Positional optional parameters**
Finally, you can pass positional parameters that are optional using `[ ]`. I rarely, if ever, use this. Might as well make your code readable with named params.
```dart
int addSomeNums(int x, int y, [int z]) {
  var sum = x + y;
  if (z != null) {
    sum += z;
  }
  return sum;
}
```

You’d call that function like this:
```dart
addSomeNums(5, 4)
addSomeNums(5, 4, 3)
```

#### 2.5.3  Default parameter values
You can define default values for parameters with the = operator in the function signature.
```dart
addSomeNums(int x, int y, [int z = 5]) => x + y + z;
```

#### 2.5.4  Advanced Function Concepts
Functions are the bread and butter of dry code because they let us define our own vocabulary in our programs. In a robust app, there are likely thousands of lines of code. It’s easy to get lost. When used correctly, higher-order functions help add a layer of abstraction to our code that makes it easy to reason about. Consider these two examples that do math:
```dart
var nums = [1,2,3,4,5];

var i = 0;
var sum = 0;
while (i < nums.length) {
  sum += nums[i];
  i+=1;
}
print(sum);
```
```dart
var nums = [1,2,3,4,5];

print((addNumbers(nums));
```

It’s possible that the `addNumbers` function, not shown here, is implemented exactly the way that the first example adds the numbers together. But, the second example adds a nice layer of abstraction, and tells us exactly what it’s doing. You don’t have to read each line of code to understand whats happening. And as a bonus, we know that if the `addNumbers` function is bug free, then it’ll remain bug free every time we use it in our app. This is a simple example, of course, but breaking up functions into single-responsibility chunks of logic make them much easier to get right.

Creating your own vocabulary by breaking up functions is called 'abstraction'. Remember, computers are dumb. We have to tell them exactly what we want them to do. But humans are smart. We use abstraction to write low-level, explicit instructions for the computer, and then wrap it up in nice little functions for future programmers that will be reading our code.

In Dart, abstracting away logic is possible because it supports these higher-order functions. A function is higher-order if it accepts a function as an argument, or if it returns a function. In other words, it’s functions operating on other functions.

If you aren’t sure about higher-order functions, you’ve likely seen them before in a different language.
```dart
var nums = [1,2,3];
nums.forEach((number) => print(num + 1));
```

`forEach` is a higher-order function because it takes a function as it’s argument. Another way to write that would be like this:
void addOneAndPrint(int num) {
  print(num +1);
}

nums.forEach(addOneAndPrint);

>NOTE
The first example using forEach uses an anonymous function. Which only means that it doesn’t have a name. It’s defined right there in the argument to forEach, and after it’s executed it’s gone forever.

Earlier I mentioned that functions are just objects, like everything in Dart. So, it isn’t particularly surprising that you can use functions like you can anything else.

In reality, you can get away without using higher-order functions very often, except when processing `Iterables` (such as `Lists`). So functions like `forEach`, `map`, `where`, etc. We’ll get into those later on in the book.

Even though you can get away without using too many higher-order functions, you might find that they’re really useful when you want to write clean, elegant code.

#### 2.5.5  Lexical Scope
Finally, Dart is lexically scoped. Every code block has access to variables "above" it. The scope is defined by the structure of the code, and you can see what variables are in the current scope by following the curly braces outward to the top-level.

```dart
String topLevel = 'Hello';

void firstFunction() {
  String secondLevel = 'Hi';
  print(topLevel);
  nestedFunction() {
    String thirdLevel = 'Howdy';
    print(topLevel);
    print(secondLevel);
    innerNestedFunction() {
      print(topLevel);
      print(secondLevel);
      print(thirdLevel);
    }
  }
  print(thirdLevel)
}
```

These all work, until that last line. The third level variable is defined outside the scope of the nested function, because scope 'flows one-way': inward.

#### 2.6  Object Oriented Programming (in Dart)
Modern applications basically all do the same thing. They give us (smart humans) a way to process and collaborate overlarge data sets. Some apps are about communication, like social media and email. Some are about organization, such as calendars and note taking. Some are simply a digital interface into a part of the real world that’s hard to navigate for programmers. Like dating apps. But, they all do the same thing. They give users a nice way to interact with data.

And data represents the real world. Literally all data describes something real. That’s what object oriented programming is all about. It gives use a nice way to model our data after real world objects. It takes all this data, which dumb computers like, and adds some abstraction so smart humans can impose our will onto these computers. It makes code easy to read, easy to reason about, and highly reusable.

When writing Dart code, you’ll likely want to create separate classes for everything that can represent a real-world "thing". "Thing" is a carefully chosen word, because it’s so vague. (This is a great example of something that would make a dumb computer explode, but a smart human can make some sense of.)

Consider you were writing a point-of-sale (POS) system used to sell goods to customers. What kind of classes do you think you’d need to represent our "things" (or data)? What kind of "things" does a POS app need to about? Perhaps we need classes to represent a `Customer`, `Business`, `Employee`, `Product`, and `Money`. Those are all classes that would indeed represent real-world 'things'. But it gets a bit hairier from here.

Ponder some questions with me:
* We may want a class for `Transaction` and `Sale`. In real life, a transaction is a process, or an event. Should this be represented with a function or a class?
* Also, if we’re selling bananas, should we use `Product` class and give it a property that describes what type of product it is? Or, should we have a `Banana` class?
* Should you define a top-level variable or a class that only has a single property? For instance, if you need to write a function that simply adds two numbers together, should you define a `Math` class with an `add` method on it, or just write your method as a static, global variable?

Ultimately, these decisions are up to you. This is what we, as programmers, have to decide. There is no right answer.

#### 2.6.1  Classes
My rule of thumb is "When in doubt, make a new class." Recall those previous questions: Should a transaction be represented by a function on Business, or its own class? I’d say make it a class. Which brings me all the way back to why I used the vague word 'thing' earlier. A thing isn’t just a physical object, it can be an idea, an event, a logical grouping of adjectives, etc. In this example, I would make a class that looks like this:
```dart
class TransactionEvent {
  // properties and methods
}
```
And that may be it. It might have no properties and no methods. Creating classes for events makes the type safety of Dart that much more effective.

The bottom line is that you can (and I’d argue, should) make a class that represents any 'thing'. Anything that isn’t obviously an action that you can do or a word you’d use to describe some detail of a 'thing'. For instance, you (a human) can exchange money with someone. It makes sense to say 'I exchange money'. It doesn’t make sense to say 'I transaction', even though a transaction is an idea. Having a `Transaction` class makes sense, but an `ExchangeMoney` class doesn’t.

#### 2.6.2  Constructors
You can give classes special instructions on what to do as soon as they’re created. These functions are called `Constructors`.

Often when creating a class, you’ll want to pass some values to it. You can use the constructor to assign those values to properties on an instance of that class.
```dart
class Animal {
  String name;
  String type;

  Animal(String name, String type) {
    this.name = name;
    this.type = type;
  }
}
```
You can achieve the same thing with the following syntactic sugar:
```dart
class Animal {
  String name, type;

  Animal(this.name, this.type);
}
```
You can put whatever code and logic you want in a constructor, it’s just a plain ol' function:
```dart
class Animal {
  String name, type;

  Animal(this.name, this.type) {
    print('Hello from Animal!');
  }
}
```

#### 2.6.3  Inheritance
In object-oriented programming, inheritance is the idea that a class can 'inherit' or 'subclass' a different class. A cat is a specific kind of mammal, so it follows that a cat will have all the same functionality and properties as a mammal.
```dart
class Cat extends Mammal {}
class Eric extends Human {}
class Honda extends Car {}
```

When a class 'inherits' from another class (called its super-class), it’s essentially a copy of the super-class with extra functionality - whatever you define in the class itself. That’s a ton of jargon. Let’s look at a concrete, but small example:
```dart
// super-class
class Animal {
  String name;
  int legCount;
}

// sub-class
class Cat extends Animal {
  String makeNoise() {
    print('purrrrrrr');
  }
}
```
In this example, if we made an instance of a cat, then it would have properties called `name` and `legCount`.
```dart
var cat = new Cat();
cat.name = 'Nora';
cat.legCount = 4;
cat.makeNoise();
```

Those are all perfectly valid expressions. You can set the cats name, because it’s also an 'Animal'. This is not valid, however:

var cat = new Animal();
cat.makeNoise();

`Animal` is the super-class, and has no concept or relationship to any of it’s subclasses that extend it.

To expand on inheritance, consider if we made a class that’s almost exactly the same, but for a Pig.
```dart
class Pig extends Animal {
  String makeNoise() {
    print('oink');
  }
}
```
It’s now perfectly valid to do this:
```dart
var pig = new Pig();
pig.name = 'Babe';
pig.legCount = 4;
pig.makeNoise();
```

Since pig extends animal, like cat, it has a 'name' property and a 'legCount' property.

Finally, inheritance is like a tree. If Pig inherits from Mammal which inherits from Animal which inherits from Life, then Pig has access to all the members on all those classes.

Every object in Dart inherits, eventually, from `Object`
![](https://dpzbhybb2pdcj.cloudfront.net/windmill/v-7/Figures/OOP_Inheritance.png)

#### 2.6.4  Factories and Named Constructors
Right now, one of the great challenges that humans are facing is creating renewable energy. There’s a common thread between all the possible sources of energy: it all ends up as 'energy' in the end. But theres a brief moment there when the wind is just wind, not yet turned into energy. And sunbeams are just sunbeams. And science needs to know how to turn these different substances into energy.

>TIP
I don’t know anything about physical science. Please go easy on me.

This is what `factory` and named constructors do. They’re static methods on classes that create an instance of that class. Named constructors always return a new instance of a class. `factory` methods have a bit more flexibility. They can return cached instances or instances that are subtypes.

In code, that energy class might look like this:
```dart
class Energy {
  int joules;

  Energy(this.joules);

  Energy.fromWind(int windBlows) {
    var joules = _convertWindToEnergy(windBlows);
    return new Energy(joules);
  }

  factory Energy.fromSolar(int sunbeams) {
    if (appState.solarEnergy != null) return solarEnergy;
    var joules = _convertSunbeamsToJoules(sunbeams);
    return appState.solarEnergy = new Energy(joules);
  }
}
```

#### 2.6.5  Enumerators
Enumerators, often called enums, are fancy classes that represent a specific number of constants. Suppose you have a method that takes a String, and then does some magic and changes the color of some text in an app. That function may look like this:
```dart
void updateColor(String color) {
  if (color == 'red') {
    text.style.color = 'rgb(255,0,0);
  } else if (color == 'blue') {
    text.style.color = 'rgb(0,0,255);
  }
}
```

This is great, unless you pass 'macaroni', 'crab cakes', '33445533', or literally any other string into `updateColor`.

You can use an enum to buy yourself some type safety, without the verbosity of a class. At the end of the day, that’s what an enum is about. It makes your code harder to break and easier to read.

So your Color enum can look like this:
```dart
enum Color { red, blue }
```
In your code you can get to colors like this: `Color.red;`

Variables and fields can now have Color as a type. And as an added bonus, `switch` statements can switch on an enum, and demand that you have a `case` statement for every type in the enum (or a default at the end).

Now, your function can look like this:
```dart
enum Color { red, green, blue }

void updateColor(Color color) {
  switch(color) {
    case(Color.red):
      // do stuff
    case(Color.green):
      // do stuff
    case(Color.blue):
      // do stuff

  }
}
```
#### 2.7  Summary
* Dart’s syntax is familiar if you know any C-like language.
* pub is the package manager for Dart
* Dart is object oriented, strictly typed language.
* Types are helpful and awesome.
* Dart is a great language. Probably the best one out there.
