## C++ Vectors

Define a new vector:
```cpp
std::vector<int> my_vector;
std::vector<int> my_vector(10); // With inital allocation
```

Get data from vector:
```cpp
int value = my_vector[index];
int value = my_vector.at(index);
```

Pushing and popping (LIFO):
```cpp
my_vector.push_back(next_value);
my_vector.pop_back(); // Note: does not return anything
```

Check if the vector has items:
```cpp
if (my_vector.empty()) { // ...
```

### Example

[cpp.sh/8ntss](try on cpp.sh)

```cpp
#include <iostream>
#include <vector>
#include <string>

int main() {
	std::vector<std::string> strings;

	strings.push_back("Hello Hoani");
	strings.push_back("Hello Emma");
	strings.push_back("Hello Indie");

	for (std::string str: strings) {
		std::cout << str << std::endl;
	}

	std::cin.get();
}
```
