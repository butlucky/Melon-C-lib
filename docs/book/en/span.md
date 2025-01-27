## Span



The span component in Melon is used to measure C language function consumption. This module needs to be used together with the `function` module, so it is also necessary to enable the function template through defining macro `MLN_FUNC_FLAG`, thereby enabling function consumption tracing.

Currently supported consumption are as follows:

- time consumption



### Header file

```c
#include "mln_span.h"
```



### Module

`span`



### Functions/Macros



#### mln_span_start

```c
mln_span_start();
```

Description: Start the measurement. This macro function sets the global variables used by the module and only tracks the functions called by the thread that calls this function during the measurement.

**Note**: This macro function needs to be called in the same scope as the `mln_span_stop` macro function, or the call stack depth of the `mln_span_stop` should be deeper than the call stack depth of the `mln_span_start`. For example:

> If there are three functions: `main`, `foo`, and `bar`, and their calling relationships are as follows:
>
> ```c
> void bar(void)
> {
> }
> void foo(void)
> {
>   bar();
> }
> int main(void)
> {
>   foo();
>   return 0;
> }
> ```
>
> If `mln_span_start` is called before the `bar` call in `foo`, then `mln_span_stop` should be called after `mln_span_start` in `foo` or within the `bar` function, but not in `main`.

Not following the above rules may lead to a situation where memory leaks could occur.

Return value: none



#### mln_span_stop

```c
mln_span_stop();
```

Description: Stop the measurement. This macro function destroys some global variables within the module but retains the structure containing the measured values for this measurement. The timing of calling this macro function follows the description in `mln_span_start`.

Return value: None



#### mln_span_release

```c
mln_span_release();
```

Description: Release the structure containing the measured values for this measurement. **Note**: This macro function should be called after `mln_span_stop`, otherwise, it may lead to memory access exceptions.

Return value: None



#### mln_span_move

```c
mln_span_move();
```

Description: Retrieve the structure containing the measured values for this measurement and set the global pointer pointing to this structure to `NULL`. **Note**: This macro function should be called after `mln_span_stop`, otherwise, unpredictable errors may occur.

Return value: Pointer to `mln_span_t`



#### mln_span_dump

```c
mln_span_dump();
```

Description: Output the data for the current measurement.

Return value: None



#### mln_span_free

```c
void mln_span_free(mln_span_t *s);
```

Description: Free the memory allocated for the `mln_span_t` structure.

Return value: None



### Example

This is a multi-threading example demonstrating the usage of the `mln_span` interfaces and its runtime behavior in a multi-threaded environment.

```c
//a.c

#include <pthread.h>
#include "mln_span.h"
#include "mln_func.h"

MLN_FUNC(int, abc, (int a, int b), (a, b), {
    return a + b;
})

MLN_FUNC(static int, bcd, (int a, int b), (a, b), {
    return abc(a, b) + abc(a, b);
})

MLN_FUNC(static int, cde, (int a, int b), (a, b), {
    return bcd(a, b) + bcd(a, b);
})

void *pentry(void *args)
{
    int i;
    mln_span_start();
    for (i = 0; i < 10; ++i) {
        cde(i, i + 1);
    }

    mln_span_stop();
    mln_span_dump();
    mln_span_release();
    return NULL;
}

int main(void)
{
    int i;
    pthread_t pth;


    pthread_create(&pth, NULL, pentry, NULL);

    for (i = 0; i < 10; ++i) {
        bcd(i, i + 1);
    }

    pthread_join(pth, NULL);

    return 0;
}
```

Compile the program:

```bash
cc -o a a.c -I /usr/local/melon/include/ -L /usr/local/melon/lib/ -lmelon -DMLN_FUNC_FLAG -lpthread
```

After execution, you should see the following output:

```
| pentry at a.c:20 takes 92 (us)
  | cde at a.c:13 takes 4 (us)
    | bcd at a.c:9 takes 1 (us)
      | abc at a.c:5 takes 0 (us)
      | abc at a.c:5 takes 0 (us)
    | bcd at a.c:9 takes 1 (us)
      | abc at a.c:5 takes 0 (us)
      | abc at a.c:5 takes 0 (us)
  | cde at a.c:13 takes 5 (us)
    | bcd at a.c:9 takes 0 (us)
      | abc at a.c:5 takes 0 (us)
      | abc at a.c:5 takes 0 (us)
    | bcd at a.c:9 takes 2 (us)
      | abc at a.c:5 takes 0 (us)
      | abc at a.c:5 takes 0 (us)
  | cde at a.c:13 takes 24 (us)
    | bcd at a.c:9 takes 1 (us)
      | abc at a.c:5 takes 0 (us)
      | abc at a.c:5 takes 0 (us)
    | bcd at a.c:9 takes 21 (us)
      | abc at a.c:5 takes 0 (us)
      | abc at a.c:5 takes 0 (us)
  | cde at a.c:13 takes 5 (us)
    | bcd at a.c:9 takes 1 (us)
      | abc at a.c:5 takes 0 (us)
      | abc at a.c:5 takes 0 (us)
    | bcd at a.c:9 takes 1 (us)
      | abc at a.c:5 takes 0 (us)
      | abc at a.c:5 takes 0 (us)
  | cde at a.c:13 takes 3 (us)
    | bcd at a.c:9 takes 2 (us)
      | abc at a.c:5 takes 0 (us)
      | abc at a.c:5 takes 0 (us)
    | bcd at a.c:9 takes 1 (us)
      | abc at a.c:5 takes 0 (us)
      | abc at a.c:5 takes 0 (us)
  | cde at a.c:13 takes 30 (us)
    | bcd at a.c:9 takes 24 (us)
      | abc at a.c:5 takes 0 (us)
      | abc at a.c:5 takes 1 (us)
    | bcd at a.c:9 takes 6 (us)
      | abc at a.c:5 takes 0 (us)
      | abc at a.c:5 takes 0 (us)
  | cde at a.c:13 takes 3 (us)
    | bcd at a.c:9 takes 2 (us)
      | abc at a.c:5 takes 0 (us)
      | abc at a.c:5 takes 0 (us)
    | bcd at a.c:9 takes 1 (us)
      | abc at a.c:5 takes 0 (us)
      | abc at a.c:5 takes 0 (us)
  | cde at a.c:13 takes 3 (us)
    | bcd at a.c:9 takes 2 (us)
      | abc at a.c:5 takes 0 (us)
      | abc at a.c:5 takes 1 (us)
    | bcd at a.c:9 takes 1 (us)
      | abc at a.c:5 takes 0 (us)
      | abc at a.c:5 takes 0 (us)
  | cde at a.c:13 takes 7 (us)
    | bcd at a.c:9 takes 1 (us)
      | abc at a.c:5 takes 0 (us)
      | abc at a.c:5 takes 0 (us)
    | bcd at a.c:9 takes 2 (us)
      | abc at a.c:5 takes 0 (us)
      | abc at a.c:5 takes 1 (us)
  | cde at a.c:13 takes 3 (us)
    | bcd at a.c:9 takes 2 (us)
      | abc at a.c:5 takes 1 (us)
      | abc at a.c:5 takes 0 (us)
    | bcd at a.c:9 takes 0 (us)
      | abc at a.c:5 takes 0 (us)
      | abc at a.c:5 takes 0 (us)
```

