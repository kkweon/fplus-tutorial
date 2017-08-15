- [Introduction](#sec-1)
  - [Installation](#sec-1-1)
  - [Check installation](#sec-1-2)
- [Correctness follows from expressive](#sec-2)
- [Type annotations](#sec-3)
  - [Variables](#sec-3-1)
  - [map](#sec-3-2)
  - [Function](#sec-3-3)
  - [Why useful?](#sec-3-4)
- [Parse and Product](#sec-4)


# Introduction<a id="sec-1"></a>

All of stuff come from

-   [Udemy Functional Programming C++](https:www.udemy.com/functional-programming-using-cpp)
-   [Functional Plus](https:github.com/Dobiasd/FunctionalPlus)

## Installation<a id="sec-1-1"></a>

-   Go to <https://github.com/Dobiasd/FunctionalPlus>
-   Follow the instruction

## Check installation<a id="sec-1-2"></a>

```C++
#include <fplus/fplus.hpp>
#include <iostream>

int main() {
  std::list<std::string> things = {"same old", "same old"};
  if (fplus::all_the_same(things))
    std::cout << "All things being equal." << std::endl;
}
```

    All things being equal.

```C++
#include <fplus/fplus.hpp>
#include <iostream>

int main() {
  std::string team = "Our team is great. I love everybody I work with.";
  std::cout << "There actually are this many 'I's in team: "
            << fplus::count("I", fplus::split_words(false, team)) << std::endl;
}
```

    There actually are this many 'I's in team: 2

# Correctness follows from expressive<a id="sec-2"></a>

-   Define a `map` function ((a -> a), [a]) -> [a]
-   Define a `keep_if` function ((a -> Bool), [a]) -> [a]

```C++
#include <algorithm>
#include <fplus/fplus.hpp>
#include <iostream>
#include <vector>

using namespace std;

struct square {
  template <typename T> T operator()(T val) { return val * val; }
};

template<typename T>
T square_fn(T val) {
  return val * val;
}

template <typename Function, typename T>
vector<T> my_transform(Function f, const vector<T> &containers) {
  vector<T> ret;
  transform(begin(containers), end(containers), back_inserter(ret), f);
  return ret;
}

template <typename Container> void print_vec(const Container &container) {
  cout << "[";
  int i = 0;
  for (auto &val : container) {
    if (i == container.size() - 1) {
      cout << val;
    } else {
      cout << val << ", ";
    }
    i++;
  }
  cout << "]\n";
}

template<typename Function, typename T>
vector<T> keep_if(Function f, const vector<T>& container) {
  vector<T> ret;
  ret.reserve(container.size());
  std::copy_if(begin(container), end(container), back_inserter(ret), f);
  return ret;
}


int main() {
  vector<int> result{1, 2, 3, 4, 5, 6, 7, 8, 9};
  print_vec(result);

  /// my transform function
  auto squared = my_transform(square_fn<int>, result);
  print_vec(squared);

  /// fplus function
  auto fplus_squared = fplus::transform(square_fn<int>, result);
  print_vec(fplus_squared);

  auto is_even = [](auto val) {
    return val % 2 == 0;
  };

  /// my_keep_if function
  print_vec(keep_if(is_even, squared));

  /// fplus_keep_if function
  print_vec(fplus::keep_if(is_even, squared));

  return 0;
}
```

    [1, 2, 3, 4, 5, 6, 7, 8, 9]
    [1, 4, 9, 16, 25, 36, 49, 64, 81]
    [1, 4, 9, 16, 25, 36, 49, 64, 81]
    [4, 16, 36, 64]
    [4, 16, 36, 64]

# Type annotations<a id="sec-3"></a>

Type annotations are used to browse the [FunctionalPlus API Search](http://www.editgym.com/fplus-api-search/).

-   Type variables are written in lower case such as `a`, `b`, &#x2026;
-   Fixed types starts with a Uppercase letter such as `Int`, `String`, `Bool`

## Variables<a id="sec-3-1"></a>

Variables can be expressed in the following way

```C++
int x;              // x : Int
vector<int> xs;     // xs: [Int]
pair<int, float> p; // p : (Int, Float)
```

## map<a id="sec-3-2"></a>

In C++, `map` is like an `dict` in Python

```C++
map<string, int> dict; // dict : Map String Int
```

## Function<a id="sec-3-3"></a>

```C++
int foo(string value);                              // foo : String -> Int
ContainerIn transform(F f, const ContainerOut& xs); // transform : ((a -> b), [a]) -> [b]
```

Previous `keep_if` can be written as

```C++
Container keep_if(F f, Container& container); // keep_if : ((a -> Bool), [a]) -> [a]
```

## Why useful?<a id="sec-3-4"></a>

Suppose I want a `concat` function as below.

```C++
concat(["Bar", "Baz", "Buz"], ";") == "bar;baz;buz"
```

We know

```C++
concat(vector<string> container, string delim); // 
```

If we type the following annotation in the API browser, `(vector<string>, string)->string`

we get

    (vector<string>,string)->string
    as parsed type: ([String], String) -> String
    ---------------------------------------------------------
    join : ([a], [[a]]) -> [a]
    fwd::join : [a] -> [[a]] -> [a]
    Inserts a separator sequence in between the elements
    of a sequence of sequences and concatenates the result.
    Also known as intercalate.
    join(", ", "["a", "bee", "cee"]) == "a, bee, cee"
    join([0, 0], [[1], [2], [3, 4]]) == [1, 0, 0, 2, 0, 0, 3, 4]
    
    template <typename Container,
        typename X = typename Container::value_type>
    X join(const X& separator, const Container& xs)

# Parse and Product<a id="sec-4"></a>

Given

-   a string of "1, 2, 3, 4, 5, 6"
-   return the int value of 1 \* 2 \* 3 \* 4 \* 5 \* 6

```C++
#include <fplus/fplus.hpp>
#include <string>
#include <iostream>
#include <sstream>

int str2int(const std::string& str) {
  int result;
  std::istringstream(str) >> result;
  return result;
}
int main() {
  const std::string input {"1, 5, 4, 7, 2, 2, 3"};
  const auto str_list = fplus::split(',', false, input);
  const auto int_list = fplus::transform(str2int, str_list);
  std::cout << fplus::show(int_list) << "\n";

  const auto product = fplus::reduce(std::multiplies<int>(), 1, int_list);
  std::cout << product << "\n";
}
```

    [1, 5, 4, 7, 2, 2, 3]
    1680

What about addition?

```C++
#include <fplus/fplus.hpp>
#include <string>
#include <iostream>
#include <sstream>

template<typename T>
struct addition {
  constexpr T operator()(T left, T right) {
    return left + right;
  }
};

int str2int(const std::string& str) {
  int result;
  std::istringstream(str) >> result;
  return result;
}

int main() {
  const std::string input {"1, 5, 4"};
  const auto str_list = fplus::split(',', false, input);
  const auto int_list = fplus::transform(str2int, str_list);
  std::cout << fplus::show(int_list) << "\n";

  const auto sum = fplus::reduce(addition<int>(), 0, int_list);
  std::cout << sum << "\n";
}
```

    [1, 5, 4]
    10
