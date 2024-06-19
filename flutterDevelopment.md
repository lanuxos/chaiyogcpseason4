# Flutter Development
## Getting started with Flutter Development []
### Open the Code Server editor
### Flutter extensions
### Create a Flutter template
### Exploring the Flutter code
### Running the Flutter Web application
### Flutter Hot reload

## Flutter Startup Namer [GSP886]
### Open the Frontend service
### create the starter Flutter app
flutter create startup_namer

### Exploro the Flutter app
### Run the Flutter code
fwr

### Using an external packages
pubspec.yalm
```
english_words: ^4.0.0   # add this line
```

main.dart
```
import 'package:flutter/material.dart';
import 'package:english_words/english_words.dart';

void main() => runApp(MyApp());

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    final wordPair = WordPair.random(); // Add this line.
    return MaterialApp(
      title: 'Welcome to Flutter',
      home: Scaffold(
        appBar: AppBar(
          title: Text('Welcome to Flutter'),
        ),
        body: Center(                       // Drop the const, and
          //child: Text('Hello World'),     // Replace this text...
          child: Text(wordPair.asPascalCase),  // With this text.
        ),
      ),
    );
  }
}
```
### Run the Flutter code
fwr

### Adding the Stateful widget
main.dart
```
class RandomWords extends StatefulWidget {
  @override
  _RandomWordsState createState() => _RandomWordsState();
}

class _RandomWordsState extends State<RandomWords> {
  @override                                  
  Widget build(BuildContext context) {
    final wordPair = WordPair.random();      // NEW
    return Text(wordPair.asPascalCase);      // NEW
  }                                         
}
```

```
final wordPair = WordPair.random();  // DELETE
child: RandomWords(),                // REPLACE this line
```

### Creating an infinite scrolling ListView
main.dart
```
import 'package:flutter/material.dart';
import 'package:english_words/english_words.dart';

void main() => runApp(MyApp());

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    // final wordPair = WordPair.random(); // Add this line.
    return MaterialApp(
      title: 'Welcome to Flutter',
      home: RandomWords(),
    );
  }
}

class RandomWords extends StatefulWidget {
  @override
  _RandomWordsState createState() => _RandomWordsState();
}

class _RandomWordsState extends State<RandomWords> {
  final _biggerFont = const TextStyle(fontSize: 18); // NEW
  final _suggestions = <WordPair>[]; // NEW

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      // Add from here...
      appBar: AppBar(
        title: Text('Startup Name Generator'),
      ),
      body: _buildSuggestions(),
    ); // ... to here.
  }

  Widget _buildSuggestions() {
    return ListView.builder(
        padding: const EdgeInsets.all(16),
        // The itemBuilder callback is called once per suggested
        // word pairing, and places each suggestion into a ListTile
        // row. For even rows, the function adds a ListTile row for
        // the word pairing. For odd rows, the function adds a
        // Divider widget to visually separate the entries. Note that
        // the divider may be difficult to see on smaller devices.
        itemBuilder: (BuildContext _context, int i) {
          // Add a one-pixel-high divider widget before each row
          // in the ListView.
          if (i.isOdd) {
            return Divider();
          }

          // The syntax "i ~/ 2" divides i by 2 and returns an
          // integer result.
          // For example: 1, 2, 3, 4, 5 becomes 0, 1, 1, 2, 2.
          // This calculates the actual number of word pairings
          // in the ListView,minus the divider widgets.
          final int index = i ~/ 2;
          // If you've reached the end of the available word
          // pairings...
          if (index >= _suggestions.length) {
            // ...then generate 10 more and add them to the
            // suggestions list.
            _suggestions.addAll(generateWordPairs().take(10));
          }
          return _buildRow(_suggestions[index]);
        });
  }

  Widget _buildRow(WordPair pair) {
    return ListTile(
      title: Text(
        pair.asPascalCase,
        style: _biggerFont,
      ),
    );
  }
}

```

## Material Components for Flutter Basics [GSP887]
### Open the Code Server Editor
### Download the MDC Starter app
https://github.com/material-components/material-components-flutter-codelabs.git

- Create a web build
cd ~/material-components-flutter-codelabs/mdc_100_series
flutter create .

fwr

### Add TextField widgets
login.dart
```
            // TODO: Add TextField widgets (101)
            // [Name]
            TextField(
              decoration: InputDecoration(
                filled: true,
                labelText: 'Username',
              ),
            ),
            // spacer
            const SizedBox(height: 12.0),
            // [Password]
            TextField(
              decoration: InputDecoration(
                filled: true,
                labelText: 'Password',
              ),
              obscureText: true,
            ),
```

