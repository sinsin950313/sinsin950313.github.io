---
title: "VSCode에서 CMake로 Build하기"
date: 2024-01-19
categories: ["Development", "Non"]
tags: ["vscode", "cmake"]
---
VSCode에서 CMake Extension을 설치할 경우 Command창에서 CMake: Quick Start가 가능

CMake:Quick Start를 통해 생성된 기본 상황에서부터 시작

Directory 구조
```
Project Directory
|-  CMakeLists.txt
|-  Main.cpp
|-  SubDirectory
    |- Source1.cpp
    |- Source1.h
```

```cmake
file(GLOB_RECURSE SRC_FILES CONFIGURE_DEPENDS
 ${CMAKE_CURRENT_SOURCE_DIR}/SubDirectory/*.cpp
)
```
위 구문을 통해 SubDirectory에 있는 cpp파일들을 SRC_FILES라는 변수에 저장 가능

```cmake
add_executable(${ProjectName} Main.cpp ${SRC_FILES})
```
SRC_FILES를 통해 읽어들인 cpp파일들을 executable에 모두 추가

```cmake
target_include_directories(${ProjectName} PUBLIC ${CMAKE_SOURCE_DIR}/SubDirectory)
```
Include 필요 시 추가

```cmake
target_compile_features(VelogToGitBlog PRIVATE cxx_std_17)
```
fileSystem의 경우 c++17 이상을 요구하므로 구문 추가

최종 구조
```cmake
cmake_minimum_required(VERSION 3.0.0)
project(VelogToGitBlog VERSION 0.1.0 LANGUAGES C CXX)

include(CTest)
enable_testing()

file(GLOB_RECURSE SRC_FILES CONFIGURE_DEPENDS
 ${CMAKE_CURRENT_SOURCE_DIR}/Project1/*.cpp
)

add_executable(VelogToGitBlog Main.cpp ${SRC_FILES})

set(CPACK_PROJECT_NAME ${PROJECT_NAME})
set(CPACK_PROJECT_VERSION ${PROJECT_VERSION})
# set(CMAKE_CXX_STANDARD 17)

target_compile_features(VelogToGitBlog PRIVATE cxx_std_17)

include(CPack)

target_include_directories(VelogToGitBlog PUBLIC ${CMAKE_SOURCE_DIR}/Project1)
```
