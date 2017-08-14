- [Introduction](#sec-1)
  - [Installation](#sec-1-1)
  - [Check installation](#sec-1-2)
- [Correctness follows from expressive](#sec-2)


# Introduction<a id="sec-1"></a>

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

-   Define a `map` function ((a -> b), [a]) -> [a]
-   Define a `keep_if` function ((a -> b), [a]) -> [a]

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

| [1 | 2  | 3  | 4   | 5  | 6  | 7  | 8  | 9]  |
| [1 | 4  | 9  | 16  | 25 | 36 | 49 | 64 | 81] |
| [1 | 4  | 9  | 16  | 25 | 36 | 49 | 64 | 81] |
| [4 | 16 | 36 | 64] |    |    |    |    |     |
| [4 | 16 | 36 | 64] |    |    |    |    |     |
