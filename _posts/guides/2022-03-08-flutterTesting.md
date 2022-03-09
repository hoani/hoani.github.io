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
