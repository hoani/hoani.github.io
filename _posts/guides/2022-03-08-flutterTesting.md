---
title: "Flutter Testing"
excerpt: "Test Cheatsheet for Flutter"
toc: true
categories:
  - guide
  - dart
  - dart-guide
  - flutter
---

## Unit Testing

Tests go into a `void main()` function.

For a single test in a single file:

```dart
import 'package:flutter_test/flutter_test.dart';

void main() {
  test('addition', (){
    expect(1 + 2, 3);
  });
}
```

## Test groups

Tests can be grouped:

```dart
import 'package:flutter_test/flutter_test.dart';

void main() {
  group('arithmetic', (){
    test('addition', (){
      expect(1 + 2, 3);
    });
    test('subtraction', (){
      // ...
    });
    // ...etc
  });
}
```

Tests within a group can have `setUp` and `tearDown` methods:
```dart
void main() {
  group('arithmetic', (){
    setUp(() {
      // test set up.
    });
    test('addition', (){
      expect(1 + 2, 3);
    });
    // ...etc
    tearDown((){
      // test clean up.
    });
  });
}
```

## Widget Tests

Import your widget and use `testWidgets` instead of `test`. This gives us a `WidgetTester` which we can use to load widgets.

We use `tester.pumpWidget(widget)` to load the `Widget` under test.

Next we can use `matchers` to find properties about our widget.

```dart
import 'package:flutter_test/flutter_test.dart';
import 'package:hoani_land/common_widgets/hoani_form.dart';

void main() {
  testWidgets('empty form', (WidgetTester tester) async {
    await tester.pumpWidget(
      const HoaniForm(),
    );

    final submitButton = find.byType(SubmitButton);
    expect(submitButton, findsOneWidget);
  });
}
```

The `WidgetTester` provides a number of methods which can be used to take actions in your widget. For example, `onTap`:

```dart
import 'package:flutter_test/flutter_test.dart';
import 'package:hoani_land/common_widgets/hoani_form.dart';

void main() {
  testWidgets('onSubmit', (WidgetTester tester) async {
    var submitted = false;
    await tester.pumpWidget(
      const HoaniForm(
        onSubmit: () => submitted = true,
      ),
    );

    final submitButton = find.byType(SubmitButton);
    expect(submitButton, findsOneWidget);
    await tester.tap(submitButton);
    expect(submitted, true);
  });
}
```

## Mocking with Mockito

[Mockito (pub.dev)](https://pub.dev/packages/mockito) is a mocking library for dart.

Mocks need to be generated in Flutter. To do this add `build_runner` to dart:

```sh
dart pub add build_runner --dev
```

If we want to mock a `Database` class, in your test file add the following:

```dart
import 'package:hoani_land/services/database.dart';
import 'package:mockito/annotations.dart';
import 'package:mockito/mockito.dart';

@GenerateMocks([Database])
main() {
  //...
}
```

Run the mock generation:

```sh
flutter pub run build_runner build
```

Now we can add the following import to our test file (assuming the test file is called `foo_test.dart`):

```dart
import 'foo_test.mocks.dart'
```

You can create and use a mock in your test:

```dart
    test('no update on creation', (){
      MockDatabase database = MockDatabase();
      final tracker = ItemsTracker(
        store: 'test store', 
        database: database,
      );
      verifyNever(database.getItems(any));
    });
```

Any calls that are made to your mock need to be stubbed. This is done using `when`:

```dart
    test('calls get items on update', (){
      MockDatabase database = MockDatabase();
      when(database.getItems(any))
        .thenAnswer((_) => Stream.value('A'));
      final tracker = ItemsTracker(
        store: 'test store', 
        database: database,
      );
      verify(database.getItems(any).called(1));
    });
```

Note - in the above example, we used `thenAnswer` in our stub. The following are guidelines are useful:
* `thenReturn` for functions which return immediately.
* `thenAnswer` for `Future` or `Stream`.
* `thenThrow` if you want to throw.


