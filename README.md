# Getting the maximum of your C compiler, for security

- [GCC TL;DR](#gcc-tldr)
- [Clang TL;DR](#clang-tldr)
- [Microsoft Visual Studio 2019 TL;DR](#microsoft-visual-studio-2019-tldr)
- [References](#references)

### Introduction

This guide is intended to help you determine which flags you should use to
compile you C Code using GCC, Clang or MSVC, in order to:

* detect the maximum number of bugs or potential security problems.
* enable security mitigations in the produced binaries.
* make fuzzing more efficient by enabling runtime sanitizers


## GCC TL;DR

[Detailed page](./gcc_compilation.md)

Always use the following [warnings](./gcc_compilation.md#warnings) and [flags](./gcc_compilation.md#compilation-flags) on the command line, disable the warnings that have too much false positives, after considering the implications:
```
-O2 -Wall -Wextra -Wpedantic -Wformat=2 -Wformat-overflow=2 -Wformat-truncation=2 -Wformat-security -Wnull-dereference -Wstack-protector -Wtrampolines -Walloca -Wvla -Warray-bounds=2 -Wimplicit-fallthrough=3 -Wtraditional-conversion -Wshift-overflow=2 -Wcast-qual -Wstringop-overflow=4 -Wconversion -Warith-conversion -Wlogical-op -Wduplicated-cond -Wduplicated-branches -Wformat-signedness -Wshadow -Wstrict-overflow=4 -Wundef -Wstrict-prototypes -Wswitch-default -Wswitch-enum -Wstack-usage=1000000 -Wcast-align=strict -fstack-protector-strong -fstack-clash-protection -fPIE
```

Run debug/test builds with [sanitizers](./gcc_compilation.md#runtime-sanitizers) (in addition to the flags above):

AddressSanitizer + UndefinedBehaviorSanitizer:
```
-fsanitize=address -fsanitize=pointer-compare -fsanitize=pointer-subtract -fsanitize=leak -fno-omit-frame-pointer -fsanitize=undefined -fsanitize=bounds-strict -fsanitize=float-divide-by-zero -fsanitize=float-cast-overflow
export ASAN_OPTIONS=strict_string_checks=1:detect_stack_use_after_return=1:check_initialization_order=1:strict_init_order=1:detect_invalid_pointer_pairs=2
```

If your program is multi-threaded, run with `-fsanitize=thread` (incompatible with ASan).

Finally, use [`-fanalyzer`](./gcc_compilation.md#code-analysis) to spot potential issues.

## Clang TL;DR

[Detailed page](./clang_compilation.md)

First compile with:

```
-O2 -Weverything
```
and disable the warnings that have too much false positives, after considering the implications:

Run debug/test builds with sanitizers (in addition to the flags above):

AddressSanitizer + UndefinedBehaviorSanitizer:
```
-fsanitize=address -fsanitize=leak -fno-omit-frame-pointer -fsanitize=undefined -fsanitize=bounds-strict -fsanitize=float-divide-by-zero -fsanitize=float-cast-overflow -fsanitize=integer -fsanitize-no-recover
export ASAN_OPTIONS=strict_string_checks=1:detect_stack_use_after_return=1:check_initialization_order=1:strict_init_order=1:detect_invalid_pointer_pairs=2
```

If your program is multi-threaded, run with `-fsanitize=thread` (incompatible with ASan).

Finally, use [`scan-build`](./clang_compilation.md#code-analysis) to spot potential issues.

In addition, you can build production code with `-fsanitize=integer -fsanitize-minimal-runtime -fsanitize-no-recover` to catch integer overflows.


## Microsoft Visual Studio 2019 TL;DR

[Detailed page](./msvc_compilation.md)

* Compile with `/Wall /sdl /guard:cf /guard:ehcont`
* Use ASan with `/fsanitize=address`
* Analyze your code with `/fanalyze`

## References

* For [GCC](./gcc_compilation.md#references)
* For [Clang](./clang_compilation.md#references)
* For [MSVC](./msvc_compilation.md#references)
* <https://github.com/pkolbus/compiler-warnings>: GCC/Clang/XCode parsers for warnings definitions.
* <https://github.com/google/sanitizers/wiki/AddressSanitizerFlags>: ASan runtime options

