# Contents

1. [Allow lambda capture `[=, this]`](#capture_this)
1. [Templated lambda expression](#templated_expression)
1. [Perfect forwarding](#perf_forward)
1. [Allow pack expansion in lambda init-capture](#pack_expansion)
1. [Lambdas (without captures) are default-constructiable and assignable](#construct)
1. [Lambdas in unevaluated contexts](#unevaluated)

<p align="right"><a href="../README.md#contents">:arrow_left: Back to Contents</a></p>

<a name="capture_this"></a>
## Allow lambda capture `[=, this]`

Since C++20, you need to capture `this` explicitly in case of using `[=]`, e.g.

- `[=]` -> `[=,  this]` - local variables by value, class members by reference
- `[=]` -> `[=, *this]` - everything by value
- `[&]` -> `[&,  this]` - everything by reference
- `[&]` -> `[&, *this]` - (this would be unusual)

<p align="right"><a href="#contents">:arrow_up: Back to Contents</a></p>

<a name="templated_expression"></a>
## Templated lambda expression

Added possibility to use templated lambda expressions, e.g. `[]<typename T>(T t) { /* ... */ }`

  * Retrieve type of parameters of generic lambdas to access static members/methods or nested aliases

  ```cpp
  // Before
  auto foo = [](const auto& value) {
    using T = std::decay_t<decltype(value)>;
    T valueCopy = value;
    T::staticMethod();
    using Alias = T::NestedAlias;
  };

  // Now
  auto foo = []<typename T>(const T& value) {
    T valueCopy = value;
    T::staticMethod();
    using Alias = T::NestedAlias;
  };
  ```

  * Retrieve type of the element of containers

  ```cpp
  // Before
  auto foo = [](const auto& data) {
    using T = typename std::decay_t<decltype(data)>::value_type;
    // ...
  };

  // Now
  auto foo = []<typename T>(const std::vector<T>& data) {
    // ...
  };
  ```

<p align="right"><a href="#contents">:arrow_up: Back to Contents</a></p>

<a name="perf_forward"></a>
## Perfect forwarding

```cpp
// Before
auto foo = [](auto&&... args) {
  return bar(std::forward<decltype(args)>(args)...);
};

// Now
auto foo = []<typename ...T>(T&&... args) {
  return bar(std::forward<T>(args)...);
};
```

<p align="right"><a href="#contents">:arrow_up: Back to Contents</a></p>

<a name="pack_expansion"></a>
## Allow pack expansion in lambda init-capture

```cpp
template<typename F, typename... Args>
auto DelayInvoke(F f, Args... args) {
  return [f = std::move(f), args = std::move(args)...]() { // ...args?
    return std::invoke(f, args...);
  };
}
```

<p align="right"><a href="#contents">:arrow_up: Back to Contents</a></p>

<a name="construct"></a>
## Lambdas (without captures) are default-constructiable and assignable

TODO:

<p align="right"><a href="#contents">:arrow_up: Back to Contents</a></p>

<a name="unevaluated"></a>
## Lambdas in unevaluated contexts

* Lambda as a member of a class

```cpp
class Widget {
  decltype([]{}) foo;
};
```

* Custom deleter for smart pointers

```cpp
template<typename T>
using MyPtr = std::unique_ptr<T, decltype([](T* t) { MyDeleter(t); })>

MyPtr<Widget> ptr;
```

* Predicate for containers

```cpp
using WidgetSet = std::set<Widget, decltype([](Widget& lhs, Widget& rhs) { return lhs.x < rhs.x; } )>

WidgetSet widgets;
```

<p align="right"><a href="#contents">:arrow_up: Back to Contents</a></p>
