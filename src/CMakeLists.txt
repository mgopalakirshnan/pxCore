cmake_minimum_required(VERSION 2.8)
project(pxcore)

set(CMAKE_CXX_STANDARD 11)

option(WINDOWLESS_EGL "WINDOWLESS_EGL" OFF)
option(PXCORE_COMPILE_WARNINGS_AS_ERRORS "PXCORE_COMPILE_WARNINGS_AS_ERRORS" ON)

set(EXTDIR "${CMAKE_CURRENT_SOURCE_DIR}/../examples/pxScene2d/external")
set(NODEPATH "${EXTDIR}/libnode-v6.9.0/")
set(NODEINC "${NODEPATH}/src ${NODEPATH}/deps/uv/include ${NODEPATH}/deps/v8 ${NODEPATH}/deps/debugger-agent/include ${NODEPATH}/deps/v8/include ${NODEPATH}/deps/cares/include")
set(ZLIBINC "${EXTDIR}/zlib/")
set(CURLINC "${EXTDIR}/curl/include")
set(JPGINC "${EXTDIR}/jpg")
set(JPGTURBOINC "${EXTDIR}/libjpeg-turbo")
set(PNGINC "${EXTDIR}/png")

if (APPLE)
    message("Building pxcore for mac")
    set(CMAKE_MACOSX_RPATH 1)
    set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/../build/mac)
    set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/../build/mac)
    set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/../build/mac)
    add_definitions(-DPX_PLATFORM_MAC -DENABLE_RT_NODE -DENABLE_HTTP_CACHE -DRUNINMAIN)
    add_definitions(-D_DARWIN_USE_64_BIT_INODE=1 -DNODE_ARCH="x64" -DNODE_WANT_INTERNALS=1 -DV8_DEPRECATION_WARNINGS= -DNODE_SHARED_MODE
            -DNODE_USE_V8_PLATFORM=1 -DNODE_HAVE_I18N_SUPPORT=1 -DNODE_HAVE_SMALL_ICU=1 -DHAVE_INSPECTOR=1 -DV8_INSPECTOR_USE_STL=1
            -DV8_INSPECTOR_USE_OLD_STL=1 -DHAVE_OPENSSL=1 -DHAVE_DTRACE=1 -D__POSIX__ -DNODE_PLATFORM=darwin
            -DUCONFIG_NO_TRANSLITERATION=1 -DUCONFIG_NO_SERVICE=1 -DUCONFIG_NO_REGULAR_EXPRESSIONS=1 -DU_ENABLE_DYLOAD=0
            -DU_STATIC_IMPLEMENTATION=1 -DU_HAVE_STD_STRING=0 -DUCONFIG_NO_BREAK_ITERATION=0 -DUCONFIG_NO_LEGACY_CONVERSION=1
            -DUCONFIG_NO_CONVERSION=1 -DHTTP_PARSER_STRICT=0 -D_LARGEFILE_SOURCE -D_FILE_OFFSET_BITS=64
            -DRT_PLATFORM_LINUX -DENABLE_NODE_V_6_9 -DENABLE_DEBUG_MODE -DPNG_APNG_SUPPORTED)
    set(PXCORE_FILES mac/pxBufferNative.mm mac/pxOffscreenNative.mm mac/pxWindowNative.mm
            mac/pxEventLoopNative.mm mac/pxTimerNative.mm mac/pxTimerNative.mm
            mac/pxClipboardNative.mm mac/window.mm)
    set(PLATFORM_LINKER_FLAGS "-framework Cocoa -framework Foundation")
    include_directories(AFTER ${ZLIBINC} ${CURLINC} ${JPGINC} ${JPGTURBOINC} ${PNGINC})
