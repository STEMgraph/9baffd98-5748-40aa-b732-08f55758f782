<!---
{
  "id": "9baffd98-5748-40aa-b732-08f55758f782",
  "teaches": "Cpp: Transforming and Reducing with std::transform, std::accumulate and std::for_each",
  "depends_on": [
    "f5737440-0818-442e-8330-396eb85df159",
    "4ad8d26e-1c11-4940-bac3-a9d84e1eb4b6"
  ],
  "author": "Stephan Bökelmann",
  "first_used": "2026-11-19",
  "keywords": [
    "C++",
    "std::transform",
    "std::accumulate",
    "std::for_each",
    "std::back_inserter",
    "std::reduce",
    "no raw loops",
    "g++"
  ]
}
--->

# Cpp: Transforming and Reducing with std::transform, std::accumulate and std::for_each

> In this exercise you turn a range of values into another one with `std::transform`, fold a range into a single value with `std::accumulate`, run a function on every element with `std::for_each`, and replace hand-written loops by these named algorithms. No `sudo` is required.

## Introduction

Most loops over a container do one of three things: they **map** every element to a new value, they **reduce** all elements to one value, or they perform an action on every element. The standard library has an algorithm for each: `std::transform`, `std::accumulate` (in `<numeric>`) and `std::for_each`. They take the operation as a callback, so everything you learned about function pointers and predicates applies.

The idea behind this is often summarized as "no raw loops" (Sean Parent, *C++ Seasoning*): a loop has to be read to be understood, a named algorithm says what it does. It is also less error prone, because the algorithm owns the index handling.

### 1.1) Further Readings and Other Sources

