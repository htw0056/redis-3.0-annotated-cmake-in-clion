# CLion调试redis源码

## 背景

`CLion`使用`CMake`来管理编译，而redis源码本身使用`make`，因此直接将redis源码导入`CLion`无法直接运行，需要配置`CMake`。

由于学习过程中参考的书籍为[《Redis 设计与实现》](http://redisbook.com/)，因此源码版本也跟本书保持一致。

## 步骤

##### 1. 下载源码

```shell
git clone git@github.com:huangz1990/redis-3.0-annotated.git
```

##### 2. deps/hiredis目录下新增`CMakeLists.txt`

```cmake
add_library(hiredis STATIC
        hiredis.c
        net.c
        dict.c
        net.c
        sds.c
        async.c
        )
```

##### 2. deps/linenoise目录下新增`CMakeLists.txt`

```cmake
add_library(linenoise
        linenoise.c
        )
```

##### 3. deps/lua目录下新增`CMakeLists.txt`

```cmake
set(LUA_SRC
        src/lauxlib.c
        src/liolib.c
        src/lopcodes.c
        src/lstate.c
        src/lobject.c
        src/print.c
        src/lmathlib.c
        src/loadlib.c
        src/lvm.c
        src/lfunc.c
        src/lstrlib.c
        src/lua.c
        src/linit.c
        src/lstring.c
        src/lundump.c
        src/luac.c
        src/ltable.c
        src/ldump.c
        src/loslib.c
        src/lgc.c
        src/lzio.c
        src/ldblib.c
        src/strbuf.c
        src/lmem.c
        src/lcode.c
        src/ltablib.c
        src/lua_struct.c
        src/lapi.c
        src/lbaselib.c
        src/lua_cmsgpack.c
        src/ldebug.c
        src/lparser.c
        src/lua_cjson.c
        src/llex.c
        src/ltm.c
        src/ldo.c
        )

add_library(lua STATIC ${LUA_SRC})
```

##### 4. deps目录下新增`CMakeLists.txt`

```cmake
add_subdirectory(hiredis)
add_subdirectory(linenoise)
add_subdirectory(lua)
```

##### 5. redis-3.0-annotated目录下新增`CMakeLists.txt5`

```cmake
cmake_minimum_required(VERSION 3.0 FATAL_ERROR)

project(redis-3.0-annotated-cmake-in-clion VERSION 3.0)

if (NOT CMAKE_BUILD_TYPE)
    message(STATUS "No build type defined; defaulting to 'Debug'")
    set(CMAKE_BUILD_TYPE "Debug" CACHE STRING
            "The type of build. Possible values are: Debug, Release, RelWithDebInfo and MinSizeRel.")
endif()

message(STATUS "Host is: ${CMAKE_HOST_SYSTEM}.  Build target is: ${CMAKE_SYSTEM}")
get_filename_component(REDIS_ROOT "${CMAKE_CURRENT_SOURCE_DIR}" ABSOLUTE)
message(STATUS "Project root directory is: ${REDIS_ROOT}")

# Just for debugging when handling a new platform.
if (false)
    message("C++ compiler supports these language features:")
    foreach(i ${CMAKE_CXX_COMPILE_FEATURES})
        message("  ${i}")
    endforeach()
endif()

message(STATUS "Generating release.h...")
execute_process(
        COMMAND sh -c ./mkreleasehdr.sh
        WORKING_DIRECTORY ${REDIS_ROOT}/src/
)
add_subdirectory(deps)

set(SRC_SERVER
        src/adlist.c src/ae.c src/anet.c src/dict.c src/redis.c src/sds.c src/zmalloc.c
        src/lzf_c.c src/lzf_d.c src/pqsort.c src/zipmap.c src/sha1.c src/ziplist.c src/release.c src/networking.c src/util.c src/object.c src/db.c src/replication.c src/rdb.c src/t_string.c src/t_list.c src/t_set.c src/t_zset.c src/t_hash.c src/config.c src/aof.c src/pubsub.c src/multi.c src/debug.c src/sort.c src/intset.c src/syncio.c src/cluster.c src/crc16.c src/endianconv.c src/slowlog.c src/scripting.c src/bio.c src/rio.c src/rand.c src/memtest.c src/crc64.c src/bitops.c src/sentinel.c src/notify.c src/setproctitle.c src/blocked.c src/hyperloglog.c
        )

set(SRC_CLI
        src/anet.c src/sds.c src/adlist.c src/redis-cli.c src/zmalloc.c src/release.c src/anet.c src/ae.c src/crc64.c
        )


if (${CMAKE_SYSTEM_NAME} MATCHES "Linux")
    # better not to work with jemalloc
endif()

add_executable(redis-server ${SRC_SERVER})
add_executable(redis-cli ${SRC_CLI})

set_property(TARGET redis-server PROPERTY C_STANDARD 99)
set_property(TARGET redis-server PROPERTY CXX_STANDARD 11)
set_property(TARGET redis-server PROPERTY CXX_STANDARD_REQUIRED ON)

set_property(TARGET redis-cli PROPERTY C_STANDARD 99)
set_property(TARGET redis-cli PROPERTY CXX_STANDARD 11)
set_property(TARGET redis-cli PROPERTY CXX_STANDARD_REQUIRED ON)


target_include_directories(redis-server
        PRIVATE ${REDIS_ROOT}/deps/hiredis
        PRIVATE ${REDIS_ROOT}/deps/linenoise
        # PRIVATE ${REDIS_ROOT}/deps/jemalloc
        PRIVATE ${REDIS_ROOT}/deps/lua/src
        )

target_include_directories(redis-cli
        PRIVATE ${REDIS_ROOT}/deps/hiredis
        PRIVATE ${REDIS_ROOT}/deps/linenoise
        # PRIVATE ${REDIS_ROOT}/deps/jemalloc
        PRIVATE ${REDIS_ROOT}/deps/lua/src
        )


target_link_libraries(redis-server
        PRIVATE pthread
        PRIVATE m
        PRIVATE lua
        PRIVATE linenoise
        PRIVATE hiredis
        )

target_link_libraries(redis-cli
        PRIVATE pthread
        PRIVATE m
        PRIVATE linenoise
        PRIVATE hiredis
        )

link_directories(deps/hiredis/ deps/linenoise/ diredeps/lua/src)
```

##### 6. 导入CLion

将项目导入CLion，导入时选择不覆盖已有的CMakeLists.txt。

##### 7. 运行

在CLion中选择`redis-server`，选择运行/调试，即可成功运行。

## 源码地址

`https://github.com/htw0056/redis-3.0-annotated-cmake-in-clion`，clone该项目导入CLion即可直接运行。