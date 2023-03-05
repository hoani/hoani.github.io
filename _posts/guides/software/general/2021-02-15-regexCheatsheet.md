---
title: "Regex Cheatsheet"
excerpt: "Regex reference with Python/Cpp examples"
permalink: /guides/software/regex
toc: true
categories:
  - guide
  - python
  - cpp
  - cpp
  - software
---

Regex tester: [regextester.com](https://www.regextester.com/)

## Syntax

A regular expression is often delimited by two forward slashes and may be followed by modifiers:
```js
/regex/i 
```

### Literal Characters
```js
/hello/       // matches `hello`
/hello/i      // case-insensitive match to `hello`
/hello/g      // match all instances of `hello`
```

### Metacharacter Symbols
```js
/^h/i;         // ^  - Must start with
/orld$/i;      // $  - Must end with
/^hello$/i;    // ^$ - Must start and end with
/h.llo/i;      // .  - Matches any ONE character
/h*w/i;        // *  - Matches any character zero + times
/gre?a?y/i;    // ?  - optional character, can exist or not
               //      (valid inputs: Gry, Grey, Gray, Greay)
/gre?a?y\?/i;  // \  - escape characters
```

### Brackets [] - Character sets
```js
/gr[ae]y/i;    // must be an a or e only
/[GF]ray/;     // must be a G or F only
/[^GF]ray/;    // Match anything except G or F (^ == NOT)
/[A-Z]ray/;    // Match any uppercase letter
/[a-z]ray/;    // Match any lowercase letter
/[A-Za-z]ray/; // Match any letter
/[0-9]ray/;    // Match any digit
```

### Braces {} - Quantifiers
```js
re = /Hel{2}o/i;    // Match exactly {N} times 
re = /Hel{2,4}o/i;  // Match within the range times 
re = /Hel{2,}o/i;   // Match at least {N} times 
```

### Parentheses () - Grouping
```js
re = /([0-9]x){3}$/;
```

### Shorthand character classes
```js
re = /\w/;          // \w - Word character (A-Z, a-z, 0-9, _)
re = /^\w+$/;       // Match word characters only
re = /\W/;          // \W - Non-word character
re = /\d/;          // \d - Any digit (0-9)
re = /\d+/;         // \d - Any digit multiple times
re = /\D/;          // \d - Any non-digit
re = /\s/;          // \s - Any whitespace character
re = /\S/;          // \S - Any non-whitespace character
re = /Hell\b/i;     // \b - Word boundary - word ends at \b
```

### Assertions
```js
re = /x(?=y)/;      // Match x only if it is followed by y
re = /x(?!y)/;      // Match x only if it is not followed by y
```

## C++ Regex

Define a regular expression with `regex`:
```cpp
std::regex myRegEx("hello", std::regex_constants::icase)
```

Determine whether an expression matches with `regex_search`:
```cpp
#include <iostream>
#include <string>
#include <regex>

int main() {
  std::regex hexRegex("^0x[\\da-f]{1,}", 
                      std::regex_constants::icase);
  
  std::vector<std::string> items = {
    "0x1234ab56e7",
    "0x1F",
    "0x",
    "1F"
  };
 
  for (auto item: items) {
    if (std::regex_search(item, hexRegex)) {
        std::cout << item << " is valid hex\n";
   	}
  }
  return 0;
}
```

Output:
```sh
0x1234ab56e7 is valid regex
0x1F is valid regex
```

The [regex library](https://en.cppreference.com/w/cpp/regex) has a few other nice features, like regex iterators and replace tools.

## Python Regex

Define a regular expression with `re.compile`:
```cpp
re.compile("hello", re.I)
```

Determine whether an expression matches with `re.match`:
```py
import re

hex_regex = re.compile("^0x[\\da-f]{1,}", re.I)
items = [
    "0x1234ab56e7",
    "0x1F",
    "0x",
    "1F"
]

for item in items:
    if re.search(hex_regex, item) is not None:
        print(item, "is valid regex")
```

Output:
```sh
0x1234ab56e7 is valid regex
0x1F is valid regex
```

Like the C++ library, the python [re library](https://docs.python.org/3/library/re.html) has a bunch of useful ways of searching through strings.

