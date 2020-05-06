# Static Libaray

## 介绍

以下展示一个创建并链接一个静态库的hello world级别的例子。在这个例子中，简单的将库和二进制文件放在同一个目录下。一般来说，这些应该在子项目下，这个将在[02-sub-projects]()中详细介绍。

该教程中的文件目录如下所示：

```
C-static-library$ tree
.
├── CMakeLists.txt
├── include
│   └── static
│       └── Hello.h
└── src
    ├── Hello.cpp
    └── main.cpp
```

* [CMakeLists.txt](https://github.com/LiYuRio/best-practice-cmake/blob/master/cmake-examples-zh/01-basic/C-static-library/CMakeLists.txt)：包含希望运行的CMake命令
* [include/Hello.h](https://github.com/LiYuRio/best-practice-cmake/blob/master/cmake-examples-zh/01-basic/C-static-library/include/static/Hello.h)：想要包含的头文件
* [src/Hello.cpp](https://github.com/LiYuRio/best-practice-cmake/blob/master/cmake-examples-zh/01-basic/C-static-library/src/Hello.cpp)：待编译的源文件
* [src/main.cpp](https://github.com/LiYuRio/best-practice-cmake/blob/master/cmake-examples-zh/01-basic/C-static-library/src/main.cpp)：待编译的main文件

## 概念

### 创建一个静态库

`add_library()`函数可以将一些源文件编译并创建成为一个库，调用方式如下：

```cmake
add_library(hello_library STATIC
    src/Hello.cpp
)
```

执行以上所示的命令后，会创建一个名为`libhello_library.a`的静态库。

和上一个例子中所说的一样，直接将源文件名放到`add_library`中，是目前的CMake比较推荐的做法。

### 进一步介绍——将目录包含进来的

在这个例子中，将编译库需要用到的头文件目录通过`target_include_directories()`函数添加进来，并把对应作用域设置为PUBLIC。

```cmake
target_include_directories(hello_library
    PUBLIC
        ${PROJECT_SOURCE_DIR}/include
)
```

包含进来的头文件目录会在以下情况中用到

* 编译库的时候
* 编译其他链接了这个库的目标的时候

三种不同的作用域：

* PRIVATE：头文件目录只在编译本目标时使用
* INTERFACE：头文件目录只在链接了该目标的其他目标编译时使用
* PUBLIC：头文件目录不仅在编译本目标时使用，在编译链接了这个目标的其他目标时也会使用

提示：对于公有的头文件，在头文件目录中创建子目录，用于放置不同库的头文件，是一个好的习惯。将所有头文件的根目录用`target_include_directories()`函数包含进来，这样具体实现的源文件就可以在此路径基础上，寻找需要的头文件。

如下所示，通过这种方式可以减少因为不同库采用相同的头文件名而造成冲突的可能性。

```c++
#include "static/Hello.h"
```

### 将库链接进来

当待创建的可执行文件需要链接一个外部库的时候，需要通过`target_link_libraries()`函数显式的告诉编译器。

```cmake
add_executable(hello_binary
    src/main.cpp
)

target_link_libraries( hello_binary
    PRIVATE
        hello_library
)
```

通过这个函数，CMake会在链接的时候将名为hello_library的这个库链接到可执行文件hello_binary上，同时也会将前面使用`target_include_directories()`函数包括进来的，并且作用域为PUBLIC或者INTERFACE的头文件目录一起包括进来。

下面是编译器编译输出的一个示例：

```
/usr/bin/c++ CMakeFiles/hello_binary.dir/src/main.cpp.o -o hello_binary -rdynamic libhello_library.a
```



## 构建示例

本例子的标准构建输出如下所示

```
$ mkdir build

$ cd build

$ cmake ..
-- The C compiler identification is GNU 4.8.4
-- The CXX compiler identification is GNU 4.8.4
-- Check for working C compiler: /usr/bin/cc
-- Check for working C compiler: /usr/bin/cc -- works
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - done
-- Check for working CXX compiler: /usr/bin/c++
-- Check for working CXX compiler: /usr/bin/c++ -- works
-- Detecting CXX compiler ABI info
-- Detecting CXX compiler ABI info - done
-- Configuring done
-- Generating done
-- Build files have been written to: /home/matrim/workspace/cmake-examples/01-basic/C-static-library/build

$ make
Scanning dependencies of target hello_library
[ 50%] Building CXX object CMakeFiles/hello_library.dir/src/Hello.cpp.o
Linking CXX static library libhello_library.a
[ 50%] Built target hello_library
Scanning dependencies of target hello_binary
[100%] Building CXX object CMakeFiles/hello_binary.dir/src/main.cpp.o
Linking CXX executable hello_binary
[100%] Built target hello_binary

$ ls
CMakeCache.txt  CMakeFiles  cmake_install.cmake  hello_binary  libhello_library.a  Makefile

$ ./hello_binary
Hello Static Library!
```
