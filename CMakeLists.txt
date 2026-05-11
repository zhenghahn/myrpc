#设置最低版本和项目名称
add_compile_definitions(ZOO_ENABLE_SYNC_API)
cmake_minimum_required(VERSION 3.0)
project(Krpc)

# 关键：启用CMP0079策略（允许跨目录链接库，解决krpc_core链接问题）
cmake_policy(SET CMP0079 NEW)

#设置全局的C++标准
set(CMAKE_CXX_STANDARD 11)#在这里我设置了11，代表使用c++11标准
set(CMAKE_CXX_STANDARD_REQUIRED True)#CMAKE_CXX_STANDARD_REQUIRED表示强制使用指定的标准

#设置头文件目录，供所有子项目使用
include_directories(${CMAKE_SOURCE_DIR}/src/include) #其中CMAKE_SOURCE_DIR表示当前目录

#查找porotbuf包
find_package(Protobuf REQUIRED)
include_directories(${Protobuf_INCLUDE_DIRS})#Protobuf_INCLUDE_DIRS表示protobuf头文件目录

#设置全局链接库
set(LIBS
    protobuf
    pthread
    zookeeper_mt
    muduo_net
    muduo_base
    glog
)

# 指定可执行文件输出到bin目录（所有子项目的可执行文件都输出到这里）
set(EXECUTABLE_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/bin)


#添加子目录
add_subdirectory(src)
add_subdirectory(example)
#nano CMakeLists.txt
