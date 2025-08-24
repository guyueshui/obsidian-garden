---
{"dg-publish":true,"permalink":"/__zettel/202306262103cmake-template/","title":202306262103,"tags":["cmake"],"created":"2023-06-26T21:03:02+08:00"}
---


```cmake
cmake_minimum_required(VERSION 3.20)
project(GREDialing VERSION 0.1)

message(STATUS "CMake version: " ${CMAKE_VERSION})
if(NOT ${CMAKE_VERSION} VERSION_LESS "3.2")
    set(CMAKE_CXX_STANDARD 17)
    set(CMAKE_CXX_STANDARD_REQUIRED ON)
else()
    message(STATUS "Checking compiler flags for C++11 support.")
    # Set C++11 support flags for various compilers
    include(CheckCXXCompilerFlag)
    check_cxx_compiler_flag("-std=c++11" COMPILER_SUPPORTS_CXX11)
    check_cxx_compiler_flag("-std=c++0x" COMPILER_SUPPORTS_CXX0X)
    if(COMPILER_SUPPORTS_CXX11)
        message(STATUS "C++11 is supported.")
        if(${CMAKE_SYSTEM_NAME} MATCHES "Darwin")
            set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11 -stdlib=libc++")
        else()
            set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")
        endif()
    elseif(COMPILER_SUPPORTS_CXX0X)
        message(STATUS "C++0x is supported.")
        if(${CMAKE_SYSTEM_NAME} MATCHES "Darwin")
            set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++0x -stdlib=libc++")
        else()
            set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++0x")
        endif()
    else()
        message(STATUS "The compiler ${CMAKE_CXX_COMPILER} has no C++11 support. Please use a different C++ compiler.")
    endif()
endif()


 
# 禁止 C++ assert terminate 程序
# add_definitions(-DNDEBUG)

# ASIO 不使用 Boost 库
#add_definitions(-DASIO_STANDALONE)

# Spdlog 使用外部 Fmt 库
#add_definitions(-DSPDLOG_FMT_EXTERNAL)

# Jwt-cpp 使用外部 Json 库
# ADD_DEFINITIONS(-DJWT_CPP_JSON_EXTERNAL)

# add_definitions(-DGLOG_ON)

# 生成编译命令文件，给 YCM/Clangd 使用
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

ADD_COMPILE_OPTIONS(-g)

#设置输出目录
# set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
# set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
# set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${PROJECT_SOURCE_DIR}/bin)


#加入包含目录
# include_directories(${CMAKE_CURRENT_SOURCE_DIR}/service/src/main/cpp/decryptSDK/include) 
include_directories(decryptSDK/include) 
include_directories(/home/hubingbing/rapidjson/include)
include_directories(/home/hubingbing/concurrentqueue)
# include_directories(/opt/Euler_compile_env/usr/src/kernels/4.19.90-vhulk2107.1.0.h699.eulerosv2r10.aarch64/include)


#静态库目录
#link_directories(/usr/lib64/mysql)

#logger lib 
#add_subdirectory(decryptSDK)
 

# aux_source_directory(decryptSDK SRCS) #加入目录下所有源码
# add_executable(demo ${SRCS}) #生成可执行文件
# target_link_libraries(demo logger)  #链接logger库

set(EXECUTABLE_OUTPUT_PATH "${CMAKE_CURRENT_SOURCE_DIR}/build")
message(STATUS "binary dir is ${EXECUTABLE_OUTPUT_PATH}")

set(BUILD_SCRATCH_TARGET 0)

if(BUILD_SCRATCH_TARGET)
    file(GLOB_RECURSE scratch_srcs scratch/*.cpp)
    # message(STATUS "all scratchs: ${scratch_srcs}")
    foreach(srcfile IN LISTS scratch_srcs)
        # Get file name without directory
        get_filename_component(elfname ${srcfile} NAME_WE)
        message(STATUS "${elfname} .. ${srcfile}")
        add_executable(${elfname} ${srcfile})
        target_link_libraries(${elfname} pthread)
    endforeach()
endif()


# add_executable(main ../main.cpp)
# add_executable(server udp_server.cpp) 
# add_executable(ConfigReader ConfigReader.cpp) 
# add_executable(packet_handler packet_handler.cpp)

set(BUILD_MAIN_TARGET 1)

if(BUILD_MAIN_TARGET)
    set(target main)

    set(src_lst main.cpp ConfigReader.cpp packet_handler.cpp SslClient.cpp packet.cpp)
    add_executable(${target} ${src_lst})

    aux_source_directory(decryptSDK KMC_DIR)
    target_sources(${target} PRIVATE ${KMC_DIR})
    target_include_directories(${target} PRIVATE /home/hubingbing/cloud-frame-kmc/include/)
    target_link_directories(${target} PRIVATE /home/hubingbing/cloud-frame-kmc/lib_so/)
    target_link_libraries(${target} kmcjni)

    target_link_libraries(${target} ssl crypto pthread)
endif()
# target_link_libraries(debug pthread)
# target_link_libraries(server pthread)
```

