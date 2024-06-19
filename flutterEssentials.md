# Flutter Essentials
## Flutter Qwik Start [GSP1009]
### Open the Code Server editor
- Copy the Flutter IDE address displayed.
- Paste the IDE address into a Browser tab.

### Create a Flutter template
menu
Terminal
New Terminal

flutter create first_app
cd first_app

### Exploring the Flutter code
- 

### Running the Flutter web application
fwr

### Flutter Hot reload

## Build a Two Screen Flutter Application [GSP1010]
### Open the Code Server Editor
Copy the Flutter IDE address displayed
Paste the IDE address into a Browser tab

### Creating a Flutter template
flutter create first_app
cd first_app
exit

### Opening the new app
Open the folder first_app

### Running the Flutter web application
fwr

### Designing the application
AppBar	    Header bar
Scaffold	Screen layout
Text	    Text entry fields
ListView	Item list

### Working with ListView
main.dart
```
import 'package:flutter/material.dart';

void main() => runApp(MyApp());

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    final title = 'MyAwesome App';

    return MaterialApp(
      title: title,
      home: Scaffold(
        appBar: AppBar(
          title: Text(title),
        ),
        body: ListView(
          children: <Widget>[
            ListTile(
              title: Text('January'),
            ),
            ListTile(
              title: Text('February'),
            ),
            ListTile(
              title: Text('March'),
            ),
          ],
        ),
      ),
    );
  }
}
```

### Managing ListView data
main.dart
```
class MyApp extends StatelessWidget {
  final List<String> items = ['January', 'February', 'March'];

  @override
  Widget build(BuildContext context) {
    final title = 'MyAwesome App';

    return MaterialApp(
      title: title,
      home: Scaffold(
        appBar: AppBar(
          title: Text(title),
        ),
        body: ListView.builder(
          itemCount: items.length,
          itemBuilder: (context, index) {
            return ListTile(
              title: Text('${items[index]}'),
            );
          },
        ),
      ),
    );
  }
}
```

### Adding interactivity
main.dart
```
class MyApp extends StatelessWidget {
  final List<String> items = ['January', 'February', 'March'];

  @override
  Widget build(BuildContext context) {
    final title = 'MyAwesome App';

    return MaterialApp(
      title: title,
      home: Scaffold(
        appBar: AppBar(
          title: Text(title),
        ),
        body: ListView.builder(
          itemCount: items.length,
          itemBuilder: (context, index) {
            return ListTile(
              title: Text('${items[index]}'),
              // When the child is tapped, show a snackbar.
              onTap: () {
                final snackBar = SnackBar(content: Text('You selected $items[index]'));
                ScaffoldMessenger.of(context).showSnackBar(snackBar);
              },
            );
          },
        ),
      ),
    );
  }
}
```

### Creating a details page
main.dart
```
class MyApp extends StatelessWidget {
  final List<String> items = ['January', 'February', 'March'];

  @override
  Widget build(BuildContext context) {
    final title = 'MyAwesome App';

    return MaterialApp(
      title: title,
      home: Scaffold(
        appBar: AppBar(
          title: Text(title),
        ),
        body: ListView.builder(
          itemCount: items.length,
          itemBuilder: (context, index) {
            return ListTile(
              title: Text('${items[index]}'),
              // When the child is tapped, show a snackbar.
              onTap: () {
                final snackBar = SnackBar(content: Text('You selected $items[index]'));
                ScaffoldMessenger.of(context).showSnackBar(snackBar);
              },
            );
          },
        ),
      ),
    );
  }
}

class MyDetails extends StatelessWidget {

  @override
  Widget build(BuildContext context) {
    final title = 'Details Page';

    return Scaffold(
        appBar: AppBar(
          title: Text(title),
        ),
        body: Text('You selected January'),
    );
  }
}
```

### Navigating between pages
main.dart
```
class MyApp extends StatelessWidget {
  final List<String> items = ['January', 'February', 'March'];

  @override
  Widget build(BuildContext context) {
    final title = 'MyAwesome App';

    return MaterialApp(
      title: title,
      home: Scaffold(
        appBar: AppBar(
          title: Text(title),
        ),
        body: ListView.builder(
          itemCount: items.length,
          itemBuilder: (context, index) {
            return ListTile(
              title: Text('${items[index]}'),
              // When the child is tapped, show a snackbar.
              onTap: () {
                Navigator.push(
                  context,
                  MaterialPageRoute(builder: (context) => MyDetails()),
                );
              },
            );
          },
        ),
      ),
    );
  }
}
```