### Add buttons
- Add the OverflowBar
login.dart
Add the OverflowBar to the ListView's children by replacing // TODO: Add button bar (101)
```
  // TODO: Add button bar (101)
  OverflowBar(
    alignment: MainAxisAlignment.end,
    // TODO: Add a beveled rectangular border to CANCEL (103)
    children: <Widget>[
      // TODO: Add buttons (101)
    ],
  ),
```
- Add the buttons
Then add two buttons to the OverflowBar's list of children
```
  // TODO: Add buttons (101)
    TextButton(
      child: const Text('CANCEL'),
      onPressed: () {
        // TODO: Clear the text fields (101)
      },
    ),
    // TODO: Add an elevation to NEXT (103)
    // TODO: Add a beveled rectangular border to NEXT (103)
    ElevatedButton(
      child: const Text('NEXT'),
      onPressed: () {
    // TODO: Show the next page (101) 
      },
    ),
```
- Add TextEditingControllers
Right under the _LoginPageState class's declaration in login.dart, find the // TODO: Add text editing controllers (101). Replace it to add the controllers as final variables
```
  // TODO: Add text editing controllers (101)
  final _usernameController = TextEditingController();
  final _passwordController = TextEditingController();
```
- Pop
In the ElevatedButton's NEXT onPressed: function, pop the most recent route from the Navigator by replacing // TODO: Show the next page (101)
```
        // TODO: Show the next page (101)
        Navigator.pop(context);
```
- set resizeToAvoidBottomInset
home.dart
```
    return Scaffold(
      // TODO: Add app bar (102)
      // TODO: Add a grid view (102)
      body: Center(
        child: Text('You did it!'),
      ),
      // TODO: Set resizeToAvoidBottomInset (101)
      resizeToAvoidBottomInset: false,
    );
```
### Add a top app bar
- Add an AppBar widget
In home.dart, add an AppBar to the Scaffold. Replace the // TODO: Add app bar (102)
home.dart
```
  // TODO: Add app bar (102)
  appBar: AppBar(
    // TODO: Add buttons and title (102)
  ),
```
- Add a text widget
home.dart
```
// TODO: Add app bar (102)  
  appBar: AppBar(
    // TODO: Add buttons and title (102)
    title: Text('SHRINE'),
    // TODO: Add trailing buttons (102)
  ),
```
- Add a leading IconButton
home.dart
set an IconButton for the AppBar's leading: field. Replace the // TODO: Add buttons and title (102)
```
    // TODO: Add buttons and title (102)
    leading: IconButton(
      icon: Icon(
        Icons.menu,
        semanticLabel: 'menu',
      ),
      onPressed: () {
        print('Menu button');
      },
    ),
```
- Add actions
home.dart
Add them to the AppBar instance after the title. Replace // TODO: Add trailing buttons (102
```
// TODO: Add trailing buttons (102)
actions: <Widget>[
  IconButton(
    icon: Icon(
      Icons.search,
      semanticLabel: 'search',
    ),
    onPressed: () {
      print('Search button');
    },
  ),
  IconButton(
    icon: Icon(
      Icons.tune,
      semanticLabel: 'filter',
    ),
    onPressed: () {
      print('Filter button');
    },
  ),
],
```
### Make a card collection
- Multiply the card into a collection
Make a new private function above the build() function
```
// TODO: Make a collection of cards (102)
List<Card> _buildGridCards(int count) {
  List<Card> cards = List.generate(
    count,
    (int index) => Card(
      clipBehavior: Clip.antiAlias,
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: <Widget>[
          AspectRatio(
            aspectRatio: 18.0 / 11.0,
            child: Image.asset('assets/diamond.png'),
          ),
          Padding(
            padding: EdgeInsets.fromLTRB(16.0, 12.0, 16.0, 8.0),
            child: Column(
              crossAxisAlignment: CrossAxisAlignment.start,
              children: <Widget>[
                Text('Title'),
                SizedBox(height: 8.0),
                Text('Secondary Text'),
              ],
            ),
          ),
        ],
      ),
    ),
  );

  return cards;
}
```
Assign the generated cards to GridView's children field. Remember to replace everything contained in the body with this new code
```
// TODO: Add a grid view (102)
body: GridView.count(
  crossAxisCount: 2,
  padding: EdgeInsets.all(16.0),
  childAspectRatio: 8.0 / 9.0,
  children: _buildGridCards(10) // Replace
),
```
Update the children of the GridView
```
  children: <Widget>[
    Card(
      clipBehavior: Clip.antiAlias,
      child: Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: <Widget>[
          AspectRatio(
            aspectRatio: 18.0 / 11.0,
            child: Image.asset('assets/diamond.png'),
          ),
          Padding(
            padding: const EdgeInsets.fromLTRB(16.0, 12.0, 16.0, 8.0),
            child: Column(
              crossAxisAlignment: CrossAxisAlignment.start,
              children: <Widget>[
                Text('Title'),
                const SizedBox(height: 8.0),
                Text('Secondary Text'),
              ],
            ),
          ),
        ],
      ),
    )
  ],
```
- Add product data
home.dart
```
import 'package:intl/intl.dart';
import 'model/products_repository.dart';
import 'model/product.dart';
```
update _buildGridCards()
```
// TODO: Make a collection of cards (102)

// Replace this entire method
List<Card> _buildGridCards(BuildContext context) {
  List<Product> products = ProductsRepository.loadProducts(Category.all);

  if (products == null || products.isEmpty) {
    return const <Card>[];
  }

  final ThemeData theme = Theme.of(context);
  final NumberFormat formatter = NumberFormat.simpleCurrency(
      locale: Localizations.localeOf(context).toString());

  return products.map((product) {
    return Card(
      clipBehavior: Clip.antiAlias,
      // TODO: Adjust card heights (103)
      child: Column(
        // TODO: Center items on the card (103)
        crossAxisAlignment: CrossAxisAlignment.start,
        children: <Widget>[
          AspectRatio(
            aspectRatio: 18 / 11,
            child: Image.asset(
              product.assetName,
              package: product.assetPackage,
             // TODO: Adjust the box size (102)
             fit: BoxFit.fitWidth,
            ),
          ),
          Expanded(
            child: Padding(
              padding: EdgeInsets.fromLTRB(16.0, 12.0, 16.0, 8.0),
              child: Column(
               // TODO: Align labels to the bottom and center (103)
               crossAxisAlignment: CrossAxisAlignment.start,
                // TODO: Change innermost Column (103)
                children: <Widget>[
                 // TODO: Handle overflowing labels (103)
                 Text(
                    product.name,
                    style: theme.textTheme.headline6,
                    maxLines: 1,
                  ),
                  SizedBox(height: 8.0),
                  Text(
                    formatter.format(product.price),
                    style: theme.textTheme.subtitle2,
                  ),
                ],
              ),
            ),
          ),
        ],
      ),
    );
  }).toList();
}
```
- Change the build() function to pass the BuildContext to _buildGridCards() before you try to compile
```
// TODO: Add a grid view (102)
body: GridView.count(
  crossAxisCount: 2,
  padding: EdgeInsets.all(16.0),
  childAspectRatio: 8.0 / 9.0,
  children: _buildGridCards(context) // Changed code
),
```

