+++
title = 'Compiler Attributes: nonnull'
+++

You can use [`nonnull` attribute](https://gcc.gnu.org/onlinedocs/gcc-4.4.7/gcc/Function-Attributes.html) to write safer, clearer, and faster code.

GCC and Clang support the `__attribute__((nonnull))` (or `[[gnu::nonnull]]` in C++) to mark function pointer arguments as never `NULL`. This helps:

1. Catch `NULL` arguments at compile time.
2. Enable better compiler optimizations.
3. Document intent clearly.

* `__attribute__((nonnull))`: All pointer args must be non-`NULL`.
* `__attribute__((nonnull(1, 3)))`: Only specific arguments (1-based indexing).

For example:

```c
void f(int *p) __attribute__((nonnull(1)));
f(NULL); // May trigger warning with -Wall
```

In C++:

```cpp
[[gnu::nonnull(2)]] void g(int a, int *b);
```

Note that

* Add `-Wall` to enable warnings.
* Apply `nonnull` in declarations, not definitions.
* Only catches statically known `NULL`s—runtime checks are still needed.
