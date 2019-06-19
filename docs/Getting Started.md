## Compiling and Installing Marisa

The Marisa source is available at [Github](https://github.com/ReimuNotMoe/Marisa).

### Install dependencies

The following dependencies are required:

- C++17 compatible compiler and runtime
- cmake
- zlib
- boost_system
- OpenSSL

#### Ubuntu 
    apt install build-essential cmake libboost-all-dev zlib1g-dev libssl-dev

### Run CMake

In the source directory, create a new directory named `build`, and change into it.

#### Production build

    cmake -DCMAKE_BUILD_TYPE=Debug -DCMAKE_CXX_FLAGS='-O2 -march=native' ..

#### Debug build

Defining `DEBUG` will turn on debugging outputs.

    cmake -DCMAKE_BUILD_TYPE=Debug -DCMAKE_CXX_FLAGS='-O0 -DDEBUG' ..

### Run make

    make -j <cpu cores>

### Install

By default, the headers and libraries will be installed to `/usr/local/` on Unix-like systems.

You could link `/usr/local/lib/marisa.so` to `/usr/lib/marisa.so` since some distros won't search libraries in `/usr/local/lib`.

Just run:

    make install


## Compiling Programs that Use Marisa

### Headers

If you have installed Marisa into your system, you can just:

    #include <Marisa/Marisa.h>
    
If you are working on the source directory, you can just include the relative path. Like:

    #include "../Marisa.h"
    
### Linking

#### Command line

    -l marisa
    
#### CMake

    target_link_libraries(target marisa)
