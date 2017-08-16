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
- [Lambda Expression](#sec-5)
  - [Example](#sec-5-1)
- [Problem with comments](#sec-6)
- [High level expressiveness](#sec-7)
- [Currying](#sec-8)
  - [Another Example](#sec-8-1)


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
  vector<int> prod_result{1, 2, 3, 4, 5, 6, 7, 8, 9};
  print_vec(prod_result);

  /// my transform function
  auto squared = my_transform(square_fn<int>, prod_result);
  print_vec(squared);

  /// fplus function
  auto fplus_squared = fplus::transform(square_fn<int>, prod_result);
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
    of a sequence of sequences and concatenates the prod_result.
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
  int prod_result;
  std::istringstream(str) >> prod_result;
  return prod_result;
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
  int prod_result;
  std::istringstream(str) >> prod_result;
  return prod_result;
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

# Lambda Expression<a id="sec-5"></a>

In C++, a lambda function can be expressed as `[capture](parameters) {body}`

```C++
int val = 10;
const auto add_val = [val](int x) -> int { return x + val; };
std::cout << add_val(10) << std::endl;
val = 15;
std::cout << add_val(20) << std::endl;
```

    20
    30

```C++
int val = 10;
const auto add_val = [&val](int x) -> int { return x + val; };
std::cout << add_val(10) << std::endl;
val = 15;
std::cout << add_val(20) << std::endl;
```

    20
    35

## Example<a id="sec-5-1"></a>

Given

polygon : [(Int, Int)]

Find the longest edge

```C++
#include <fplus/fplus.hpp>
#include <iostream>
#include <vector>

using point = std::pair<float, float>;

float get_dist(const point& a, const point& b) {
  auto square = [](auto val) { return val * val; };
  const auto x_diff = a.first - b.first;
  const auto y_diff = a.second - b.second;

  return std::sqrt(square(x_diff) + square(y_diff));
}

int main() {
  std::vector<point> polygon{{1, 2}, {7, 3}, {6, 5}, {4, 4}, {2, 9}};
  std::cout << fplus::show(polygon) << std::endl;

  const auto each_pairs = fplus::overlapping_pairs_cyclic(polygon);
  std::cout << fplus::show(each_pairs) << std::endl;

  const auto max_distance = fplus::maximum_on(
      [](const auto& point_pair) -> float {
        return get_dist(point_pair.first, point_pair.second);
      },
      each_pairs);
  std::cout << fplus::show(max_distance) << std::endl;

  return 0;
}
```

    [(1, 2), (7, 3), (6, 5), (4, 4), (2, 9)]
    [((1, 2), (7, 3)), ((7, 3), (6, 5)), ((6, 5), (4, 4)), ((4, 4), (2, 9)), ((2, 9), (1, 2))]
    ((2, 9), (1, 2))

# Problem with comments<a id="sec-6"></a>

When code changes, comments should change manually. It's not good. So, make the code self-document instead.

For example, the below code is a simple example..

```C++
#include <fplus/fplus.hpp>
#include <iostream>
#include <sstream>

int str2int(const std::string& str) {
  int ret;
  std::istringstream(str) >> ret;
  return ret;
}

int main() {
  const std::string input{"1,5,4,7,2,2,3"};
  const auto str_list = fplus::split(',', false, input);
  const auto int_list = fplus::transform(str2int, str_list);
  // the following line sums up the list
  const auto prod_result = fplus::reduce(std::plus<int>(), 0, int_list);
  std::cout << fplus::show(prod_result) << std::endl;
}
```

    24

It can be refactored to make it more self-documenting.

```C++
auto sum(const std::vector<int>& containers) {
  return fplus::reduce(std::plus<int>(), 0, containers);
}
auto prod(const std::vector<int>& containers) {
  return fplus::reduce(std::multiplies<int>(), 1, containers);
}
int main() {
  const std::vector<int> nums{1, 2, 3, 4};
  const auto sum_result = sum(nums);
  std::cout << fplus::show(sum_result) << std::endl;

  const auto prod_result = prod(nums);
  std::cout << fplus::show(prod_result) << std::endl;
  return 0;
}
```

    10
    24

# High level expressiveness<a id="sec-7"></a>

Let's refactor the previous polygon distance function.

Given a set of points (polygon), find the longest edge.

```C++
typedef std::pair<float, float> Point;
typedef std::vector<Point> Polygon;

auto get_distance = [](const Point& a, const Point& b) -> float {
  auto square = [](auto val) { return val * val; };
  auto x_diff = a.first - b.second;
  auto y_diff = a.second - b.second;
  return std::sqrt(square(x_diff) + square(y_diff));
};
auto edge_length =
    [&get_distance](const std::pair<Point, Point>& two_points) -> float {
  return get_distance(two_points.first, two_points.second);
};
auto get_edges = [](const Polygon& polygon) {
  return fplus::overlapping_pairs_cyclic(polygon);
};

Polygon polygon{{1, 2}, {7, 3}, {6, 5}, {4, 4}, {2, 9}};
const auto result = fplus::maximum_on(edge_length, get_edges(polygon));

std::cout << "Longest edge: " << fplus::show(edge_length(result))
          << ", Two points are " << fplus::show(result) << std::endl;
```

    Longest edge: 7.07107, Two points are ((4, 4), (2, 9))

# Currying<a id="sec-8"></a>

Suppose there is a function, `add : (Int, Int) -> Int`

```C++
std::function<int(int, int)> add(auto first, auto second) { return first + second; };
```

we can create a function `add_3 : Int -> Int`

```C++
std::function<std::function<int(int)>> add_3(auto second) {
  return add(3, second);
};
```

Here is an example:

```C++
auto add_curried = [](auto first) {
  return [first](auto second) { return first + second; };
};
std::cout << add_curried(8)(5) << "\n";
```

    13

Notice `add_curried : Int -> (Int -> Int)` then it is same as `Int -> Int -> Int`.

The following code does not work because

-   `transform : ((a -> b), [a]) -> [b]`
-   `keep_if : ((a -> Bool), [a]) -> [a]`

```C++
constexpr bool is_even(int x) {
  return x % 2 == 0;
}

std::vector<std::vector<int>> xss{{0, 1, 2}, {3, 4, 5}};
auto result = fplus::transform(keep_if(is_even), xss);
```

We need some currying here.

Something like this

`keep_if_curry : (a -> Bool) -> [a] -> [a]` `transform_curry : (a -> b) -> [a] -> [b]`

You can use a lambda function to define it. Or, there is a `fplus::fwd` such as

`fwd::keep_if : (a -> Bool) -> [a] -> [a]`

```C++
using namespace fplus;
auto is_even = [](int val) -> bool { return val % 2 == 0; };
std::vector<std::vector<int>> xss{{0, 1, 2}, {3, 4, 5}};

auto result = transform(fwd::keep_if(is_even), xss);
std::cout << show(result) << std::endl;

auto square = [](int val) {return val * val; };
result = transform(fwd::transform(square), xss);
std::cout << show(result) << std::endl;

```

    [[0, 2], [4]]
    [[0, 1, 4], [9, 16, 25]]

Using purely STL,

```C++
#include <algorithm>
#include <iostream>
#include <iterator>
#include <vector>

template <typename Container, typename Function>
Container transform(Function f, const Container& input) {
  Container out;
  std::transform(std::begin(input), std::end(input), std::back_inserter(out),
                 f);
  return out;
}

template <typename Function>
auto transform_fwd(Function f) {

  return [&](const auto& input) {
    return transform(f, input);
  };
}

template <typename T>
void show(const std::vector<T>& input) {
  int i = 0;
  std::cout << "[";
  for (const T& elem : input) {
    if (i == input.size() - 1) {
      std::cout << elem;
    } else {
      std::cout << elem << ", ";
    }
    i++;
  }
  std::cout << "]";
}

template <typename T>
void show(const std::vector<std::vector<T>>& input) {
  for (const std::vector<T>& elem : input) {
    show(elem);
  }
}

int main() {
  std::vector<std::vector<int>> xss{{0, 1, 2}, {3, 4, 5}};
  show(xss);
  std::cout << "\n-------------------\n";
  std::cout << "\nChange to \n";
  std::cout << "\n-------------------\n";

  auto square = [](int val) { return val * val; };
  auto result = transform(transform_fwd(square), xss);

  show(result);

  return 0;
}
```

    [0, 1, 2][3, 4, 5]
    -------------------
    
    Change to 
    
    -------------------
    [0, 1, 4][9, 16, 25]

## Another Example<a id="sec-8-1"></a>

Find a fully curried version of add<sub>four</sub>

`add_four : Int -> Int -> Int -> Int`

```C++
auto add_four = [](auto first) {
  return [first](auto second) {
    return [first, second](auto third) {
      return [first, second, third](auto fourth) {
        return first + second + third + fourth;
      };
    };
  };
};

std::cout << add_four(1)(2)(3)(4) << std::endl;
```

    10
