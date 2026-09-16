---
title: "Events Engine"
excerpt: "Managing Triggered Game Events"
permalink: /guides/game-dev/events-engine
toc: true
categories:
  - guide
  - software
  - gamedev
---

In [Truly Madly Deeply](/truly-madly-deeply) there are several cutscenes or interactions that trigger once various condtions are met, such as:
* Spending enough time with a character
* Payments being due
* Running into an enemy for the first time

Keeping track of these events, the conditions to trigger them and determining which event to trigger are all part of the game's event system.

## Event Structure

The constructor for events in Truly Madly Deeply looks like:
```
EventDefinition(name, condition, interaction, allowMultiple=false)
```

The key parts are:
* name - a unique identifier 
  * useful for creating an event registry where we set a flag if an event has already been played
  * also useful for putting together an event zoo as discussed in [zoos](/guides/game-dev/zoos)
* condition - a condition object which is called and returns whether the event is ready
* interaction - the `interaction` defines the event's content, see [sequencing engine](/guides/game-dev/sequencer-pattern)
* allowMultiple - many events are one-offs, but some may be triggered multiple times

Technically `allowMultiple` could be part of `condition`... but I didn't think about that when I first implemented events...

Events have the following interface:
```
bool Ready(ctx)
Spawn(ctx, callback)
```

`Ready(ctx)` takes the game's current context and determines if the event is able to be executed. This is achieved by checking if the `condition` is met, and if `allowMultiple` is `false` we check if the event has ran before.

`Spawn(ctx, callback)` will run the event's `interaction`. This is the same `Spawn(ctx, callback)` interface used for actions in the [sequencing engine](/guides/game-dev/sequencer-pattern), which means that an `event` can be spawned as an `action` within another interaction. For example, in Truly Madly Deeply, some events trigger when a new mine node has been explored. Exploring a mine node is an interaction, which has an action to runs any eligible events before returning control back to the exploring interaction.

## Conditions

An event being ready depends on a condition object.

The barebones condition object has the following interface:

```
bool Check(ctx)
```

`ctx` provides the game state, so as long as you can write conditions which can navigate the gamestate, we can trigger events on whatever we like.

For example, some of the conditions in Truly Madly Deeply looked like:

```
NewConditions($"event.{EvTradeIntro}", $"flag.{FlagTrade}", OnRandom(0.25), "game.bond.merchant > 1")
```

In this case we get 4 conditions:
* The first is a string which requires that the Trade Introduction event has occurred before this event can occur
* The second checks that the flag `Trade` has been set (this is set for each day you mine - except the first day where the character flees from the mine).
* The third condtion `OnRandom(0.25)` provides a custom condition implementation, which is then resolved by calling it's `Check(ctx)` method.
* The fourth checks that a game state value stored in `ctx` meets a conditon - in this case, the merchant likes you enough.

## Event Runner

The final part of the equation is the event runner.

The event runner has a `Spawn(ctx, callback)` method (yes.. this is everywhere).

When `Spawn` is called, it iterates through all of the events in order until it finds a suitable candidate and then it runs it.

If no candidate is found it calls the original `callback`.

If a candidate it found it calls the candidates `Spawn` and passed the `callback` to that event.

The code in Truly Madly Deeply for this runner is:
```
function EventRunner(_definitions, _multievent=true) constructor {
    events = _definitions;
    ctx = noone;
    callback = noone;
    multievent = _multievent
    
    static Spawn = function(_ctx, _callback) {
        ctx = _ctx;
        callback = _callback;
        _RunEvents();
    }
    
    _RunEvents = function() {    
        for (var i = 0; i<array_length(events); i++) {
            var eventDef = events[i];
            if eventDef.Ready(ctx) {
                var cb = multievent ? _RunEvents : callback;
                eventDef.Spawn(ctx, cb)
                return;
            }
        }
        callback();
    }
}
```