### Final code
- home.dart
```
// Copyright 2018-present the Flutter authors. All Rights Reserved.
//
// Licensed under the Apache License, Version 2.0 (the "License");
// you may not use this file except in compliance with the License.
// You may obtain a copy of the License at
//
// http://www.apache.org/licenses/LICENSE-2.0
//
// Unless required by applicable law or agreed to in writing, software
// distributed under the License is distributed on an "AS IS" BASIS,
// WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
// See the License for the specific language governing permissions and
// limitations under the License.

import 'package:flutter/material.dart';
import 'package:intl/intl.dart';
import 'model/products_repository.dart';
import 'model/product.dart';

class HomePage extends StatelessWidget {
  const HomePage({Key? key}) : super(key: key);

  // TODO: Make a collection of cards (102)

// Replace this entire method
  List<Card> _buildGridCards(BuildContext context) {
    List<Product> products = ProductsRepository.loadProducts(Category.all);

    if (products == null || products.isEmpty) {
      return const <Card>[];
    }

    final ThemeData theme = Theme.of(context);
    final NumberFormat formatter = NumberFormat.simpleCurrency(
        locale: Localizations.localeOf(context).toString());

    return products.map((product) {
      return Card(
        clipBehavior: Clip.antiAlias,
        // TODO: Adjust card heights (103)
        child: Column(
          // TODO: Center items on the card (103)
          crossAxisAlignment: CrossAxisAlignment.start,
          children: <Widget>[
            AspectRatio(
              aspectRatio: 18 / 11,
              child: Image.asset(
                product.assetName,
                package: product.assetPackage,
                // TODO: Adjust the box size (102)
                fit: BoxFit.fitWidth,
              ),
            ),
            Expanded(
              child: Padding(
                padding: EdgeInsets.fromLTRB(16.0, 12.0, 16.0, 8.0),
                child: Column(
                  // TODO: Align labels to the bottom and center (103)
                  crossAxisAlignment: CrossAxisAlignment.start,
                  // TODO: Change innermost Column (103)
                  children: <Widget>[
                    // TODO: Handle overflowing labels (103)
                    Text(
                      product.name,
                      style: theme.textTheme.headline6,
                      maxLines: 1,
                    ),
                    SizedBox(height: 8.0),
                    Text(
                      formatter.format(product.price),
                      style: theme.textTheme.subtitle2,
                    ),
                  ],
                ),
              ),
            ),
          ],
        ),
      );
    }).toList();
  }

  // TODO: Add a variable for Category (104)
  @override
  Widget build(BuildContext context) {
    // TODO: Return an AsymmetricView (104)
    // TODO: Pass Category variable to AsymmetricView (104)
    return Scaffold(
      // TODO: Add app bar (102)
      // TODO: Add app bar (102)
      // TODO: Add app bar (102)
      appBar: AppBar(
        // TODO: Add buttons and title (102)
        leading: IconButton(
          icon: Icon(Icons.menu, semanticLabel: 'menu'),
          onPressed: () {
            print('Menu Button');
          },
        ),
        title: Text('SHRINE'),
        // TODO: Add trailing buttons (102)
        actions: <Widget>[
          IconButton(
              onPressed: () {
                print('Search button');
              },
              icon: Icon(
                Icons.search,
                semanticLabel: 'search',
              ))
        ],
      ),
      // TODO: Add a grid view (102)
      // TODO: Add a grid view (102)
      body: GridView.count(
          crossAxisCount: 2,
          padding: EdgeInsets.all(16.0),
          childAspectRatio: 8.0 / 9.0,
          children: _buildGridCards(context) // Replace
          ),
      // TODO: Set resizeToAvoidBottomInset (101)
      resizeToAvoidBottomInset: false,
    );
  }
}
```
## Dart Functions Framework [GSP889]
### Create a backend service
gcloud services enable run.googleapis.com
gcloud config set run/region  
git clone https://github.com/GoogleCloudPlatform/functions-framework-dart.git && cd functions-framework-dart/examples/fullstack/backend
nano cloudbuild.yaml
```
steps:
- name: gcr.io/cloud-builders/docker
  args: ['build', '-t', 'gcr.io/$PROJECT_ID/backend-service', '.']
- name: 'gcr.io/cloud-builders/docker'
  args: ['push', 'gcr.io/$PROJECT_ID/backend-service']
```
gcloud builds submit --config cloudbuild.yaml
gcloud run deploy backend-service \
   --image gcr.io/$GOOGLE_CLOUD_PROJECT/backend-service \
   --platform managed \
   --region us-central1 \
   --allow-unauthenticated \
   --max-instances=2