### Passing data between pages
main.dart
```
class MyApp extends StatelessWidget {
  final List<String> items = ['January', 'February', 'March'];

  @override
  Widget build(BuildContext context) {
    final title = 'MyAwesome App';

    return MaterialApp(
      title: title,
      home: Scaffold(
        appBar: AppBar(
          title: Text(title),
        ),
        body: ListView.builder(
          itemCount: items.length,
          itemBuilder: (context, index) {
            return ListTile(
              title: Text('${items[index]}'),
              // When the child is tapped, show a snackbar.
              onTap: () {
                Navigator.push(
                  context,
                  MaterialPageRoute(builder: (context) => MyDetails(items[index])),
                );
              },
            );
          },
        ),
      ),
    );
  }
}

class MyDetails extends StatelessWidget {
  final String month;
  MyDetails(this.month);

  @override
  Widget build(BuildContext context) {
    final title = 'Details Page';

    return Scaffold(
      appBar: AppBar(
        title: Text(title),
      ),
      body: Text('You selected $month'),
    );
  }
}
```


## Working with Onscreen Data in a Flutter Application [GSP1011]
### Open the Code Server Editor
Copy the Flutter IDE address displayed.
Paste the Editor address into a Browser tab.

### Creating a Flutter Template
flutter create first_app 

### Opening the new app
Open the folder first_app.

### Running the Flutter Web application
fwr

### Adding Boilerplate code
main.dart
```
import 'package:flutter/material.dart';

// TODO: Embedded List
class GoogleProducts {
  final List<String> items = [
    'Cloud Functions',
    'App Engine',
    'Kubernetes Engine',
    'Compute Engine',
    'Bare Metal',
    'Preemptible VMs',
    'Shielded VMs',
    'Sole-tenet Nodes',
    'VMWare Engine',
    'Cloud Firestore',
    'Cloud Storage',
    'Persistent Disk',
    'Local SSD',
    'Cloud Bigtable',
    'Cloud Firestore',
    'Cloud Memorystore',
    'Cloud Spanner',
  ];
}

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({Key? key}) : super(key: key);
  @override
  Widget build(BuildContext context) {
    const title = 'Google Products';
    return const MaterialApp(
      title: title,
      debugShowCheckedModeBanner: false,
      home: ProductHomeWidget(title),
    );
  }
}

// TODO: ProductHomeWidget
class ProductHomeWidget extends StatelessWidget {
  final String title;

  const ProductHomeWidget(this.title, {Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      // backgroundColor: Colors.white,
      appBar: AppBar(
        // backgroundColor: Colors.transparent,
        // elevation: 0,
        // TODO: Enable AppBarLeading
        // leading: const AppBarLeading(),
        // TODO: Enable AppBarLeading
        // actions: const [
        //   AppBarActionsShare(),
        // ],
        title: Text(title, style: const TextStyle(color: Colors.black)),
      ),
      // body: ProductListView(),
    );
  }
}

// TODO: AppBarLeading
class AppBarLeading extends StatelessWidget {
  const AppBarLeading({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return const IconButton(
      icon: Icon(
        Icons.menu,
      ),
      onPressed: null,
    );
  }
}

// TODO: AppBarActionsShare
class AppBarActionsShare extends StatelessWidget {
  const AppBarActionsShare({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return IconButton(
        icon: const Icon(
          Icons.share,
        ),
        onPressed: () {
          const snackBar =
              SnackBar(content: Text('You selected the Share Action'));
          ScaffoldMessenger.of(context).showSnackBar(snackBar);
        });
  }
}

// TODO: ProductListView
class ProductListView extends StatelessWidget {
  final googleProducts = GoogleProducts();

  ProductListView({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return ListView.builder(
      itemCount: googleProducts.items.length,
      itemBuilder: (context, index) {
        return ProductListTile(googleProducts.items[index]);
      },
    );
  }
}

// TODO: ProductListTile
class ProductListTile extends StatelessWidget {
  final String? productLabel;

  const ProductListTile(this.productLabel, {Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return ListTile(
      title: Text('$productLabel', style: const TextStyle(color: Colors.black)),
      subtitle: const Text('SubTitle', style: TextStyle(color: Colors.black)),
      leading: const Icon(Icons.help_center_outlined, color: Colors.black),
      // When the child is tapped, show a snackbar.
      onTap: () {
        // TODO: Enable onTap
        // final snackBar = SnackBar(content: Text('You selected $productLabel'));
        // ScaffoldMessenger.of(context).showSnackBar(snackBar);

        // TODO: Navigation to the Details Page
        // Navigator.push(
        //   context,
        //   MaterialPageRoute(builder: (context) => const MyDetails()),
        // );
      },
    );
  }
}

// TODO: Add a details page
// class MyDetails extends StatelessWidget {
//   const MyDetails({Key? key}) : super(key: key);

//   @override
//   Widget build(BuildContext context) {
//     const title = 'Details Page';

//     return Scaffold(
//       // backgroundColor: Colors.white,
//       appBar: AppBar(
//         iconTheme: const IconThemeData(
//           color: Colors.grey, //change your color here
//         ),
//         backgroundColor: Colors.transparent,
//         elevation: 0,
//         // leading: const AppBarLeading(),
//         actions: const [
//           AppBarActionsShare(),
//         ],
//         title: const Text(title, style: TextStyle(color: Colors.black)),
//       ),
//       // appBar: AppBar(
//       //   title: const Text(title),
//       // ),
//       body: const Center(
//         child: Text('Hello Details Page')),
//     );
//   }
// }
```

