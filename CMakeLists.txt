# skate1 - ReXGlue Recompiled Project
#
# SDK boilerplate lives in generated/rexglue.cmake.

cmake_minimum_required(VERSION 3.25)
project(skate1 LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 23)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# Default to the SDK submodule; pass -DREXSDK_DIR=... to use another checkout.
if(NOT REXSDK_DIR AND EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/third_party/rexglue-sdk/CMakeLists.txt")
    set(REXSDK_DIR "${CMAKE_CURRENT_SOURCE_DIR}/third_party/rexglue-sdk" CACHE PATH "Path to rexglue-sdk source tree")
endif()

include(generated/rexglue.cmake)

# Sources
set(SKATE1_SOURCES
    src/main.cpp
)

# Exe/window icon. The icon is the game's cover art, so it is not committed:
# drop your own src/skate1.ico in place to embed it.
if(WIN32 AND EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/src/skate1.ico")
    enable_language(RC)
    list(APPEND SKATE1_SOURCES src/skate1.rc)
endif()

if(WIN32)
    add_executable(skate1 WIN32 ${SKATE1_SOURCES})
else()
    add_executable(skate1 ${SKATE1_SOURCES})
endif()

# Since SDK 0.10 the Xenos GPU emulation is a runtime-loaded plugin; this stages
# rexgpu-xenos next to the exe (the runtime config selects it via gpu_plugin).
rexglue_setup_target(skate1 GPU_PLUGINS xenos)

# The SDK writes librexruntime into its own out/<platform> folder, but the host
# only looks next to itself (@loader_path / $ORIGIN). Windows stages it via
# TARGET_RUNTIME_DLLS in the SDK; do the same here for macOS and Linux.
if(NOT WIN32 AND TARGET rexruntime)
    add_custom_command(TARGET skate1 POST_BUILD
        COMMAND ${CMAKE_COMMAND} -E copy_if_different
            $<TARGET_FILE:rexruntime> $<TARGET_FILE_DIR:skate1>
        VERBATIM
    )
endif()
