+++
title = 'Compiler Attributes: nonnull'
+++

GCC and Clang provide the `__attribute__((nonnull))` or `gnu::nonnull` attribute, a compile-time mechanism to enhance code safety by allowing you to declare that specific function pointer arguments must never be `NULL`.
This has the following benefits:

1. Catch potential `NULL` pointer errors during compilation, preventing runtime crashes.
2. By knowing certain pointers are never `NULL`, the compiler can perform more aggressive optimizations, potentially leading to faster code.
3. The `nonnull` attribute acts as a clear contract, communicating to developers which arguments are expected to be valid pointers.

You can apply the `nonnull` attribute in two ways:

- `__attribute__((nonnull))`: This form indicates that *all* pointer arguments of the function must be non-`NULL`.
- `__attribute__((nonnull(arg_index_1, arg_index_2, ...)))`: This allows you to specify which specific pointer arguments (using 1-based indexing) are expected to be non-`NULL`.

C++ users might encounter the `[[gnu::nonnull(index)]]` syntax, which is a GNU extension even within C++11 attribute brackets.
Be aware that some compilers might warn or ignore this form.

```c
void process_data(int *data, size_t size) __attribute__((nonnull(1)));

void test_process() {
    process_data(NULL, 10); // Compiler may warn: argument 1 is NULL
    int values[] = {1, 2, 3};
    process_data(values, sizeof(values) / sizeof(int)); // OK
}

[[gnu::nonnull(2, 4)]]
int calculate_result(const char *operation, const int *values,
                     size_t count, int (*process)(int));

void test_calculation() {
    int numbers[] = {5, 10};
    calculate_result("+", numbers, 2, NULL);
    // Compiler may warn: argument 4 is NULL
}
```

## Important Notes

First, for this warnings to be emitted, `-Wall` must be passed to the compiler.

```console
test.c: In function 'test_process':
test.c:4:5: warning: argument 1 null where non-null expected [-Wnonnull]
    4 |     process_data(NULL, 10); // Compiler may warn: argument 1 is NULL
      |     ^~~~~~~~~~~~
test.c:1:6: note: in a call to function 'process_data' declared 'nonnull'
    1 | void process_data(int *data, size_t size) __attribute__((nonnull(1)));
      |      ^~~~~~~~~~~~
test.c: In function 'test_calculation':
test.c:15:5: warning: argument 4 null where non-null expected [-Wnonnull]
   15 |     calculate_result("+", numbers, 2, NULL);
      |     ^~~~~~~~~~~~~~~~
test.c:10:5: note: in a call to function 'calculate_result' declared 'nonnull'
   10 | int calculate_result(const char *operation, const int *values,
      |     ^~~~~~~~~~~~~~~~
```

Second, remember to apply the `nonnull` attribute during the function *declaration*, not within the function definition itself.

```c
// Correct
void analyze_string(const char *str) __attribute__((nonnull(1)));

void analyze_string(const char *str) {
    // ... implementation ...
}

// ~~~

// Incorrect:
void analyze_string(const char *str) __attribute__((nonnull(1))) {
    // ... implementation ...
}
```

## Runtime Checks Still Matter

While `nonnull` is a fantastic tool for compile-time safety, it's crucial to understand its limitations.
The compiler can only warn about `NULL` values that it can determine *statically* (e.g., literal `NULL`).
In dynamic scenarios where a pointer might become `NULL` during runtime, the `nonnull` attribute won't provide protection.

Therefore, especially for public APIs or library functions, it's still best practice to include runtime checks (like `assert(ptr != NULL)` for debugging or explicit `if (!ptr) return;` for production) to handle potential `NULL` values gracefully.

## In Conclusion

The `__attribute__((nonnull))` attribute in GCC and Clang is a valuable asset in writing safer and more efficient C and C++ code.
By clearly specifying which pointer arguments should never be `NULL`, you empower the compiler to help you catch potential errors early, leading to more robust and maintainable software.