* [cppreference: std::transform](https://en.cppreference.com/w/cpp/algorithm/transform)
* [cppreference: std::accumulate](https://en.cppreference.com/w/cpp/algorithm/accumulate)
* [cppreference: std::for_each](https://en.cppreference.com/w/cpp/algorithm/for_each)
* [Sean Parent: C++ Seasoning (talk)](https://www.youtube.com/watch?v=W2tWOdzgXHA)

## Tasks

### Task 1: Five ways to use `std::transform`

```bash
mkdir -p ~/cpp-transform && cd ~/cpp-transform
```

```c++
// file: transform.cpp
#include <algorithm>
#include <iostream>
#include <iterator>
#include <vector>

int square(int x)     { return x * x; }
int add(int a, int b) { return a + b; }

void print(const char *label, const std::vector<int> &v) {
    std::cout << label;
    for (int x : v) std::cout << " " << x;
    std::cout << std::endl;
}

int main() {
    std::vector<int> v{1, 2, 3, 4, 5};

    // 1) into a second vector that already has the right size
    std::vector<int> squares(v.size());
    std::transform(v.begin(), v.end(), squares.begin(), square);
    print("sized:   ", squares);

    // 2) into an empty vector, growing it with back_inserter
    std::vector<int> grown;
    std::transform(v.begin(), v.end(), std::back_inserter(grown), square);
    print("grown:   ", grown);

    // 3) two input ranges, one output range
    std::vector<int> sums(v.size());
    std::transform(v.begin(), v.end(), squares.begin(), sums.begin(), add);
    print("v + v*v: ", sums);

    // 4) straight to the terminal
    std::cout << "stream:  ";
    std::transform(v.begin(), v.end(), std::ostream_iterator<int>(std::cout, " "), square);
    std::cout << std::endl;

    // 5) in place: the destination is the source
    std::transform(v.begin(), v.end(), v.begin(), square);
    print("in place:", v);
}
```

```bash
g++ -std=c++17 -Wall -Wextra transform.cpp -o transform && ./transform
```

`std::transform` writes through an **output iterator**. If the destination is a vector with the right size, `squares.begin()` is enough. For an empty vector, `std::back_inserter` calls `push_back` for every element. Replace `std::back_inserter(grown)` by `grown.begin()` and run the program (with `-fsanitize=address` you get a readable report): the destination has no elements yet, so you write through the start of an empty vector, and the program crashes. Which variant of the five suits "convert and print without storing"?

### Task 2: Fold with `std::accumulate`

```c++
// file: accumulate.cpp
#include <iostream>
#include <numeric>
#include <vector>

int main() {
    std::vector<int>    ints{1, 2, 3, 4, 5};
    std::vector<double> reals{1.5, 2.5, 3.5};

    std::cout << "sum ints:        " << std::accumulate(ints.begin(), ints.end(), 0) << std::endl;
    std::cout << "sum reals (0):   " << std::accumulate(reals.begin(), reals.end(), 0) << std::endl;
    std::cout << "sum reals (0.0): " << std::accumulate(reals.begin(), reals.end(), 0.0) << std::endl;

    std::vector<int> big{2000000000, 2000000000};
    std::cout << "sum big (0LL):   " << std::accumulate(big.begin(), big.end(), 0LL) << std::endl;

    std::cout << "reduce ints:     " << std::reduce(ints.begin(), ints.end(), 0) << std::endl;
}
```

```bash
g++ -std=c++17 -Wall -Wextra accumulate.cpp -o accumulate && ./accumulate
```

The result type of `std::accumulate` is the type of the **start value**, not the type of the elements. With `0` as start the sum of `1.5 + 2.5 + 3.5` is computed in `int` and gives 6 instead of 7.5, with `0.0` it is right. Likewise `0LL` is needed for the two large numbers, whose sum does not fit into an `int`. `std::reduce` (C++17) gives the same result here, but it may add the elements in any order, so it needs an operation that is associative and commutative. Compute the sum of `{0.1, 0.2, 0.3}` with both, print it with `std::setprecision(17)` (from `<iomanip>`) and compare it to `0.6` with `==`: what does this say about floating point sums?

### Task 3: A different operation

```c++
// file: accumulate_op.cpp
#include <algorithm>
#include <iostream>
#include <numeric>
#include <string>
#include <vector>

int mul(int a, int b)           { return a * b; }
int larger(int a, int b)        { return a > b ? a : b; }
std::string join(const std::string &acc, const std::string &word) {
    return acc.empty() ? word : acc + ", " + word;
}

int main() {
    std::vector<int> v{3, 1, 4, 1, 5};
    std::cout << "product: " << std::accumulate(v.begin(), v.end(), 1, mul) << std::endl;
    std::cout << "maximum: " << std::accumulate(v.begin(), v.end(), v.front(), larger) << std::endl;

    std::vector<std::string> words{"alpha", "beta", "gamma"};
    std::cout << "joined:  " << std::accumulate(words.begin(), words.end(), std::string(), join) << std::endl;

    // The result type is the type of the start value, not of the elements:
    // std::accumulate(words.begin(), words.end(), "", join);   // does not compile
}
```

```bash
g++ -std=c++17 -Wall -Wextra accumulate_op.cpp -o accumulate_op && ./accumulate_op
```

`std::accumulate` takes an optional fourth argument: a function of two parameters, the accumulated value so far and the next element. With `mul` it is a product, with `larger` a maximum, with `join` a string builder. Remove the comment from the last line and read the error: the start value `""` is a `const char *`, so the compiler cannot store the string result in it. Write a callback `shorter` that returns the shortest word of `words`.

### Task 4: An action for every element

```c++
// file: for_each.cpp
#include <algorithm>
#include <iostream>
#include <vector>

void print(int x)    { std::cout << x << " "; }
void twice(int &x)   { x *= 2; }

int main() {
    std::vector<int> v{1, 2, 3, 4, 5};

    std::for_each(v.begin(), v.end(), print);                 std::cout << std::endl;
    std::for_each(v.begin(), v.end(), twice);                 // the parameter is a reference
    std::for_each(v.begin(), v.end(), print);                 std::cout << std::endl;

    std::for_each(v.begin() + 1, v.end() - 1, print);         std::cout << std::endl;   // a sub-range
    std::for_each_n(v.begin(), 3, print);                     std::cout << std::endl;   // first n elements (C++17)
}
```

```bash
g++ -std=c++17 -Wall -Wextra for_each.cpp -o for_each && ./for_each
```

`std::for_each` applies a function to every element and ignores its result. If the callback takes a **reference** parameter (`int &x`), it changes the element in place. The range can be any part of the container, and `std::for_each_n` (C++17) takes a count instead of an end. When would you prefer `for_each` over a range-based `for`, and when the other way round? Write the callback `print_pair` and use it on a `std::map<std::string, int>`.

### Task 5: Replace three raw loops

```c++
// file: loops.cpp
#include <algorithm>
#include <cassert>
#include <iostream>
#include <iterator>
#include <numeric>
#include <vector>

int clamp100(int x) { return x > 100 ? 100 : x; }

int main() {
    std::vector<int> scores{42, 120, 77, 100, 8, 150};

    // --- raw loops -------------------------------------------------
    std::vector<int> capped_a;
    for (std::size_t i = 0; i < scores.size(); ++i)
        capped_a.push_back(scores[i] > 100 ? 100 : scores[i]);

    int total_a = 0;
    for (std::size_t i = 0; i < capped_a.size(); ++i) total_a += capped_a[i];

    int best_a = capped_a[0];
    for (std::size_t i = 1; i < capped_a.size(); ++i)
        if (capped_a[i] > best_a) best_a = capped_a[i];

    // --- the same with algorithms -----------------------------------
    std::vector<int> capped_b;
    std::transform(scores.begin(), scores.end(), std::back_inserter(capped_b), clamp100);
    int total_b = std::accumulate(capped_b.begin(), capped_b.end(), 0);
    int best_b  = *std::max_element(capped_b.begin(), capped_b.end());

    assert(capped_a == capped_b && total_a == total_b && best_a == best_b);
    std::cout << "total " << total_b << ", best " << best_b << std::endl;
}
```

```bash
g++ -std=c++17 -Wall -Wextra loops.cpp -o loops && ./loops
```

The program computes the same three results twice, with loops and with algorithms, and `assert` checks that they agree. Count the lines and the variables that are needed in each version. Then extend both versions by one more step: the **number of scores that were capped**. Which algorithm do you use?

## Questions

- Which three kinds of work do `transform`, `accumulate` and `for_each` stand for?
- Why does `std::transform` into an empty vector need `std::back_inserter`?
- What is the result type of `std::accumulate`, and what happens if the start value is `0` for a vector of `double`?
- How can `std::accumulate` compute a product, a maximum or a joined string?
- Why does `std::reduce` need an associative operation while `std::accumulate` does not?
- What do you gain from replacing a raw loop by a named algorithm, and when would you keep the loop?

## Advice

When you write a loop, ask first which of the three jobs it does: map, reduce, or act. Choose the start value of `accumulate` with the result type in mind (`0.0`, `0LL`, `std::string()`). The callbacks in this exercise had to be named functions defined elsewhere; the next exercises show how to write them where you use them.