new:
```cmake
cmake_minimum_required(VERSION 3.20)
project(GREDialing VERSION 0.1)

message(STATUS "CMake version: " ${CMAKE_VERSION})
if(NOT ${CMAKE_VERSION} VERSION_LESS "3.2")
    set(CMAKE_CXX_STANDARD 17)
    set(CMAKE_CXX_STANDARD_REQUIRED ON)
else()
    message(STATUS "Checking compiler flags for C++11 support.")
    # Set C++11 support flags for various compilers
    include(CheckCXXCompilerFlag)
    check_cxx_compiler_flag("-std=c++11" COMPILER_SUPPORTS_CXX11)
    check_cxx_compiler_flag("-std=c++0x" COMPILER_SUPPORTS_CXX0X)
    if(COMPILER_SUPPORTS_CXX11)
        message(STATUS "C++11 is supported.")
        if(${CMAKE_SYSTEM_NAME} MATCHES "Darwin")
            set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11 -stdlib=libc++")
        else()
            set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")
        endif()
    elseif(COMPILER_SUPPORTS_CXX0X)
        message(STATUS "C++0x is supported.")
        if(${CMAKE_SYSTEM_NAME} MATCHES "Darwin")
            set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++0x -stdlib=libc++")
        else()
            set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++0x")
        endif()
    else()
        message(STATUS "The compiler ${CMAKE_CXX_COMPILER} has no C++11 support. Please use a different C++ compiler.")
    endif()
endif()


 
# 禁止 C++ assert terminate 程序
# add_definitions(-DNDEBUG)

# ASIO 不使用 Boost 库
#add_definitions(-DASIO_STANDALONE)

# Spdlog 使用外部 Fmt 库
#add_definitions(-DSPDLOG_FMT_EXTERNAL)

# Jwt-cpp 使用外部 Json 库
# ADD_DEFINITIONS(-DJWT_CPP_JSON_EXTERNAL)

# add_definitions(-DGLOG_ON)

# 生成编译命令文件，给 YCM/Clangd 使用
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

ADD_COMPILE_OPTIONS(-g)

#设置输出目录
# set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
# set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
# set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${PROJECT_SOURCE_DIR}/bin)


#加入包含目录
# include_directories(${CMAKE_CURRENT_SOURCE_DIR}/service/src/main/cpp/decryptSDK/include) 
# include_directories(decryptSDK/include)
# include_directories(/home/hubingbing/concurrentqueue)
include_directories(json/include)  # for baize json


#静态库目录
#link_directories(/usr/lib64/mysql)

#logger lib 
#add_subdirectory(decryptSDK)
# add_subdirectory(json)

aux_source_directory(json/src BAIZE_JSON_SRC)
# aux_source_directory(decryptSDK SRCS) #加入目录下所有源码
# add_executable(demo ${SRCS}) #生成可执行文件
# target_link_libraries(demo logger)  #链接logger库

set(EXECUTABLE_OUTPUT_PATH "${CMAKE_CURRENT_SOURCE_DIR}/build")
message(STATUS "binary dir is ${EXECUTABLE_OUTPUT_PATH}")

set(BUILD_SCRATCH_TARGET 1)
if(BUILD_SCRATCH_TARGET)
    file(GLOB_RECURSE scratch_srcs scratch/*.cpp)
    # message(STATUS "all scratchs: ${scratch_srcs}")
    foreach(srcfile IN LISTS scratch_srcs)
        # Get file name without directory
        get_filename_component(elfname ${srcfile} NAME_WE)
        message(STATUS "${elfname} .. ${srcfile}")
        if (${elfname} STREQUAL "test_logger")
            add_executable(${elfname} ${srcfile} SimpleLogger.cpp)
        elseif(${elfname} STREQUAL "cqueue_test")
            add_executable(${elfname} ${srcfile} helper.cpp SimpleLogger.cpp packet_handler.cpp ConfigReader.cpp packet.cpp ${BAIZE_JSON_SRC})
            target_link_libraries(${elfname} securec)
        elseif(${elfname} STREQUAL "baizejson")
            add_executable(${elfname} ${srcfile} ${BAIZE_JSON_SRC})
            target_link_libraries(${elfname} securec)
        else()
            add_executable(${elfname} ${srcfile})
        endif()
        target_link_libraries(${elfname} pthread)
    endforeach()
endif()


# add_executable(main ../main.cpp)
# add_executable(server udp_server.cpp) 
# add_executable(ConfigReader ConfigReader.cpp) 
# add_executable(packet_handler packet_handler.cpp)

set(LIBKMC_PATH decryptSDK)
set(KMC_LINK_OPTION "STATIC")
set(BUILD_MAIN_TARGET 1)
if(BUILD_MAIN_TARGET)
    set(target gredialingagent)  # 可执行文件名
    if(${CMAKE_BUILD_TYPE} MATCHES "RELEASE")
        message(STATUS "build with release, add compile options to remove debug info...")
        # 安全编译选项：http://3ms.huawei.com/hi/group/2028271/wiki_6704184.html
        set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fstack-protector-strong -Wl,-z,relro,-z,now,-z,noexecstack -s -fPIE -pie")
    endif()
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} \
        -Wall -Wextra -Weffc++ -Wconversion -Wsign-conversion \
        -Wtrampolines -Wdate-time -Wfloat-equal -Wswitch-default \
        -Wshadow -Wno-return-local-addr -Wcast-qual -Wcast-align -Wvla -Wunused \
        -Wundef -Wnon-virtual-dtor -Woverloaded-virtual -pedantic-errors -Wsizeof-array-argument \
    ")
    # message(STATUS "compile options are ${CMAKE_CXX_FLAGS}")

    aux_source_directory(./ src_lst)
    # message(STATUS "source list of ${target}: ${src_lst}")
    add_executable(${target} ${src_lst})
    target_sources(${target} PRIVATE ${BAIZE_JSON_SRC})


    if(${KMC_LINK_OPTION} STREQUAL "STATIC")
        add_subdirectory(decryptSDK)  # 编译kmc静态库
        target_link_directories(${target} PRIVATE ${LIBKMC_PATH}/lib/)
        # target_link_libraries(${target} decrypt libkmcext.a libkmc.a libsecurec.a libsdp.a libssl.a libcrypto.a dl)
        target_link_libraries(${target} decrypt kmcext kmc securec sdp ssl crypto dl)
    else()
        aux_source_directory(decryptSDK KMC_DIR)
        target_sources(${target} PRIVATE ${KMC_DIR})
        target_include_directories(${target} PRIVATE ${LIBKMC_PATH}/include/)
        target_link_directories(${target} PRIVATE /home/hubingbing/cloud-frame-kmc/lib_so/)
        target_link_libraries(${target} kmcjni)
    endif()

    # find_library(KMC_STATIC_LIB kmc sdp /home/hubingbing/cloud-frame-kmc/lib_a/)
    # message(STATUS "finded lib: ${KMC_STATIC_LIB}")

    target_link_libraries(${target} pthread)
endif()
# target_link_libraries(debug pthread)
# target_link_libraries(server pthread)
```

see also [202304230920 cmake learn](202304230920%20cmake%20learn)