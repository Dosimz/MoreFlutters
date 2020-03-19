#### 这个仓库用来存储每天抽空在 Flutter 官网的实践，以便后面自己查阅。

> 以下内容来自  https://flutter.dev/docs/development/ui/widgets-intro


### Day Seven

![basicWidget](https://github.com/Dosimz/MoreFlutters/blob/master/introductionTowidget/widgetset/outimages/non-materialApp.jpeg)


##### Material App

```dart
class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Flutter layout demo',
      home: Scaffold(
        appBar: AppBar(
          title: Text('Flutter layout demo'),
        ),
        body: Center(
          child: Text('Hello World'),
        ),
      ),
    );
  }
}
```
> For a Material app, you can use a Scaffold widget; it provides a default banner, background color, and has API for adding drawers, snack bars, and bottom sheets. Then you can add the Center widget directly to the body property for the home page.

##### Non-Material App

> For a non-Material app, you can add the Center widget to the app’s build() method:

```dart
class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Container(
      decoration: BoxDecoration(color: Colors.white),
      child: Center(
        child: Text(
          'Hello World',
          textDirection: TextDirection.ltr,
          style: TextStyle(
            fontSize: 32,
            color: Colors.black87,
          ),
        ),
      ),
    );
  }
}
```
> By default a non-Material app doesn’t include an AppBar, title, or background color. If you want these features in a non-Material app, you have to build them yourself. This app changes the background color to white and the text to dark grey to mimic a Material app.

### Day Six

![basicWidget](https://github.com/Dosimz/MoreFlutters/blob/master/introductionTowidget/widgetset/outimages/shoplistApp.jpeg)

##### 首先构造 ShoppingListItem 购物列表单条的 statelesswidgets
```Dart
class Product {
  const Product({this.name});
  final String name;
}

typedef void CartChangedCallback(Product product, bool inCart);

class ShoppingListItem extends StatelessWidget {
  ShoppingListItem({this.product, this.inCart, this.onCartChanged})
      : super(key: ObjectKey(product));

  final Product product;
  final bool inCart;
  final CartChangedCallback onCartChanged;

  Color _getColor(BuildContext context) {
    // The theme depends on the BuildContext because different parts of the tree
    // can have different themes.  The BuildContext indicates where the build is
    // taking place and therefore which theme to use.

    return inCart ? Colors.black54 : Theme.of(context).primaryColor;
  }

  TextStyle _getTextStyle(BuildContext context) {
    if (!inCart) return null;

    return TextStyle(
      color: Colors.black54,
      decoration: TextDecoration.lineThrough,
    );
  }

  @override
  Widget build(BuildContext context) {
    return ListTile(
      onTap: () {
        onCartChanged(product, inCart);
      },
      leading: CircleAvatar(
        backgroundColor: _getColor(context),
        child: Text(product.name[0]),
      ),
      title: Text(product.name, style: _getTextStyle(context)),
    );
  }
}

```
>The ShoppingListItem widget follows a common pattern for stateless widgets. It stores the values it receives in its constructor in final member variables, which it then uses during its build() function. For example, the inCart boolean toggles between two visual appearances: one that uses the primary color from the current theme, and another that uses gray.

>When the user taps the list item, the widget doesn’t modify its inCart value directly. Instead, the widget calls the onCartChanged function it received from its parent widget. This pattern lets you store state higher in the widget hierarchy, which causes the state to persist for longer periods of time. In the extreme, the state stored on the widget passed to runApp() persists for the lifetime of the application.

>When the parent receives the onCartChanged callback, the parent updates its internal state, which triggers the parent to rebuild and create a new instance of ShoppingListItem with the new inCart value. Although the parent creates a new instance of ShoppingListItem when it rebuilds, that operation is cheap because the framework compares the newly built widgets with the previously built widgets and applies only the differences to the underlying RenderObject.

##### 接着再构建 ShoppingList 的 StatefulWidget

```dart
class ShoppingList extends StatefulWidget {
  ShoppingList({Key key, this.products}) : super(key: key);

  final List<Product> products;

  // The framework calls createState the first time a widget appears at a given
  // location in the tree. If the parent rebuilds and uses the same type of
  // widget (with the same key), the framework re-uses the State object
  // instead of creating a new State object.

  @override
  _ShoppingListState createState() => _ShoppingListState();
}

class _ShoppingListState extends State<ShoppingList> {
  Set<Product> _shoppingCart = Set<Product>();

  void _handleCartChanged(Product product, bool inCart) {
    setState(() {
      // When a user changes what's in the cart, you need to change
      // _shoppingCart inside a setState call to trigger a rebuild.
      // The framework then calls build, below,
      // which updates the visual appearance of the app.

      if (!inCart)
        _shoppingCart.add(product);
      else
        _shoppingCart.remove(product);
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Shopping List'),
      ),
      body: ListView(
        padding: EdgeInsets.symmetric(vertical: 8.0),
        children: widget.products.map((Product product) {
          return ShoppingListItem(
            product: product,
            inCart: _shoppingCart.contains(product),
            onCartChanged: _handleCartChanged,
          );
        }).toList(),
      ),
    );
  }
}

void main() {
  runApp(MaterialApp(
    title: 'Shopping App',
    home: ShoppingList(
      products: <Product>[
        Product(name: 'Eggs'),
        Product(name: 'Flour'),
        Product(name: 'Chocolate chips'),
      ],
    ),
  ));
}

```

>The ShoppingList class extends StatefulWidget, which means this widget stores mutable state. When the ShoppingList widget is first inserted into the tree, the framework calls the createState() function to create a fresh instance of _ShoppingListState to associate with that location in the tree. (Notice that subclasses of State are typically named with leading underscores to indicate that they are private implementation details.) When this widget’s parent rebuilds, the parent creates a new instance of ShoppingList, but the framework reuses the _ShoppingListState instance that is already in the tree rather than calling createState again.

>To access properties of the current ShoppingList, the _ShoppingListState can use its widget property. If the parent rebuilds and creates a new ShoppingList, the _ShoppingListState rebuilds with the new widget value. If you wish to be notified when the widget property changes, override the didUpdateWidget() function, which is passed as oldWidget to let you compare the old widget with the current widget.

>When handling the onCartChanged callback, the _ShoppingListState mutates its internal state by either adding or removing a product from _shoppingCart. To signal to the framework that it changed its internal state, it wraps those calls in a setState() call. Calling setState marks this widget as dirty and schedules it to be rebuilt the next time your app needs to update the screen. If you forget to call setState when modifying the internal state of a widget, the framework won’t know your widget is dirty and might not call the widget’s build() function, which means the user interface might not update to reflect the changed state. By managing state in this way, you don’t need to write separate code for creating and updating child widgets. Instead, you simply implement the build function, which handles both situations

### Day Five

![basicWidget](https://github.com/Dosimz/MoreFlutters/blob/master/introductionTowidget/widgetset/outimages/statelessinvolestatefull.jpeg)

改变 Couter Widget 里的 button 和显示数字的 widget 为自定义的 widget
```dart

class CounterDisplay extends StatelessWidget {
  CounterDisplay({this.count});

  final int count;

  @override
  Widget build(BuildContext context) {
    return Text('Count: $count');
  }
}

class CounterIncrementor extends StatelessWidget {
  CounterIncrementor({this.onPressed});

  final VoidCallback onPressed;

  @override
  Widget build(BuildContext context) {
    return RaisedButton(
      onPressed: onPressed,
      child: Text('Increment'),
    );
  }
}

class Counter extends StatefulWidget {
  @override
  _CounterState createState() => _CounterState();
}

class _CounterState extends State<Counter> {
  int _counter = 0;

  void _increment() {
    setState(() {
      ++_counter;
    });
  }

  @override
  Widget build(BuildContext context) {
    return Row(children: <Widget>[
      CounterIncrementor(onPressed: _increment),
      CounterDisplay(count: _counter),
    ]);
  }
}

```
>The example above accepts user input and directly uses the result in its build() method. In more complex applications, different parts of the widget hierarchy might be responsible for different concerns; for example, one widget might present a complex user interface with the goal of gathering specific information, such as a date or location, while another widget might use that information to change the overall presentation.

>In Flutter, change notifications flow “up” the widget hierarchy by way of callbacks, while current state flows “down” to the stateless widgets that do presentation. The common parent that redirects this flow is the State. The following slightly more complex example shows how this works in practice:



### Day Four

![basicWidget](https://github.com/Dosimz/MoreFlutters/blob/master/introductionTowidget/widgetset/outimages/statefullWidget1.jpeg)

```Dart

...
      body: Center(
        child: 
        // Text('Hello, world!'),
        Counter(),
      ),
...


class Counter extends StatefulWidget {
  // This class is the configuration for the state. It holds the
  // values (in this case nothing) provided by the parent and used by the build
  // method of the State. Fields in a Widget subclass are always marked "final".

  @override
  _CounterState createState() => _CounterState();
}

class _CounterState extends State<Counter> {
  int _counter = 0;

  void _increment() {
    setState(() {
      // This call to setState tells the Flutter framework that
      // something has changed in this State, which causes it to rerun
      // the build method below so that the display can reflect the
      // updated values. If you change _counter without calling
      // setState(), then the build method won't be called again,
      // and so nothing would appear to happen.
      _counter++;
    });
  }

  @override
  Widget build(BuildContext context) {
    // This method is rerun every time setState is called,
    // for instance, as done by the _increment method above.
    // The Flutter framework has been optimized to make rerunning
    // build methods fast, so that you can just rebuild anything that
    // needs updating rather than having to individually change
    // instances of widgets.
    return Row(
      children: <Widget>[
        RaisedButton(
          onPressed: _increment,
          child: Text('Increment'),
        ),
        Text('Count: $_counter'),
      ],
    );
  }
}
```
>Stateless widgets receive arguments from their parent widget, which they store in final member variables. When a widget is asked to build(), it uses these stored values to derive new arguments for the widgets it creates.

>In order to build more complex experiences—for example, to react in more interesting ways to user input—applications typically carry some state. Flutter uses StatefulWidgets to capture this idea. StatefulWidgets are special widgets that know how to generate State objects, which are then used to hold state. Consider this basic example, using the RaisedButton mentioned earlier:

### Day Three

![basicWidget](https://github.com/Dosimz/MoreFlutters/blob/master/introductionTowidget/widgetset/outimages/GestureDetector.jpeg)

在 Day Two 内容里加入 MyButton
```dart
...
		actions: <Widget>[
          IconButton(
            icon: Icon(Icons.search),
            tooltip: 'Search',
            onPressed: null,
          ),
          MyButton(),
        ],
...

class MyButton extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return GestureDetector(
      onTap: () {
        print('MyButton was tapped!');
      },
      child: Container(
        height: 36.0,
        padding: const EdgeInsets.all(8.0),
        margin: const EdgeInsets.symmetric(horizontal: 8.0),
        decoration: BoxDecoration(
          borderRadius: BorderRadius.circular(5.0),
          color: Colors.lightGreen[500],
        ),
        child: Center(
          child: Text('Engage'),
        ),
      ),
    );
  }
}

```
>The GestureDetector widget doesn’t have a visual representation but instead detects gestures made by the user. When the user taps the Container, the GestureDetector calls its onTap() callback, in this case printing a message to the console. You can use GestureDetector to detect a variety of input gestures, including taps, drags, and scales.

>Many widgets use a GestureDetector to provide optional callbacks for other widgets. For example, the IconButton, RaisedButton, and FloatingActionButton widgets have onPressed() callbacks that are triggered when the user taps the widget.


### Day Two

![basicWidget](https://github.com/Dosimz/MoreFlutters/blob/master/introductionTowidget/widgetset/outimages/MaterialApp.jpeg)



```Dart
import 'package:flutter/material.dart';

void main() {
  runApp(MaterialApp(
    title: 'Flutter Tutorial',
    home: TutorialHome(),
  ));
}

class TutorialHome extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    // Scaffold is a layout for the major Material Components.
    return Scaffold(
      appBar: AppBar(
        // A widget to display before the [title].
        leading: IconButton(
          icon: Icon(Icons.menu),
          tooltip: 'Navigation menu',
          onPressed: null,
        ),
        title: Text('Example title'),
        // Widgets to display after the [title] widget.
        actions: <Widget>[
          IconButton(
            icon: Icon(Icons.search),
            tooltip: 'Search',
            onPressed: null,
          ),
        ],
      ),
      // body is the majority of the screen.
      body: Center(
        child: Text('Hello, world!'),
      ),
      floatingActionButton: FloatingActionButton(
        tooltip: 'Add', // used by assistive technologies
        child: Icon(Icons.add),
        onPressed: null,
      ),
    );
  }
}

```

> 当按住 button 后，会显示一个小提示，即 tooltip 定义的内容。







------------



### Day One

![basicWidget](https://github.com/Dosimz/MoreFlutters/blob/master/introductionTowidget/widgetset/outimages/basicWidget.jpeg)

```Dart
import 'package:flutter/material.dart';

class MyAppBar extends StatelessWidget {
  MyAppBar({this.title});

  // Fields in a Widget subclass are always marked "final".

  final Widget title;

  @override
  Widget build(BuildContext context) {
    return Container(
      height: 56.0, // in logical pixels
      padding: const EdgeInsets.symmetric(horizontal: 8.0),
      decoration: BoxDecoration(color: Colors.blue[500]),
      // Row is a horizontal, linear layout.
      child: Row(
        // <Widget> is the type of items in the list.
        children: <Widget>[
          IconButton(
            icon: Icon(Icons.menu),
            tooltip: 'Navigation menu',
            onPressed: null, // null disables the button
          ),
          // Expanded expands its child to fill the available space.
          Expanded(
            child: title,
          ),
          IconButton(
            icon: Icon(Icons.search),
            tooltip: 'Search',
            onPressed: null,
          ),
        ],
      ),
    );
  }
}

class MyScaffold extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    // Material is a conceptual piece of paper on which the UI appears.
    return Material(
      // Column is a vertical, linear layout.
      child: Column(
        children: <Widget>[
          MyAppBar(
            title: Text(
              'Example title',
              style: Theme.of(context).primaryTextTheme.headline6,
            ),
          ),
          Expanded(
            child: Center(
              child: Text('Hello, world!'),
            ),
          ),
        ],
      ),
    );
  }
}

void main() {
  runApp(MaterialApp(
    title: 'My app', // used by the OS task switcher
    home: MyScaffold(),
  ));
}

```

### Row, Column
> These flex widgets let you create flexible layouts in both the horizontal (Row) and vertical (Column) directions. The design of these objects is based on the web’s flexbox layout model.

### Container

> The `Container` widget lets you create a rectangular visual element. A container can be decorated with a [`BoxDecoration`](https://api.flutter.dev/flutter/painting/BoxDecoration-class.html), such as a background, a border, or a shadow. A `Container` can also have margins, padding, and constraints applied to its size. In addition, a `Container` can be transformed in three dimensional space using a matrix