### Designing our application
AppBar	    Header bar
Scaffold	Screen layout
Text	    Text entry fields
ListView	Item list

### Working with AppBar
leading	An icon placed at the left side of the AppBar
title	A title placed on the App Bar
actions	An icon placed on the right side of the AppBar
main.dart
```
  // TODO: Enable AppBarLeading
  // leading: const AppBarLeading(),
```
```
        // TODO: Enable AppBarActionsShare
        // actions: const [
        //   AppBarActionsShare(),
        // ],
```
```
      // TODO: Style the AppBar
      // backgroundColor: Colors.white,
      appBar: AppBar(
        // backgroundColor: Colors.transparent,
        // elevation: 0,
```

### Working with ListView
main.dart
```
  // TODO: Enable the ListView 
  // body: ProductListView(),
```

### Adding interactivity
main.dart
```
  // TODO: Enable onTap  
  // onTap: () {
  //   final snackBar = SnackBar(content: Text('You selected $productLabel'));
  //   ScaffoldMessenger.of(context).showSnackBar(snackBar);
  // },
```

### Creating a Details page
main.dart
```
// TODO: Add a details page 
// class MyDetails extends StatelessWidget {
//   const MyDetails({Key? key}) : super(key: key);
// 
//   @override
//   Widget build(BuildContext context) {
//     const title = 'Details Page';
// 
//     return Scaffold(
//       appBar: AppBar(
//         iconTheme: const IconThemeData(
//           color: Colors.grey, //change your color here
//         ),
//         backgroundColor: Colors.transparent,
//         elevation: 0,
//         // leading: const AppBarLeading(),
//         actions: const [
//           AppBarActionsShare(),
//         ],
//         title: const Text(title, style: TextStyle(color: Colors.black)),
//       ),
//       body: const Center(
//         child: Text('Hello Details Page')),
//     );
//   }
// }
```

### Navigating between pages
main.dart
```
  @override
  Widget build(BuildContext context) {
    return ListTile(
      title: Text('$productLabel', style: const TextStyle(color: Colors.black)),
      subtitle: const Text('SubTitle', style: TextStyle(color: Colors.black)),
      leading: const Icon(Icons.help_center_outlined, color: Colors.black),
      // When the child is tapped, show a snackbar.
      onTap: () {
        // TODO: SnackBar onscreen notification
        // final snackBar = SnackBar(content: Text('You selected $productLabel'));
        // ScaffoldMessenger.of(context).showSnackBar(snackBar);

        // TODO: Navigation to the Details Page
        Navigator.push(
          context,
          MaterialPageRoute(builder: (context) => const MyDetails()),
        );
      },
    );
  }
```


