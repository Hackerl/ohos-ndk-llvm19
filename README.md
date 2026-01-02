# ohos-ndk-llvm19
Update OHOS NDK's LLVM to version 19.1.7 and make it compatible with CMake and vcpkg.

## Get Command Line Tools
Download and extract from the [official website](https://developer.huawei.com/consumer/cn/download/command-line-tools-for-hmos).

## Get Customized LLVM 19
Download and extract from the [official website](https://mirrors.huaweicloud.com/harmonyos/compiler/clang/19.1.7-410fca/linux/).

## Replace LLVM
Delete the `sdk/default/openharmony/native/llvm` directory in `Command Line Tools` and replace it with the newly downloaded version.

## Adapt CMake
```sh
sed -i 's/set(CMAKE_SYSTEM_NAME OHOS)/set(CMAKE_SYSTEM_NAME Linux)/' /opt/command-line-tools/sdk/default/openharmony/native/build/cmake/ohos.toolchain.cmake
```

## Add libc++ Header Path
```sh
sed -i 's/set(CMAKE_CXX_FLAGS "${OHOS_C_COMPILER_FLAGS} ${OHOS_CXX_COMPILER_FLAGS} ${CMAKE_CXX_FLAGS} -D__MUSL__)/set(CMAKE_CXX_FLAGS "${OHOS_C_COMPILER_FLAGS} ${OHOS_CXX_COMPILER_FLAGS} ${CMAKE_CXX_FLAGS} -D__MUSL__ -isystem ${OHOS_SDK_NATIVE}/llvm/include/libcxx-ohos/include/c++/v1")/' /opt/command-line-tools/sdk/default/openharmony/native/build/cmake/ohos.toolchain.cmake
```

## Compile
```sh
cat <<EOF > arm64-ohos.cmake
set(VCPKG_TARGET_ARCHITECTURE arm64)
set(VCPKG_CRT_LINKAGE dynamic)
set(VCPKG_LIBRARY_LINKAGE static)

set(VCPKG_CMAKE_SYSTEM_NAME Linux)
set(VCPKG_CHAINLOAD_TOOLCHAIN_FILE /opt/command-line-tools/sdk/default/openharmony/native/build/cmake/ohos.toolchain.cmake)
EOF

cmake -B build -DCMAKE_TOOLCHAIN_FILE="/opt/vcpkg/scripts/buildsystems/vcpkg.cmake" -DVCPKG_CHAINLOAD_TOOLCHAIN_FILE="/opt/command-line-tools/sdk/default/openharmony/native/build/cmake/ohos.toolchain.cmake" -DVCPKG_OVERLAY_TRIPLETS=$(pwd) -DVCPKG_TARGET_TRIPLET=arm64-ohos
```
