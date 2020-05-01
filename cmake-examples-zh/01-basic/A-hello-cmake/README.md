# Hello CMake

## 介绍

以下展示的是一个非常基本的hello world的例子。

该教程中的文件目录如下所示：

```
A-hello-cmake$ tree
.
├── CMakeLists.txt
├── main.cpp
```

* [CMakeLists.txt](https://github.com/LiYuRio/best-practice-cmake/blob/master/cmake-examples-zh/01-basic/A-hello-cmake/CMakeLists.txt)：包含希望运行的CMake命令
* [main.cpp](https://github.com/LiYuRio/best-practice-cmake/blob/master/cmake-examples-zh/01-basic/A-hello-cmake/main.cpp)：一个简单的“Hello World”的cpp程序

## 概念

### CMakeLists.txt

CMakeLists.txt存放了项目所有相关的CMake命令，当在一个文件夹中执行cmake命令时，会在所指向的文件夹中，查找CMakeLists.txt文件是否存在，如果不存在直接退出并返回一个错误信息。

### 对CMake最低版本的限制

当用CMake创建一个项目时，可以显式指明构建本项目所使用CMake的最小版本号，作为约束条件。

```cmake
cmake_minimum_required(VERSION 3.5)
```

### 设置项目

构建一个相对比较复杂的项目，可能会包含多个子项目的构建，给每个子项目的CMake构建过程设置一个名字，有利于变量的引用。

```cmake
project (hello_cmake)
```

### 创建一个可执行文件

在示例的源文件`main.cpp`中，调用`add_executable()`函数，编译某些/某个源文件并产生其对应的可执行文件的操作。`add_executable()`的第一个参数为，被创建的可执行文件的名字，第二个参数为将要被编译的源文件列表。

```cmake
add_executable(hello_cmake main.cpp)
```

不过很多人通常使用一个更简洁的写法，即将项目名和可执行文件名保持一致，CMakeLists.txt如下所示：

```cmake
cmake_minimum_required(VERSION 2.6)
project (hello_cmake)
add_executable($(PROJECT_NAME) main.cpp)
```

在上面的例子中，首先调用`project()`函数会创建一个值为hello_cmake的`${PROJECT_NAME}`变量，然后用该变量作为`add_executable()`函数的第一个参数，最终产生一个名为hello_cmake的可执行文件。

### 二进制目录

运行cmake命令时所在的目录，会被存放在`${CMAKE_BINARY_DIR}`变量中。所有因为执行cmake命令而产生的二进制文件都会直接放在该目录下。cmake支持in-place和out-of-source两种方式来构建程序。

#### in-Place构建

使用in-place构建方式会将构建产生的所有临时文件放在和源代码同一级的目录下，这就意味着，所有的编译输出的中间文件和正常的代码文件混在一起。使用这种构建方式只需要在项目根目录下直接执行cmake命令即可，如下所示：

```
A-hello-cmake$ cmake .
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
-- Build files have been written to: /home/matrim/workspace/cmake-examples/01-basic/A-hello-cmake

A-hello-cmake$ tree
.
├── CMakeCache.txt
├── CMakeFiles
│   ├── 2.8.12.2
│   │   ├── CMakeCCompiler.cmake
│   │   ├── CMakeCXXCompiler.cmake
│   │   ├── CMakeDetermineCompilerABI_C.bin
│   │   ├── CMakeDetermineCompilerABI_CXX.bin
│   │   ├── CMakeSystem.cmake
│   │   ├── CompilerIdC
│   │   │   ├── a.out
│   │   │   └── CMakeCCompilerId.c
│   │   └── CompilerIdCXX
│   │       ├── a.out
│   │       └── CMakeCXXCompilerId.cpp
│   ├── cmake.check_cache
│   ├── CMakeDirectoryInformation.cmake
│   ├── CMakeOutput.log
│   ├── CMakeTmp
│   ├── hello_cmake.dir
│   │   ├── build.make
│   │   ├── cmake_clean.cmake
│   │   ├── DependInfo.cmake
│   │   ├── depend.make
│   │   ├── flags.make
│   │   ├── link.txt
│   │   └── progress.make
│   ├── Makefile2
│   ├── Makefile.cmake
│   ├── progress.marks
│   └── TargetDirectories.txt
├── cmake_install.cmake
├── CMakeLists.txt
├── main.cpp
├── Makefile
```

####Out-of-Source构建

Out-of-Source构建方式需要创建一个新的名为build的目录，构建过程中产生的所有中间文件都会被放在该目录下。使用这种构建方式，需要在build文件夹下执行cmake命令，并且指明执行目录为（包含CMakeLists.txt文件）的项目根目录。这样如果想要重新构建项目，只需要删掉build目录，然后重新执行cmake即可。如下所示：

```
A-hello-cmake$ mkdir build

A-hello-cmake$ cd build/

A-hello-cmake/build$ make ..
make: Nothing to be done for `..'.
matrim@freyr:~/workspace/cmake-examples/01-basic/A-hello-cmake/build$ cmake ..
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
-- Build files have been written to: /home/matrim/workspace/cmake-examples/01-basic/A-hello-cmake/build

A-hello-cmake/build$ cd ..

A-hello-cmake$ tree
.
├── build
│   ├── CMakeCache.txt
│   ├── CMakeFiles
│   │   ├── 2.8.12.2
│   │   │   ├── CMakeCCompiler.cmake
│   │   │   ├── CMakeCXXCompiler.cmake
│   │   │   ├── CMakeDetermineCompilerABI_C.bin
│   │   │   ├── CMakeDetermineCompilerABI_CXX.bin
│   │   │   ├── CMakeSystem.cmake
│   │   │   ├── CompilerIdC
│   │   │   │   ├── a.out
│   │   │   │   └── CMakeCCompilerId.c
│   │   │   └── CompilerIdCXX
│   │   │       ├── a.out
│   │   │       └── CMakeCXXCompilerId.cpp
│   │   ├── cmake.check_cache
│   │   ├── CMakeDirectoryInformation.cmake
│   │   ├── CMakeOutput.log
│   │   ├── CMakeTmp
│   │   ├── hello_cmake.dir
│   │   │   ├── build.make
│   │   │   ├── cmake_clean.cmake
│   │   │   ├── DependInfo.cmake
│   │   │   ├── depend.make
│   │   │   ├── flags.make
│   │   │   ├── link.txt
│   │   │   └── progress.make
│   │   ├── Makefile2
│   │   ├── Makefile.cmake
│   │   ├── progress.marks
│   │   └── TargetDirectories.txt
│   ├── cmake_install.cmake
│   └── Makefile
├── CMakeLists.txt
├── main.cpp
```

本教程中所有的例子都采用out-of-source构建方式。

## 构建示例

构建示例程序的输出信息如下所示：

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
-- Build files have been written to: /workspace/cmake-examples/01-basic/hello_cmake/build

$ make
Scanning dependencies of target hello_cmake
[100%] Building CXX object CMakeFiles/hello_cmake.dir/hello_cmake.cpp.o
Linking CXX executable hello_cmake
[100%] Built target hello_cmake

$ ./hello_cmake
Hello CMake!
```