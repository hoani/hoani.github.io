---
title: "Flutter State Management"
excerpt: "State patterns in Flutter"
permalink: "/guides/software/dart/flutter-state-patterns"
toc: true
categories:
  - guide
  - dart
  - dart-guide
  - flutter
---

## Stateful Widgets

Usually fine for simple widgets whose state remains local to the widget.

```dart
void _onPressed() {
  setState(
    (){
      _submitted = true;
  })
}
```

## BLoCs (buisness logic components)

BLoCs use streams of immutable objects to update state and decouple the UI from services.

Given a model (oversimplified for this example):

```dart
class SubmittedModel {
  SubmittedModel({this.submitted = false});

  final bool submitted;
  // ...
}
```

Define a BLoC which provides the logic around this model.  

```dart
class SubmittedBloc {
  SubmittedBloc({required this.service})

  final MyService service;

  static SubmittedBloc create(BuildContext context) {
    final service = Provider.of<MyService>(
      context, 
      listen: false
    );
    return SubmittedBloc(service: service);
  }

  final StreamController<SubmittedModel> _modelController = 
      StreamController<SubmittedModel>();

  Stream<SubmittedModel> get modelStream => 
      _modelController.stream;

  SubmittedModel _model = SubmittedModel();

  void dispose() {
    _modelController.close();
  }

  Future<void> submit() async {
    updateWith(submitted: true);
    try {
      await service.submit("<some buisness logic>");
    } catch (e) {
      updateWith(submitted: false);
      rethrow;
    }
  }

  void updateWith({
    bool? submitted,
  }) {
    _model = SubmittedModel(
      submitted: submitted ?? _model.submitted,
    );
    _modelController.add(_model);
  }
}
```

Building a widget which delegates buisness logic to a bloc:

```dart
class MyForm extends StatelessWidget {
  MyForm({Key? key, required this.bloc}) : super(key: key);
  final SubmittedBloc bloc;

  static Widget create(BuildContext context) {
    return Provider<SubmittedBloc>(
      create: SubmittedBloc.create,
      child: Consumer<SubmittedBloc>(
        builder: (_, bloc, __) => MyForm(bloc: bloc),
      ),
      dispose: (_, bloc) => bloc.dispose(),
    );
  }

  void _submit() {
    try {
      await bloc.submit();
      // UI updates on success.
    } catch (e) {
      // UI exception handling.
    }
  }

  //...
  @override
  Widget build(BuildContext context) {
    return StreamBuilder<SubmittedModel>(
      stream: bloc.modelStream,
      initialData: SubmittedModel(),
      builder: (context, snapshot) {
        return // ...
            onPressed: snapshot.submitted ? _submit : null,
            // ...
      }
    );
  }
}
```

## Value Notifier

`ValueNotifier` is simpler than a BLoC, but does not handle services. It is a specialized implementation of `ChangeNotifier` meant for simple values.

We can configure a widget with a `ValueNotifier` and `ChangeNotifierProvider`.

```dart
class MyForm extends StatelessWidget {
  const MyForm({Key? key, required this.submitted}) 
    : super(key: key);

  final bool submitted;

  static Widget create(BuildContext context) {
    return ChangeNotifierProvider<ValueNotifier<bool>>(
      create: (_) => ValueNotifier<bool>(model: false),
      child: Consumer<ValueNotifier<bool>>(
        builder: (_, model, __) => MyForm(
          model: model,
        ),
      ),
    );
  }

  //...
  @override
  Widget build(BuildContext context) {
    return  // ...
            onPressed: submitted ? _submit : null,
            // ...
      }
    );
  }
}
```

## Change Notifier

A BLoC uses streams of immutable models, but `ChangeNotifier` is used with a mutable model. Like BLoCs, change notifiers may interact with services.

Change notifiers are mixed in with a model:

```dart
class SubmittedModel with ChangeNotifier {
  SubmittedModel({
    required this.service, 
    this.submitted = false,
  });

  final MyService service;
  bool submitted;

  Future<void> submit() async {
    updateWith(submitted: true);
    try {
      await service.submit("<some buisness logic>");
    } catch (e) {
      updateWith(submitted: false);
      rethrow;
    }
  }
  
  void updateWith({bool? submitted}) {
    this.submitted = submitted ?? this.submitted;

    notifyListeners();
  }
}
```

We can configure a widget with a `ChangeNotifier` and `ChangeNotifierProvider`.

```dart
class MyForm extends StatelessWidget {
  const MyForm({Key? key, required this.model}) 
    : super(key: key);

  final SubmittedModel model;

  static Widget create(BuildContext context) {
    final service = Provider.of<MyService>(
      context, 
      listen: false,
    );
    return ChangeNotifierProvider<SubmittedModel>(
      create: (_) => SubmittedModel(
        service: service, 
        submitted: false,
      ),
      child: Consumer<SubmittedModel>(
        builder: (_, model, __) => MyForm(
          model: model,
        ),
      ),
    );
  }

  void _submit() {
    try {
      await model.submit();
      // UI updates on success.
    } catch (e) {
      // UI exception handling.
    }
  }

  //...
  @override
  Widget build(BuildContext context) {
    return  // ...
            onPressed: model.submitted ? _submit : null,
            // ...
      }
    );
  }
}
```
