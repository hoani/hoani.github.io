---
title: "Sequencing Engine"
excerpt: "A Pattern for Sequencing Game Events"
permalink: /guides/game-dev/sequencer-pattern
toc: true
categories:
  - guide
  - software
  - gamedev
---

When making [Truly Madly Deeply](/truly-madly-deeply) and [Final Scoundrel](/final-scoundrel), there were several events where animations, dialog, sound effects or some other sequence occurs which:
* Changes the way the player interacts with the game
* Must be handled in a structured way

## Final Scoundrel Sequencing

For Final Scoundrel, I handled this all via a textbox object - which parsed and handled sequences of events - including ones which didn't write to the textbox. This handled all sorts of events, for example:
* monsters appearing is a sequence handled by the text box
* fighting is a set of sequences with forking logic depending on whether you die or the monster dies
* picking up a potion is a set of sequences, which will show different text if you are already at full health
* the tutorial is... you guessed it, a set of sequences that run inbetween gameplay elements

This sequencing worked okay, but it was quite messy, and had a lot of awkward branching logic to optionally choose various sequences.

## Actions, Options, Sequences and Interactions

According to the Google Doc for Truly Madly Deeply, it has around 15000 words and over 100 different cutscenes. 

To manage all of this, I came up and engine with four parts:
* `Interaction` is a graph of `Sequences`
* `Sequences` are a list of `Actions`, and may terminate with an `Option`
* `Actions` define things that happen in game 
  * e.g. such as a character appearing, speech, sound effects, location changes etc.
* `Option` determines traversal to other `Sequences` in an `Interaction`
  * e.g. player decision, money check, minigame outcomes

### Actions, Options and context

An `action` has the following interface (note: written in GML):

```
function Action() constructor {
    static Spawn = function(ctx, callback) {
        callback();
    }
}
```

Similarly, an `option` is defined as:

```
function Option() constructor {
    static Outcomes = function() {
      // Options must have at least one possible outcome.
      return 1; 
    }
    static Spawn = function(ctx, callback) {
        callback(0); // Note: Empty option.
    }
}
```

The heart of `Actions` and `Options` is the `ctx` object which is passed to them. This gives them access to the game's UI elements, game state, HUD elements and more. The Action simply needs to do something like trigger the textbox and have the textbox call the `callback` when the user is done with the textbox.

For example, a more complex `Action` which makes use of the `ctx` looks like:
```
function ActionTradeSell() : Action() constructor {
    static Spawn = function(ctx, callback, isLast=false) {
        var resource = ctx.cycle.work.mine.resource;
        var price = TradeCalculatePrice(ctx);
        
        for (var i = 0; i < array_length(resource); i++) {
            resource[i] = 0;
        }
        
        var cc = new ConcurrentCallback(2, callback);
        
        ctx.signal.AddMoney(price, cc.Call);
        ctx.signal.UpdateMinerals(cc.Call)
    }
}
```

### Sequence and Interaction Graphs

A simple `Sequence` with no branching logic looks like this:
```
{
  script: [
    Speech("Foo", "Hello World"),
    Speech("Foo", "Good bye"),
  ],
  next: 0,
}
```

But a `Sequence` with branching logic looks like:
```
{
  script: [ Narrate("You see an ogre") ],
  option: Choice(["Say Hello", "Run away"]),
  next: ["greet", "leave"],
}
```

Note: in these examples, `Narrate` and `Speech` implement the `Action` interface and `Choice` implements the `Option` interface.

An `Interaction` is a graph of `Sequences`, for example:

```
{
  id: "find_ogre",
  entry: "ogre",
  sequences: {
    ogre: {
      script: [ Narrate("You see an ogre") ],
      option: Choice(["Say Hello", "Run away"]),
      next: ["greet", "leave"],
    },
    greet: {
      script: [ Speech("Hello Ogre") ],
      next: 0,
    }
    leave: {
      script: [ Narrate("You escaped") ],
      next: 1,
    }
  }
}
```

The numeric `next` values are return indexes from the interaction - `interactions` are called with a `callback` which this index is passed to on completion, which determines what to do once the interaction has finished.

I've added an example of this pattern from Truly Madly Deeply below.

