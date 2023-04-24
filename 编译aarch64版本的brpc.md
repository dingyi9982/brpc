# 编译aarch64版本的brpc

以/opt/aarch64-mr813-linux-gnu交叉编译工具链为例。如果不使用工具链中自带的protoc，需要确保protobuf库和使用的protoc版本兼容。

1. 编译leveldb

```shell
cd ~
git clone https://github.com/google/leveldb.git
cd leveldb
CC=/opt/aarch64-mr813-linux-gnu/bin/aarch64-mr813-linux-gnu-gcc CXX=/opt/aarch64-mr813-linux-gnu/bin/aarch64-mr813-linux-gnu-g++ make -j8
```

2. 编译brpc

```shell
mkdir build && cd build
cmake -DBUILD_SHARED_LIBS=1 -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=`pwd`/out -DCMAKE_TOOLCHAIN_FILE=~/robot/cmake/aarch64-mr813-linux-gnu.cmake -DPROTOBUF_PROTOC_EXECUTABLE=/opt/aarch64-mr813-linux-gnu/bin/protoc -DWITH_DEBUG_SYMBOLS=0 -DBUILD_BRPC_TOOLS=0 -DLEVELDB_INCLUDE_PATH=$HOME/leveldb/include -DLEVELDB_LIB=$HOME/leveldb/out-shared/libleveldb.so ..
make install/strip -j8
```shell

生成的库在build/out目录下。

3. 编译example/streaming_echo_c++

```shell
cd example/streaming_echo_c++
mkdir build && cd build
cmake -DCMAKE_BUILD_TYPE=Release -DCMAKE_TOOLCHAIN_FILE=~/robot/cmake/aarch64-mr813-linux-gnu.cmake -DPROTOBUF_PROTOC_EXECUTABLE=/opt/aarch64-mr813-linux-gnu/bin/protoc -DBRPC_INCLUDE_PATH=$HOME/brpc/build/out/include -DBRPC_LIB=$HOME/brpc/build/out/lib/libbrpc.so -DLEVELDB_INCLUDE_PATH=$HOME/leveldb/include -DLEVELDB_LIB=$HOME/leveldb/out-shared/libleveldb.so ..
make -j8
```

4. aarch64-mr813-linux-gnu.cmake内容如下：
PS:

```shell
set(CMAKE_SYSTEM_NAME Linux)
set(CMAKE_SYSTEM_PROCESSOR aarch64)
set(CMAKE_LIBRARY_ARCHITECTURE aarch64-mr813-linux-gnu)

set(TOOLCHAIN_TOP_PATH         "/opt/${CMAKE_LIBRARY_ARCHITECTURE}")
set(TOOLCHAIN_BIN_PATH         ${TOOLCHAIN_TOP_PATH}/bin)
set(TOOLCHAIN_SYSROOT_PATH     ${TOOLCHAIN_TOP_PATH}/${CMAKE_LIBRARY_ARCHITECTURE}/sysroot)
set(CMAKE_C_COMPILER           ${TOOLCHAIN_BIN_PATH}/${CMAKE_LIBRARY_ARCHITECTURE}-gcc)
set(CMAKE_CXX_COMPILER         ${TOOLCHAIN_BIN_PATH}/${CMAKE_LIBRARY_ARCHITECTURE}-g++)

if (NOT EXISTS ${TOOLCHAIN_TOP_PATH}/.toolchain.location)
    string(ASCII 27 Esc)
    MESSAGE (FATAL_ERROR "\n\n${Esc}[31mPlease run the following command to initialize Toolchain:\n\tcd ${TOOLCHAIN_TOP_PATH}\n\t./.toolchain.init${Esc}[m\n\n")
endif()

#add_definitions(-D_GLIBCXX_USE_CXX11_ABI=0)
set(CMAKE_FIND_ROOT_PATH    ${TOOLCHAIN_SYSROOT_PATH})
set(CMAKE_SYSROOT           ${TOOLCHAIN_SYSROOT_PATH})
set(CMAKE_CROSSCOMPILING true)
set(CMAKE_FIND_ROOT_PATH_MODE_PROGRAM NEVER)
set(CMAKE_FIND_ROOT_PATH_MODE_LIBRARY ONLY)
set(CMAKE_FIND_ROOT_PATH_MODE_INCLUDE ONLY)
```
