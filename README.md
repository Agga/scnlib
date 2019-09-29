# scnlib

[![Travis CI Build Status](https://travis-ci.com/eliaskosunen/scnlib.svg?branch=master)](https://travis-ci.com/eliaskosunen/scnlib)
[![Appveyor CI Build Status](https://ci.appveyor.com/api/projects/status/ex0q59kt5h8yciqa/branch/master?svg=true)](https://ci.appveyor.com/project/eliaskosunen/scnlib/branch/master)
[![Codecov Coverage](https://codecov.io/gh/eliaskosunen/scnlib/branch/master/graph/badge.svg)](https://codecov.io/gh/eliaskosunen/scnlib)
[![Codacy Code Quality](https://api.codacy.com/project/badge/Grade/daf649bfab44407fa7afda6cb97add2a)](https://www.codacy.com/app/eliaskosunen/scnlib?utm_source=github.com&amp;utm_medium=referral&amp;utm_content=eliaskosunen/scnlib&amp;utm_campaign=Badge_Grade)
[![License](https://img.shields.io/github/license/eliaskosunen/scnlib.svg)](https://github.com/eliaskosunen/scnlib/blob/master/LICENSE)
[![C++ Standard](https://img.shields.io/badge/C%2B%2B-11%2F14%2F17%2F20-blue.svg)](https://img.shields.io/badge/C%2B%2B-11%2F14%2F17%2F20-blue.svg)

```cpp
#include <scn/scn.h>
#include <cstdio>

int main() {
    int i;
    scn::prompt("What's your favorite number?", "{}", i);
    printf("Oh, cool, %d!", i);
}

// Example result:
// What's your favorite number? 42
// Oh, cool, 42!
```

## What is this?

`scnlib` is a modern C++ library for replacing `scanf` and `std::istream`.
This library attempts to move us ever so closer to replacing `iostream`s and C stdio altogether.
It's (going to, eventually, be) faster than `iostream` (see Benchmarks) and type-safe, unlike `scanf`.
Think [{fmt}](https://github.com/fmtlib/fmt) but in the other direction.

The library is still in early development (0.2), so don't expect perfection.
See [Stability](#stability) for more details.

This library is an implementation of the ISO C++ standards proposal
[P1729 "Text Parsing"](https://wg21.link/p1729).

## Documentation and tutorial

The documentation can be found online, from https://scnlib.dev.
The documentation includes a tutorial.

To build the docs yourself, build the `doc` target generated by CMake.
Building the documentation requires Doxygen.

Rather lacking examples can be found from the `examples/` folder.

Should you get stuck somewhere, you can find me on the [CppLang Slack](https://cpplang.now.sh),
or by DMing me on [Twitter](https://twitter.com/eliaskosunen).

## Installing

### Submodule

The easiest way of integrating `scnlib` to your project would likely be to use a `git submodule`.

```sh
# Replace `external/scnlib` with the directory where you'd like the submodule be located
$ git submodule add external/scnlib https://github.com/eliaskosunen/scnlib
```

Then just do the following in your CMakeLists.txt

```cmake
# Include the submodule directory
add_subdirectory(external/scnlib EXCLUDE_FROM_ALL)

# Target which you'd like to use scnlib
add_executable(my_program ...)
# Use scn::scn-header-only if you'd prefer
target_link_libraries(my_program scn::scn)
```

### External project

You can also build `scnlib` separately.

```sh
# Whereever you cloned scnlib to
$ mkdir build
$ cd build
$ cmake ..
$ make -j
$ make install
```

Then, in your project:

```cmake
# Find scnlib package
find_package(scn CONFIG REQUIRED)

# Target which you'd like to use scnlib
add_executable(my_program ...)
target_link_libraries(my_program scn::scn)
```

## Compiler support

Every commit is tested with
 * gcc 5.5 and newer
 * clang 3.6 and newer
 * Visual Studio 2017 and 2019
with very extreme warning flags (see CMakeLists.txt) and with multiple build configurations for each compiler.

Older compilers may work, but it is not guaranteed.
GCC 4.x support will not be provided, as its C++11 support is too buggy.

## Stability

The master-branch will be API-stable until the next minor release, 0.2.
It will receive API-compatible bugfixes, should any be necessary, tagged appropriately (0.1.1 etc).

The dev-branch will not be stable. It may be rebased, and will contain changes that may break your code.
Please don't depend on it, unless you really want to live on the edge.

No ABI stability is guaranteed at this point, and none will be pre-1.0.

This is still a pre-1.0 library, so there's bound to be bugs. If you find one, _please_ report it.

## Benchmarks

### Run-time performance

These benchmarks were run on a Ubuntu 19.04 machine running kernel version 5.0.0-15, with an Intel Core i7-8750H processor, and compiled with gcc version 8.3.0, with `-O3 -march=native`.
The source code for the benchmarks can be seen in the `benchmark` directory.

Times are in nanoseconds of CPU time. Lower is better.

#### Reading random integers

| Integer type | `scn::scan` | `scn::get_value` | `std::stringstream` |
| :----------- | ----------- | ---------------: | ------------------: |
| `int`        | 71          | 50               | 70                  |
| `long long`  | 106         | 81               | 95                  |
| `unsigned`   | 81          | 60               | 53                  |

#### Reading random floating-point numbers

| Floating-point type | `scn::scan` | `scn::get_value` | `std::stringstream` |
| :------------------ | ----------: | ---------------: | ------------------: |
| `float`             | 136         | 130              | 203                 |
| `double`            | 141         | 135              | 208                 |
| `long double`       | 155         | 143              | 219                 |

#### Reading random whitespace-separated `std::basic_string`s

| Character type | `scn::scan` | `scn::get_value` | `std::stringstream` |
| :------------- | ----------: | ---------------: | ------------------: |
| `char`         | 51          | 48               | 46                  |
| `wchar_t`      | 71          | 80               | 110                 |

#### Reading random characters

| Character type | `scn::scan` | `scn::get_value` | `scn::getchar` | `std::stringstream` |
| :------------- | ----------: | ---------------: | -------------: | ------------------: |
| `char`         | 20          | 12               | 5              | 8                   |
| `wchar_t`      | 19          | 12               | 5              | 15                  |

You can run the benchmarks yourself by enabling `SCN_BUILD_BENCHMARKS` and building the target `bench`.
`SCN_BUILD_BENCHMARKS` is enabled by default if `scn` is the root CMake project, and disabled otherwise.

### Code size

Code size benchmarks test code bloat for nontrivial projects.
It generates 25 translation units and reads values from stdin\* five times to simulate a medium sized project.
The resulting executable size is shown in the following tables.

The code was compiled on Ubuntu 19.04 with g++ 8.3.0.
`scnlib` is linked dynamically to level out the playing field compared to already dynamically linked `libc` and `libstdc++`.
See the directory `benchmark/bloat` for more information, e.g. templates for each TU.

To run these tests yourself:

```sh
$ cd build
$ cmake -DCMAKE_BUILD_TYPE=Debug -DBUILD_SHARED_LIBS=ON -DSCN_INSTALL=OFF -DSCN_BLOAT=ON ..
$ make -j
$ ./benchmark/bloat/run-bloat-tests.py ./benchmark/bloat
```

`(erased)` marks the usage of `scn::make_erased_stream()` instead of `scn::make_stream()`.

#### Minimized build (-Os -DNDEBUG)

| Method                              | Executable size (KiB) | Stripped size (KiB) |
| :---------------------------------- | --------------------: | ------------------: |
| empty                               | 18                    | 14                  |
| `scanf`                             | 23                    | 18                  |
| `std::istream` / `std::cin`         | 25                    | 18                  |
| `scn::input`                        | 35                    | 30                  |
| `scn::input` (header only)          | 138                   | 98                  |

#### Release build (-O3 -DNDEBUG)

| Method                              | Executable size (KiB) | Stripped size (KiB) |
| :---------------------------------- | --------------------: | ------------------: |
| empty                               | 18                    | 14                  |
| `scanf`                             | 24                    | 18                  |
| `std::istream` / `std::cin`         | 30                    | 22                  |
| `scn::input`                        | 41                    | 34                  |
| `scn::input` (header only)          | 177                   | 146                 |

#### Debug build (-g)

| Method                              | Executable size (KiB) | Stripped size (KiB) |
| :---------------------------------- | --------------------: | ------------------: |
| empty                               | 29                    | 14                  |
| `scanf`                             | 600                   | 18                  |
| `std::istream` / `std::cin`         | 662                   | 22                  |
| `scn::input`                        | 1709                  | 51                  |
| `scn::input` (header only)          | 6858                  | 281                 |

## Acknowledgements

The contents of this library are heavily influenced by fmtlib and its derivative works.

<https://github.com/fmtlib/fmt>  
<https://fmt.dev>  

fmtlib is licensed under the BSD 2-clause license.  
Copyright (c) 2012-2019 Victor Zverovich

The ranges implementation found from this library is based on NanoRange:

<https://github.com/tcbrindle/NanoRange>

NanoRange is licensed under the Boost Software License, version 1.0.  
Copyright (c) 2018 Tristan Brindle

## License

scnlib is licensed under the Apache License, version 2.0.  
Copyright (c) Elias Kosunen 2017-2019  
See LICENSE for further details