backend_service=$(gcloud run services list --platform managed --format='value(URL)' --filter='backend-service')
curl -X POST -H "content-type: application/json" \
   -d '{ "name": "World" }' -i -w "\n" $backend_service

### Create a proxy service
gcloud run deploy backend-proxy \
   --image gcr.io/qwiklabs-resources/backend-proxy \
   --platform managed \
   --region us-central1 \
   --allow-unauthenticated \
   --set-env-vars "ENDPOINT=$backend_service" \
   --max-instances=2
backend_proxy=$(gcloud run services list --platform managed --format='value(URL)' --filter='backend-proxy')
curl -X POST -H "content-type: application/json" \
   -d '{ "name": "World" }' -i -w "\n" $backend_proxy

### Open the frontend service
### Accepting the Flutter extensions
### Clone the code repository
https://github.com/GoogleCloudPlatform/functions-framework-dart.git

### Import the starter app
/home/ide-dev/flutter/functions-framework-dart/examples/fullstack/frontend

### Exploring the Flutter code

### Creating a frontend service
cp assets/config/dev.json assets/config/prod.json
prod.jon
```
{
  "greetingUrl": "https://backend-o6txmjtuaa-uc.a.run.app"
}
```

### Run the Flutter code
fwr --dart-define=ENV=prod
