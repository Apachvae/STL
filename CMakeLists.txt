# Copyright (c) Microsoft Corporation.
# SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception

cmake_minimum_required(VERSION 3.15)
set(CMAKE_TRY_COMPILE_TARGET_TYPE STATIC_LIBRARY)
project(msvc_standard_libraries LANGUAGES CXX)

find_package(Boost REQUIRED)

set(VCLIBS_MIN_BOOST_VERSION 1.70.0)
if("${Boost_VERSION}" VERSION_LESS "${VCLIBS_MIN_BOOST_VERSION}")
    message(FATAL_ERROR "Detected Boost version is too old (older than ${VCLIBS_MIN_BOOST_VERSION}).")
endif()

set(VCLIBS_SUFFIX "_oss")

if(NOT DEFINED VCLIBS_TARGET_ARCHITECTURE)
    set(VCLIBS_TARGET_ARCHITECTURE "${CMAKE_CXX_COMPILER_ARCHITECTURE_ID}")
endif()

set(CMAKE_CXX_FLAGS "")
set(CMAKE_CXX_FLAGS_DEBUG "")
set(CMAKE_CXX_FLAGS_RELEASE "")
set(CMAKE_CXX_STANDARD_LIBRARIES "kernel32.lib")
set(CMAKE_CXX_STANDARD_LIBRARIES_INIT "kernel32.lib")
set(CMAKE_MSVC_RUNTIME_LIBRARY "")

if("${VCLIBS_TARGET_ARCHITECTURE}" MATCHES "^[xX]86$")
    set(VCLIBS_TARGET_ARCHITECTURE "x86")
    set(VCLIBS_I386_OR_AMD64 "i386")
    set(VCLIBS_X86_OR_X64 "x86")
    # Note that we set _WIN32_WINNT to a high level to make declarations available, but still engage downlevel
    # runtime dynamic linking by setting our own _STL_WIN32_WINNT back to Windows XP.
    add_compile_definitions(_X86_ _VCRT_WIN32_WINNT=_WIN32_WINNT_WINXP _STL_WIN32_WINNT=_WIN32_WINNT_WINXP
        _WIN32_WINNT=0x0602 NTDDI_VERSION=NTDDI_WIN8)
    add_compile_options(/arch:IA32)
elseif(VCLIBS_TARGET_ARCHITECTURE MATCHES "^[xX]64$")
    set(VCLIBS_TARGET_ARCHITECTURE "x64")
    set(VCLIBS_I386_OR_AMD64 "amd64")
    set(VCLIBS_X86_OR_X64 "x64")
    add_compile_definitions(_AMD64_ _VCRT_WIN32_WINNT=_WIN32_WINNT_WINXP _STL_WIN32_WINNT=_WIN32_WINNT_WINXP
        _WIN32_WINNT=0x0602 NTDDI_VERSION=NTDDI_WIN8)
elseif(VCLIBS_TARGET_ARCHITECTURE MATCHES "^[aA][rR][mM][vV]7$")
    set(VCLIBS_TARGET_ARCHITECTURE "arm")
    set(VCLIBS_I386_OR_AMD64 "arm")
    set(VCLIBS_X86_OR_X64 "arm")
    add_compile_definitions(_ARM_ _VCRT_WIN32_WINNT=_WIN32_WINNT_WIN8 _STL_WIN32_WINNT=_WIN32_WINNT_WIN8
        _WIN32_WINNT=0x0602 NTDDI_VERSION=NTDDI_WIN8)
    string(APPEND CMAKE_CXX_STANDARD_LIBRARIES " Synchronization.lib")
elseif(VCLIBS_TARGET_ARCHITECTURE MATCHES "^[aA][rR][mM]64$")
    set(VCLIBS_TARGET_ARCHITECTURE "arm64")
    set(VCLIBS_I386_OR_AMD64 "arm64")
    set(VCLIBS_X86_OR_X64 "arm64")
    add_compile_definitions(_ARM64_ _VCRT_WIN32_WINNT=_WIN32_WINNT_WIN10 _STL_WIN32_WINNT=_WIN32_WINNT_WIN10
        _WIN32_WINNT=0x0A00 NTDDI_VERSION=NTDDI_WIN10)
    string(APPEND CMAKE_CXX_STANDARD_LIBRARIES " Synchronization.lib")
else()
    message(FATAL_ERROR "Could not determine target architecture: VCLIBS_TARGET_ARCHITECTURE: ${VCLIBS_TARGET_ARCHITECTURE}")
endif()

add_compile_definitions(
    _ALLOW_ITERATOR_DEBUG_LEVEL_MISMATCH WIN32_LEAN_AND_MEAN STRICT _CRT_STDIO_ARBITRARY_WIDE_SPECIFIERS
    _CRT_DECLARE_NONSTDC_NAMES=1 )

# TRANSITION, /analyze ?
# TRANSITION, /d2FH4- ?
add_compile_options(/diagnostics:caret /W4 /WX /w14265 /w15038 /d1FastFail /guard:cf /Z7 /d2Zi+ /Gm- /Gy /Zp8 /std:c++latest /permissive- /Zc:threadSafeInit- /d2FH4-)

set(VCLIBS_DEBUG_OPTIONS "/Od")
set(VCLIBS_RELEASE_FLAGS "/O2;/Os") # TRANSITION: Potentially remove /Os

set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY "${PROJECT_BINARY_DIR}/out/lib/${VCLIBS_I386_OR_AMD64}")
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY "${PROJECT_BINARY_DIR}/out/lib/${VCLIBS_I386_OR_AMD64}")
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY "${PROJECT_BINARY_DIR}/out/bin/${VCLIBS_I386_OR_AMD64}")

set(CMAKE_STATIC_LINKER_FLAGS "/WX")
set(CMAKE_STATIC_LINKER_FLAGS_DEBUG "")
set(CMAKE_STATIC_LINKER_FLAGS_RELEASE "")
set(CMAKE_SHARED_LINKER_FLAGS "/DEBUG:FULL /WX /RELEASE /SUBSYSTEM:Console /NODEFAULTLIB /INCREMENTAL:NO /MANIFEST:NO /DLL /profile /guard:cf /DEBUGTYPE:cv,fixup /LARGEADDRESSAWARE")
set(CMAKE_SHARED_LINKER_FLAGS_DEBUG "")
set(CMAKE_SHARED_LINKER_FLAGS_RELEASE "")

get_filename_component(TOOLSET_BINARIES_DIR "${CMAKE_CXX_COMPILER}" DIRECTORY) # Example: $\VC\Tools\MSVC\14.23.27931\bin\Hostx86\x86
get_filename_component(TOOLSET_ROOT_DIR "${TOOLSET_BINARIES_DIR}" DIRECTORY) # $\VC\Tools\MSVC\14.23.27931\bin\Hostx86
get_filename_component(TOOLSET_ROOT_DIR "${TOOLSET_ROOT_DIR}" DIRECTORY) # $\VC\Tools\MSVC\14.23.27931\bin
get_filename_component(TOOLSET_ROOT_DIR "${TOOLSET_ROOT_DIR}" DIRECTORY) # $\VC\Tools\MSVC\14.23.27931

set(TOOLSET_LIB "${TOOLSET_ROOT_DIR}/lib/${VCLIBS_X86_OR_X64}")

add_subdirectory(stl)
