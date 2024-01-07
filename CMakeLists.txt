cmake_minimum_required(VERSION 3.10)

// !!!!!replace xxx_plugin to your project project name!!!!!
project(xxx_plugin_library VERSION 0.0.1)
set(PLUGIN_NAME "xxx_plugin")

if (MSVC)
    set(CC "gcc")
    set(CXX "g++")
    set(GOOS "windows")
    set(GOARCH "amd64")
    set(PLUGIN_SUFFIX ".dll")
    set(LIB_PATH "${CMAKE_CURRENT_BINARY_DIR}/go")
elseif (ANDROID)
    string(REPLACE "android-" "" SDK_VERSION ${ANDROID_PLATFORM})
    if (ANDROID_ABI STREQUAL "arm64-v8a")
        if(SDK_VERSION LESS 21)
            set(SDK_VERSION "21")
        endif()
        set(CC_PREFIX "aarch64-linux-android")
        set(GOARCH "arm64")
    elseif (ANDROID_ABI STREQUAL "armeabi-v7a")
        set(CC_PREFIX "armv7a-linux-androideabi")
        set(GOARCH "arm")
    elseif (ANDROID_ABI STREQUAL "x86_64")
        if(SDK_VERSION LESS 21)
            set(SDK_VERSION "21")
        endif()
        set(CC_PREFIX "x86_64-linux-android")
        set(GOARCH "amd64")
    elseif (ANDROID_ABI STREQUAL "x86")
        set(CC_PREFIX "i686-linux-android")
        set(GOARCH "386")
    endif ()
    set(CC "${ANDROID_TOOLCHAIN_ROOT}/bin/${CC_PREFIX}${SDK_VERSION}-clang")
    set(CXX "${ANDROID_TOOLCHAIN_ROOT}/bin/${CC_PREFIX}${SDK_VERSION}-clang++")
    set(GOOS "android")
    set(PLUGIN_PREFIX "lib")
    set(PLUGIN_SUFFIX ".so")
    set(LIB_PATH "${CMAKE_LIBRARY_OUTPUT_DIRECTORY}")
endif ()

set(PLUGIN_LIB "${PLUGIN_PREFIX}${PLUGIN_NAME}${PLUGIN_SUFFIX}")

add_custom_command(
        OUTPUT "${LIB_PATH}/${PLUGIN_LIB}"
        COMMAND
        ${CMAKE_COMMAND} -E env
        "CGO_ENABLED=1"
        "CC=${CC}"
        "CXX=${CXX}"
        "GOOS=${GOOS}"
        "GOARCH=${GOARCH}"
        go build -buildmode=c-shared -o "${LIB_PATH}/${PLUGIN_LIB}"
        WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/../src/
)

add_custom_target(shared_go_lib ALL DEPENDS "${LIB_PATH}/${PLUGIN_LIB}")

add_library(${PLUGIN_NAME} SHARED IMPORTED GLOBAL)

add_dependencies(${PLUGIN_NAME} shared_go_lib)

set_target_properties(
        ${PLUGIN_NAME}
        PROPERTIES
        IMPORTED_LOCATION "${LIB_PATH}/${PLUGIN_LIB}"
)