## Implementing Page Navigation in a Flutter Application [GSP1012]
### Open the Code Server Editor
Copy the Flutter IDE address displayed.
Paste the Editor address into a Browser tab.

### Creating a Flutter Template
flutter create first_app 

### Opening the new app
Open the folder first_app.

### Running the Flutter Web application
fwr

### Adding Boilerplate code
main.dart
```
import 'package:flutter/material.dart';

// TODO: Embedded List
class GoogleProducts {
  final List<String> items = [
    'Cloud Functions',
    'App Engine',
    'Kubernetes Engine',
    'Compute Engine',
    'Bare Metal',
    'Pre-emtible VMs',
    'Shielded VMs',
    'Sole-tenet Nodes',
    'VMWare Engine',
    'Cloud Firestore',
    'Cloud Storage',
    'Persistent Disk',
    'Local SSD',
    'Cloud Bigtable',
    'Cloud Firestore',
    'Cloud Memorystore',
    'Cloud Spanner',
  ];
}

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({Key? key}) : super(key: key);
  @override
  Widget build(BuildContext context) {
    const title = 'Google Products';
    return const MaterialApp(
      title: title,
      debugShowCheckedModeBanner: false,
      home: ProductHomeWidget(title),
    );
  }
}

// TODO: ProductHomeWidget
class ProductHomeWidget extends StatelessWidget {
  final String title;

  const ProductHomeWidget(this.title, {Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: Colors.white,
      appBar: AppBar(
        backgroundColor: Colors.transparent,
        elevation: 0,
        iconTheme: const IconThemeData(
          color: Colors.grey, //change your color here
        ),
        actions: const [
          AppBarActionsShare(),
        ],
        title: Text(title, style: const TextStyle(color: Colors.black)),
      ),
      body: ProductListView(),
      // TODO: Add Drawer
      // drawer: const MyDrawerWidget(),
    );
  }
}

// TODO: AppBarLeading
class AppBarLeading extends StatelessWidget {
  const AppBarLeading({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return const IconButton(
      icon: Icon(
        Icons.menu,
      ),
      onPressed: null,
    );
  }
}

// TODO: AppBarActionsShare
class AppBarActionsShare extends StatelessWidget {
  const AppBarActionsShare({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return IconButton(
        icon: const Icon(
          Icons.share,
        ),
        onPressed: () {
          const snackBar =
              SnackBar(content: Text('You selected the Action: Share'));
          ScaffoldMessenger.of(context).showSnackBar(snackBar);
        });
  }
}


// TODO: Enable Drawer
class MyDrawerWidget extends StatelessWidget {
  const MyDrawerWidget({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return Drawer(
      child: ListView(
        children: const [
          DrawerHeader(
            child: Icon(Icons.flutter_dash, size: 35),
          ),
        ],
      ),
    );
  }
}

// TODO: ProductListView
class ProductListView extends StatelessWidget {
  final googleProducts = GoogleProducts();

  ProductListView({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return ListView.builder(
      itemCount: googleProducts.items.length,
      itemBuilder: (context, index) {
        return ProductListTile(googleProducts.items[index]);
      },
    );
  }
}

// TODO: ProductListTile
class ProductListTile extends StatelessWidget {
  final String? productLabel;

  const ProductListTile(this.productLabel, {Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return ListTile(
      title: Text('$productLabel', style: const TextStyle(color: Colors.black)),
      subtitle: const Text('SubTitle', style: TextStyle(color: Colors.black)),
      leading: const Icon(Icons.info, color: Colors.black),
      // When the child is tapped, show a snackbar.
      onTap: () {
        Navigator.push(
          context,
          MaterialPageRoute(builder: (context) => const MyDetails()),
        );
      },
    );
  }
}

// TODO: Add a details page
class MyDetails extends StatelessWidget {
  final title = 'Details Page';

  const MyDetails({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: Colors.black,
      body: DefaultTabController(
        length: 4,
        child: Scaffold(
          appBar: AppBar(
            backgroundColor: Colors.transparent,
            elevation: 0,
            iconTheme: const IconThemeData(
              color: Colors.grey, //change your color here
            ),
            title: Text(title, style: const TextStyle(color: Colors.grey)),
            actions: const [
              AppBarActionsShare(),
            ],
            // TODO: Add TabBar
            // bottom: const TabBar(
            //   indicatorColor: Colors.black,
            //   tabs: [
            //     Tab(
            //       icon: Icon(Icons.home, color: Colors.grey),
            //       child: Text('Overview',
            //           style: TextStyle(
            //               color: Colors.grey, fontWeight: FontWeight.bold)),
            //     ),
            //     Tab(
            //       icon: Icon(Icons.favorite, color: Colors.grey),
            //       child: Text('Docs',
            //           style: TextStyle(
            //               color: Colors.grey, fontWeight: FontWeight.bold)),
            //     ),
            //     Tab(
            //       icon: Icon(Icons.list, color: Colors.grey),
            //       child: Text('Information',
            //           style: TextStyle(
            //               color: Colors.grey, fontWeight: FontWeight.bold)),
            //     ),
            //     Tab(
            //       icon: Icon(Icons.info, color: Colors.grey),
            //       child: Text('Other',
            //           style: TextStyle(
            //               color: Colors.grey, fontWeight: FontWeight.bold)),
            //     ),
            //   ],
            // ),
          ),
          // TODO: Add TabBarView
          // body: const TabBarView(
          //   children: [
          //     SizedBox(
          //       child: Center(
          //         child: Text('Tab Page 1'),
          //       ),
          //     ),
          //     SizedBox(
          //       child: Center(
          //         child: Text('Tab Page 2'),
          //       ),
          //     ),
          //     SizedBox(
          //       child: Center(
          //         child: Text('Tab Page 3'),
          //       ),
          //     ),
          //     SizedBox(
          //       child: Center(
          //         child: Text('Tab Page 4'),
          //       ),
          //     ),
          //   ],
          // ),
        ),
      ),
    );
  }
}
```
### Design our application
TabView	  Tabbed page view for information
RichText	Enhanced text for displaying information
Image	    Image display

