# Hello Headers

## 介绍

以下展示一个将源文件和头文件分别放在不同目录下的hello world级别的例子。

该教程中的文件目录如下所示：

```
B-hello-headers$ tree
.
├── CMakeLists.txt
├── include
│   └── Hello.h
└── src
    ├── Hello.cpp
    └── main.cpp
```

* [CMakeLists.txt](https://github.com/LiYuRio/best-practice-cmake/blob/master/cmake-examples-zh/01-basic/B-hello-headers/CMakeLists.txt)：包含希望运行的CMake命令
* [include/Hello.h](https://github.com/LiYuRio/best-practice-cmake/blob/master/cmake-examples-zh/01-basic/B-hello-headers/include/Hello.h)：想要包含的头文件
* [src/Hello.cpp](https://github.com/LiYuRio/best-practice-cmake/blob/master/cmake-examples-zh/01-basic/B-hello-headers/src/Hello.cpp)：待编译的源文件
* [src/main.cpp](https://github.com/LiYuRio/best-practice-cmake/blob/master/cmake-examples-zh/01-basic/B-hello-headers/src/main.cpp)：待编译的main文件

## 概念

### 目录路径

CMake语法中自带一些[变量](https://gitlab.kitware.com/cmake/community/-/wikis/doc/cmake/Useful-Variables)，这些变量中存放着项目中的一些有用的目录的路径，如下面所示，是其中的一些常用的变量：

| 变量                     | 介绍                                       |
| ------------------------ | ------------------------------------------ |
| CMAKE_SOURCE_DIR         | 项目的根源目录                             |
| CMAKE_CURRENT_SOURCE_DIR | 当前项目的源目录（对一些包含子项目的项目） |
| PROJECT_SOURCE_DIR       | 当前cmake项目的源目录                      |
| CMAKE_BINARY_DIR         | 根二进制目录，就是执行cmake命令所在的目录  |
| CMAKE_CURRENT_BINARY_DIR | 当前所在的二进制目录                       |
| PROJECT_BINARY_DIR       | 当前项目的二进制目录                       |

（注：如果不确定这个目录指向哪，可以用`MESSAGE(STATUS '${...}')`将变量打印出来）

### 源文件变量

把所有源文件包含放在一个变量中，可以很方便地将它们直接应用于不同的命令中，比如`add_executable()`函数。

```cmake
# 创建一个名为sources的变量，其中包含了所有需要编译的cpp文件
set(SOURCES
    src/Hello.cpp
    src/main.cpp
)

add_executable(${PROJECT_NAME} ${SOURCES})
```

除了上面的这种列举源文件的方式，还可以使用GLOB命令，将与通配符规则匹配的所有文件名，加入变量中。

```cmake
file(GLOB SOURCES "src/*.cpp")
```

不过，对现代的CMake来说，不是很推荐使用变量代替所有的源文件，比如当新增了一个新的源文件时，glob命令可能不总是会返回正确的结果。将源文件的声明直接放在add_xxx函数中是比较典型的做法。

### 将目录包含进来

为了让编译器知道被放在其他目录下（`include/`）的头文件的存在，可以使用`target_directories()`函数。在编译目标的时候，会通过加`-I`编译参数把这些目录引用进来，比如`-I/directory/path`

```cmake
target_include_directories(target
    PRIVATE
        ${PROJECT_SOURCE_DIR}/include
)
```

PRIVATE标识符指明，包含进来文件夹的作用范围，这对下个例子中将要介绍的库来说，是比较重要的。有关本函数更多细节可以看[这里](https://cmake.org/cmake/help/v3.0/command/target_include_directories.html)。

## 构建示例

### 标准输出

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
-- Build files have been written to: /home/matrim/workspace/cmake-examples/01-basic/hello_headers/build

$ make
Scanning dependencies of target hello_headers
[ 50%] Building CXX object CMakeFiles/hello_headers.dir/src/Hello.cpp.o
[100%] Building CXX object CMakeFiles/hello_headers.dir/src/main.cpp.o
Linking CXX executable hello_headers
[100%] Built target hello_headers

$ ./hello_headers
Hello Headers!
```

### Verbose（带一些信息的）输出

上前面的例子中，执行make命令后，控制行只会输出构建过程中的状态信息，如果为了调试方便，想看完整的输出，可以在make命令后面增加`VERBOSE=1`的标志。

加了VERBOSE后的输出如下所示，通过观察发现，这次的输出信息中包括了将include文件夹引用进来的编译命令（在`50%`那一行）

```
$ make clean

$ make VERBOSE=1
/usr/bin/cmake -H/home/matrim/workspace/cmake-examples/01-basic/hello_headers -B/home/matrim/workspace/cmake-examples/01-basic/hello_headers/build --check-build-system CMakeFiles/Makefile.cmake 0
/usr/bin/cmake -E cmake_progress_start /home/matrim/workspace/cmake-examples/01-basic/hello_headers/build/CMakeFiles /home/matrim/workspace/cmake-examples/01-basic/hello_headers/build/CMakeFiles/progress.marks
make -f CMakeFiles/Makefile2 all
make[1]: Entering directory `/home/matrim/workspace/cmake-examples/01-basic/hello_headers/build'
make -f CMakeFiles/hello_headers.dir/build.make CMakeFiles/hello_headers.dir/depend
make[2]: Entering directory `/home/matrim/workspace/cmake-examples/01-basic/hello_headers/build'
cd /home/matrim/workspace/cmake-examples/01-basic/hello_headers/build && /usr/bin/cmake -E cmake_depends "Unix Makefiles" /home/matrim/workspace/cmake-examples/01-basic/hello_headers /home/matrim/workspace/cmake-examples/01-basic/hello_headers /home/matrim/workspace/cmake-examples/01-basic/hello_headers/build /home/matrim/workspace/cmake-examples/01-basic/hello_headers/build /home/matrim/workspace/cmake-examples/01-basic/hello_headers/build/CMakeFiles/hello_headers.dir/DependInfo.cmake --color=
make[2]: Leaving directory `/home/matrim/workspace/cmake-examples/01-basic/hello_headers/build'
make -f CMakeFiles/hello_headers.dir/build.make CMakeFiles/hello_headers.dir/build
make[2]: Entering directory `/home/matrim/workspace/cmake-examples/01-basic/hello_headers/build'
/usr/bin/cmake -E cmake_progress_report /home/matrim/workspace/cmake-examples/01-basic/hello_headers/build/CMakeFiles 1
[ 50%] Building CXX object CMakeFiles/hello_headers.dir/src/Hello.cpp.o
/usr/bin/c++    -I/home/matrim/workspace/cmake-examples/01-basic/hello_headers/include    -o CMakeFiles/hello_headers.dir/src/Hello.cpp.o -c /home/matrim/workspace/cmake-examples/01-basic/hello_headers/src/Hello.cpp
/usr/bin/cmake -E cmake_progress_report /home/matrim/workspace/cmake-examples/01-basic/hello_headers/build/CMakeFiles 2
[100%] Building CXX object CMakeFiles/hello_headers.dir/src/main.cpp.o
/usr/bin/c++    -I/home/matrim/workspace/cmake-examples/01-basic/hello_headers/include    -o CMakeFiles/hello_headers.dir/src/main.cpp.o -c /home/matrim/workspace/cmake-examples/01-basic/hello_headers/src/main.cpp
Linking CXX executable hello_headers
/usr/bin/cmake -E cmake_link_script CMakeFiles/hello_headers.dir/link.txt --verbose=1
/usr/bin/c++       CMakeFiles/hello_headers.dir/src/Hello.cpp.o CMakeFiles/hello_headers.dir/src/main.cpp.o  -o hello_headers -rdynamic
make[2]: Leaving directory `/home/matrim/workspace/cmake-examples/01-basic/hello_headers/build'
/usr/bin/cmake -E cmake_progress_report /home/matrim/workspace/cmake-examples/01-basic/hello_headers/build/CMakeFiles  1 2
[100%] Built target hello_headers
make[1]: Leaving directory `/home/matrim/workspace/cmake-examples/01-basic/hello_headers/build'
/usr/bin/cmake -E cmake_progress_start /home/matrim/workspace/cmake-examples/01-basic/hello_headers/build/CMakeFiles 0
```

