---
title: "Flutter Unit Testing"
excerpt: "Unit Test Cheatsheet for Flutter"
toc: true
categories:
  - guide
  - dart
  - dart-guide
  - flutter
---

## Simple Test

Tests go into a `void main()` function.

For a single test in a single file:

```dart
import 'package:flutter_test/flutter_test.dart'

void main() {
  test('addition', (){
    expect(1 + 2, 3);
  });
}
```

## Test groups

Tests can be grouped:

```dart
import 'package:flutter_test/flutter_test.dart'

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