Drawer	Application navigation
Icon	Add an avatar to the Drawer

### Including a Tabview
main.dart
```
  // TODO: Add TabBar
  // bottom: const TabBar(
  //   indicatorColor: Colors.black,
  //   tabs: [
  //     Tab(
  //       icon: Icon(Icons.home, color: Colors.grey),
  //       child: Text('Overview',
  //           style: TextStyle(
  //               color: Colors.grey, fontWeight: FontWeight.bold)),
  //     ),
  //     Tab(
  //       icon: Icon(Icons.favorite, color: Colors.grey),
  //       child: Text('Docs',
  //           style: TextStyle(
  //               color: Colors.grey, fontWeight: FontWeight.bold)),
  //     ),
  //     Tab(
  //       icon: Icon(Icons.list, color: Colors.grey),
  //       child: Text('Information',
  //           style: TextStyle(
  //               color: Colors.grey, fontWeight: FontWeight.bold)),
  //     ),
  //     Tab(
  //       icon: Icon(Icons.info, color: Colors.grey),
  //       child: Text('Other',
  //           style: TextStyle(
  //               color: Colors.grey, fontWeight: FontWeight.bold)),
  //     ),
  //   ],
  // ),
```
### Adding a TabBarView
main.dart
```
 // body: const TabBarView(
 //   children: [
 //     SizedBox(
 //       child: Center(
 //         child: Text('Tab Page 1'),
 //       ),
 //     ),
 //     SizedBox(
 //       child: Center(
 //         child: Text('Tab Page 2'),
 //       ),
 //     ),
 //     SizedBox(
 //       child: Center(
 //         child: Text('Tab Page 3'),
 //       ),
 //     ),
 //     SizedBox(
 //       child: Center(
 //         child: Text('Tab Page 4'),
 //       ),
 //     ),
 //   ],
 // ),
```
### Adding a drawer
main.dart
```
  drawer: const MyDrawerWidget(),
```
### Routing to Home
main.dart
```
  children: [
    const DrawerHeader(
      child: Icon(Icons.flutter_dash, size: 35),
    ),
    ListTile(
      leading: const Icon(Icons.home),
      title: const Text('Home'),
      onTap: () {
        Navigator.of(context).push(
          MaterialPageRoute(builder: (context) => const MyApp()),
        );
      },
    ),    
```
### Routing to Details
main.dart
```
  onTap: () {
    Navigator.of(context).push(
      MaterialPageRoute(builder: (context) => const MyDetails()),
    );
  },
```