elseif (CMAKE_HOST_SYSTEM_NAME STREQUAL "Linux")
    execute_process(COMMAND "hostname" OUTPUT_VARIABLE HOSTNAME)
    string(STRIP ${HOSTNAME} HOSTNAME)
    if (HOSTNAME STREQUAL "raspberrypi")
        message("Building pxcore for raspberry pi")
        set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/../build/egl)
        set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/../build/egl)
        set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/../build/egl)
        add_definitions(-DENABLE_NODE_V_6_9 -DNODE_PLATFORM=linux -DNODE_WANT_INTERNALS=1 -DV8_DEPRECATION_WARNINGS=1 -DNODE_SHARED_MODE -DNODE_USE_V8_PLATFORM=1 -DNODE_HAVE_I18N_SUPPORT=1 -DNODE_HAVE_SMALL_ICU=1 -DHAVE_INSPECTOR=1 -DV8_INSPECTOR_USE_STL=1 -DV8_INSPECTOR_USE_OLD_STL=1 -DENABLE_DEBUG_MODE)
        add_definitions(-DPX_PLATFORM_GENERIC_EGL -DRT_PLATFORM_LINUX -DENABLE_RT_NODE -DENABLE_HTTP_CACHE -DRUNINMAIN -DPNG_APNG_SUPPORTED)
        set(PXCORE_FILES gles/pxBufferNative.cpp gles/pxOffscreenNative.cpp gles/pxWindowNative.cpp
                gles/pxEventLoopNative.cpp gles/pxTimerNative.cpp
                gles/pxClipboardNative.cpp gles/pxEGLProviderRPi.cpp gles/LinuxInputEventDispatcher.cpp pxViewWindow.cpp)
        set(PLATFORM_LINKER_FLAGS)
        include_directories(AFTER /opt/vc/include /opt/vc/include/interface/vcos/pthreads
                /opt/vc/include/interface/vmcs_host/linux /opt/vc/include)
        include_directories(AFTER ${ZLIBINC} ${CURLINC} ${JPGINC} ${JPGTURBOINC} ${PNGINC})
    elseif (WINDOWLESS_EGL)
        message("Building pxcore for windowless egl")
        set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/../build/egl)
        set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/../build/egl)
        set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/../build/egl)
        add_definitions(-DENABLE_NODE_V_6_9 -DNODE_PLATFORM=linux -DNODE_WANT_INTERNALS=1 -DV8_DEPRECATION_WARNINGS=1 -DNODE_SHARED_MODE -DNODE_USE_V8_PLATFORM=1 -DNODE_HAVE_I18N_SUPPORT=1 -DNODE_HAVE_SMALL_ICU=1 -DHAVE_INSPECTOR=1 -DV8_INSPECTOR_USE_STL=1 -DV8_INSPECTOR_USE_OLD_STL=1)
        add_definitions(-DPX_PLATFORM_GENERIC_EGL -DRT_PLATFORM_LINUX -DPX_NO_WINDOW -DENABLE_RT_NODE -DENABLE_HTTP_CACHE -DRUNINMAIN -DRAPIDJSON_HAS_STDSTRING)
        if (DEFINED ENV{PXSCENE_DISABLE_APNG_SUPPORT})
            message("Not including APNG support")
        else ()
            message("Including APNG support")
            add_definitions(-DPNG_APNG_SUPPORTED)
        endif (DEFINED ENV{PXSCENE_DISABLE_APNG_SUPPORT})
        set(PXCORE_STAGING_LIST $ENV{PXCORE_STAGING})
        separate_arguments(PXCORE_STAGING_LIST)
        include_directories(AFTER ${PXCORE_STAGING_LIST})
        set(PXCORE_FILES gles/pxBufferNative.cpp gles/pxOffscreenNative.cpp
                gles/pxEventLoopNative.cpp gles/pxTimerNative.cpp gles/pxClipboardNative.cpp)
        set(PLATFORM_LINKER_FLAGS)
    else ()
        message("Building pxcore for glut")
        set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/../build/glut)
        set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/../build/glut)
        set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/../build/glut)
        add_definitions(-DENABLE_NODE_V_6_9 -DNODE_PLATFORM=linux -DNODE_WANT_INTERNALS=1 -DV8_DEPRECATION_WARNINGS=1 -DNODE_SHARED_MODE -DNODE_USE_V8_PLATFORM=1 -DNODE_HAVE_I18N_SUPPORT=1 -DNODE_HAVE_SMALL_ICU=1 -DHAVE_INSPECTOR=1 -DV8_INSPECTOR_USE_STL=1 -DV8_INSPECTOR_USE_OLD_STL=1 -DENABLE_DEBUG_MODE)
        add_definitions(-DPX_PLATFORM_GLUT -DRT_PLATFORM_LINUX -DENABLE_RT_NODE -DENABLE_HTTP_CACHE -DRUNINMAIN -DENABLE_LIBJPEG_TURBO -DPNG_APNG_SUPPORTED)
        set(PXCORE_FILES glut/pxBufferNative.cpp glut/pxOffscreenNative.cpp glut/pxWindowNative.cpp
                glut/pxEventLoopNative.cpp glut/pxTimerNative.cpp glut/pxClipboardNative.cpp pxViewWindow.cpp)
        set(PLATFORM_LINKER_FLAGS)
        include_directories(AFTER ${ZLIBINC} ${CURLINC} ${JPGINC} ${JPGTURBOINC} ${PNGINC} ${GLUTINC})
    endif (HOSTNAME STREQUAL "raspberrypi")
else ()
    message(FATAL_ERROR "Cannot build pxcore.  Unknown platform")
endif (APPLE)


set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -fPIC -g")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fPIC -Wno-deprecated -fpermissive -std=c++11 -g")

if (PXCORE_COMPILE_WARNINGS_AS_ERRORS)
  set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Werror")
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Werror")
endif (PXCORE_COMPILE_WARNINGS_AS_ERRORS)

include_directories(AFTER ${NODEPATH}/src ${NODEPATH}/deps/uv/include ${NODEPATH}/deps/v8 ${NODEPATH}/deps/debugger-agent/include ${NODEPATH}/deps/v8/include ${NODEPATH}/deps/cares/include)
include_directories(AFTER ${CMAKE_CURRENT_SOURCE_DIR})

set(PXCORE_FILES ${PXCORE_FILES} pxOffscreen.cpp pxWindowUtil.cpp
        rtFile.cpp rtLibrary.cpp rtNode.cpp
        rtPathUtils.cpp rtTest.cpp rtThreadPool.cpp
        rtThreadQueue.cpp rtThreadTask.cpp rtUrlUtils.cpp
        rtZip.cpp pxInterpolators.cpp pxMatrix4T.cpp pxUtil.cpp
        rtFileDownloader.cpp rtFileCache.cpp rtHttpCache.cpp unzip.c ioapi.c
        jsbindings/jsCallback.cpp jsbindings/rtFunctionWrapper.cpp jsbindings/rtObjectWrapper.cpp
        jsbindings/rtWrapperUtils.cpp
        unix/rtMutexNative.cpp unix/rtThreadPoolNative.cpp)

set(RTCORE_FILES utf8.c rtString.cpp rtLog.cpp rtValue.cpp rtObject.cpp rtError.cpp ioapi_mem.c)

add_library(pxCore ${PXCORE_FILES})
add_library(rtCore_s ${RTCORE_FILES})
add_library(rtCore SHARED ${RTCORE_FILES})