```
{
  id: EvTradeIntro,
  entry: "enter",
  sequences: {
    enter: {
      script: [
        ClearFlag(FlagTrade),
        HudHide(),
        Location(LOCATION_MINE_EVENING).Music(musTrade),
        Outfit(EMCEE, OutfitEmceeMine),
        Outfit(MERCHANT, OutfitMerchantTrader),
        Enter(ELI, POS_RIGHT).Emotion(xNEU),
        Speech(ELI, "Miss Marceline!!!").Emotion(xNEU),
        Enter(EMCEE, POS_LEFT).Emotion(xSUR),
        Speech(EMCEE, "...!").Emotion(xSUR),
        Speech(EMCEE, "BOSS?!!"),
        Speech(ELI, "A-hem! Oh, Miss Marceline...").Emotion(xWRM),
        Speech(ELI, "...you flatter me..."),
        Narrate("...?").Emotion(xSUR),
        Speech(ELI, "For, toil as I might...").Emotion(xCRY),
        Speech(ELI, "I am still, merely, my father's son..."),
        Speech(ELI, "And so, for all intents and purposes...").Emotion(xSAD),
        Speech(ELI, "(And in accordance with Statute UA167-B3...)").Emotion(xSUR),
        Speech(ELI, "I'm still more of a...").Emotion(xBLU),
        Speech(ELI, "Son-of-a-Boss!").Emotion(xWRM),
        Narrate("For 'all intents and purposes', it means the same for me...").Emotion(xBLU),
        Speech(EMCEE, "...Boss, I--").Emotion(xSUR),
        Speech(ELI, "Please, Miss Marceline!").Emotion(xNEU),
        Speech(ELI, "Out here, you may simply call me...").Emotion(xWRM),
        Speech(ELI, "'Eli'...!").Emotion(xSMU),
        Speech(ELI, "And 'out here' is certainly where you are!"),
        Speech(EMCEE, "...?").Emotion(xBLU),
        Speech(ELI, "I must confess, I--").Emotion(xBLU),
        Speech(ELI, "Miss Marceline, where is 'here', exactly?").Emotion(xSUR),
        Speech(EMCEE, "...?").Emotion(xSUR),
        Speech(ELI, "This place.").Emotion(xNEU),
        Speech(ELI, "It's name, if you will."),
        Speech(EMCEE, "It's Talley Town, Boss.").Emotion(xBLU),
        Emotion(ELI, xBLU).Confirm(),
        Speech(ELI, "Please, Miss Marceline.").Emotion(xWRM),
        Speech(ELI, "It's Eli.").Emotion(xHAP),
        Speech(ELI, "I do insist.").Emotion(xSMU),
        Speech(EMCEE, "...").Emotion(xBLU),
        Narrate("...").Emotion(xSUR),
        Speech(ELI, "...").Emotion(xSAD),
        Narrate("*sigh*").Emotion(xBLU),
        Speech(EMCEE, "It's Talley Town... Eli.").Emotion(xNEU),
        Speech(ELI, "TALLEY TOWN!!!").Emotion(xHAP),
        Speech(ELI, "Delightful! I'm sure, delightful...").Emotion(xSMU),
        Speech(ELI, "A-hem.").Emotion(xSUR),
        Emotion(EMCEE, xBLU).Confirm(),
        Speech(ELI, "But -- Miss Marceline--").Emotion(xBLU),
        Speech(ELI, "--And please do forgive me if I'm mistaken--").Emotion(xSUR),
        Speech(ELI, "Does this mean that you handed in your resignation..."),
        Speech(ELI, "-- without warning, I might add --").Emotion(xBLU),
        Speech(ELI, "And tossed aside a promising corporate career...").Emotion(xSUR),
        Speech(ELI, "All to come to this...").Emotion(xBLU),
        Speech(ELI, "Tarrey... town?").Emotion(xCRY),
        Speech(EMCEE, "Talley. T-A-L-L-E-Y - ").Emotion(xSUR),
        Speech(ELI, "...!").Emotion(xBLU),
        Speech(ELI, "Miss Marceline...!!").Emotion(xSUR),
        Speech(ELI, "This town... it...").Emotion(xBLU),
        Speech(ELI, "...it doesn't have streetlamps!!!"),
        Emotion(EMCEE, xBLU).Confirm(),
        Speech(ELI, "Did you even consider how you would stay safe,\n...walking home alone at night...!").Emotion(xCRY),
        Speech(EMCEE, "...?").Emotion(xSUR),
        Emotion(EMCEE, xBLU).Confirm(),
        Emotion(ELI, xBLU).Confirm(),
        Emotion(ELI, xSUR).Confirm(),
        Speech(ELI, "...And, Miss Marceline!"),
        Speech(ELI, "Were you aware that there is but one,").Emotion(xCRY),
        Speech(ELI, "woeful eating establishment here...?"),
        Speech(ELI, "'The Crow's Perch', they call it!").Emotion(xSAD),
        Speech(ELI, "...And it serves nothing but PIE!").Emotion(xCRY),
        Emotion(EMCEE, xSUR).Confirm(),
        Emotion(EMCEE, xBLU).Confirm(),
        Speech(ELI, "Miss Marceline, I must beg--").Emotion(xSAD),
        Speech(ELI, "--Nay--"),
        Speech(ELI, "PLEAD you to reconsider...!").Emotion(xCRY),
        Narrate("Thirteen years of my life..."),
        Narrate("...working fourteen-hour days...").Emotion(xNEU),
        Narrate("...sitting in a cubicle...").Emotion(xSAD),
        Narrate("Day in... day out..."),
        Narrate("Not a single word of thanks."),
        Narrate("Nor a single, measly bonus!").Emotion(xSUR),
        Narrate("But I hand in my resignation, and then 'Eli' follows me home to beg for my return...?").Emotion(xBLU),
        Speech(EMCEE, "I'll have you know, Eli, that I grew up in this town.").Emotion(xSUR),
        Speech(EMCEE, "And I quite like their pies.").Emotion(xSMU),
        Narrate("Just not the blackberry ones...").Emotion(xBLU),
        Emotion(EMCEE, xSMU).Confirm(),
        Speech(ELI, "...?").Emotion(xSUR),
        Speech(ELI, "...?"),
        Speech(ELI, "But...!").Emotion(xBLU),
        Emotion(ELI, xSUR).Confirm(),
        Speech(EMCEE, "Besides. I have secured a loan from the bank,").Emotion(xHAP),
        Speech(EMCEE, "Put a down payment on a mine, and after a single day,"),
        Speech(EMCEE, "I now have freshly-mined ore in my hot little hands.").Emotion(xWRM),
        Speech(ELI, "...!").Emotion(xBLU),
        Speech(ELI, "...?").Emotion(xSUR),
        Speech(ELI, "... A mine, you say...?").Emotion(xNEU),
        Speech(ELI, "...").Emotion(xSMU),
        Emotion(EMCEE, xSUR).Confirm(),
        Speech(ELI, "...A-hem."),
        Speech(ELI, "...And how might you be intending to SELL that ore to pay off your loan, Miss Marceline?").Emotion(xHAP),
        Narrate("Wait... selling the ore?").Emotion(xSUR),
        Narrate("Can't I just hand over shiny rocks in exchange for cold, hard coin?"),
        Narrate("No, right, I'd need a United Alliance UA24-350-C for that...").Emotion(xBLU),
        Speech(ELI, "Might you require the assistance of...").Emotion(xHAP),
        Speech(ELI, "Say...").Emotion(xNEU),
        Speech(ELI, "A handsome...").Emotion(xWRM),
        Speech(ELI, "...Dashing...!").Emotion(xHAP),
        Speech(ELI, "...Intelligent...!!").Emotion(xCRY),
        Speech(ELI, "Son-of-a-Boss...?").Emotion(xSMU),
        Speech(EMCEE, "I--").Emotion(xSUR),
        Narrate("Please, Emcee!!!\nBefore you say no, think of the consequences!"),
        Speech(EMCEE, "...").Emotion(xBLU),
        HudSwitch(HudLayoutTradeSell),
        Speech(EMCEE, "... Fine."),
        Speech(ELI, "Let's see, for this ore, I can offer thee..."),
      ],
      option: Choice([TradeSellRenderer()]),
      next: ["sell"],
    },
    sell: {
      script: [
        new ActionTradeSell(),
      ],
      next: "leave",
    },
    leave: {
      script: [
        Speech(ELI, "Excellent!").Emotion(xHAP),
        Speech(ELI, "Superb!!"),
        Speech(ELI, "Simply... delightful!!!"),
        Speech(ELI, "Goodbye, Miss Marceline!").Emotion(xWRM),
        Speech(ELI, "Fare-thee-well!").Emotion(xHAP),
        HudHide(),
        Exit(ELI),
        Speech(ELI, "And I promise, I shall humbly grace you and your mine with my presence again forthwith!").Emotion(xHAP),
        Narrate("Huh.").Emotion(xSUR),
        Narrate("Annoyingly, that worked out fairly well...").Emotion(xBLU),
      ],
      next: 0,
    },
  }
}
```
The action's interface is very flexible; in some cases I've used a creational design pattern to customize or simplify actions. 

For example, I can both have a character enter, but also specify thier emotion as they enter. Or change the music as the location changes.
