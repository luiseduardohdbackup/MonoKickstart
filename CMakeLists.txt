CMAKE_MINIMUM_REQUIRED(VERSION 2.8.11)
PROJECT(kick)

if (APPLE)
    set(CMAKE_OSX_ARCHITECTURES i386 x86_64)
else()
    OPTION(FORCE32 "Force a 32bit compile on 64bit" OFF)
    if(FORCE32)
        SET(CMAKE_C_FLAGS "-m32")
    endif()
endif()

include_directories(${CMAKE_CURRENT_SOURCE_DIR}/precompiled/include/mono-2.0)

if (APPLE)
    SET(BIN_LIBROOT "osx")
    SET(CMAKE_EXECUTABLE_SUFFIX ".bin.osx")
    SET(BIN_RPATH "@executable_path/osx")
elseif (CMAKE_SIZEOF_VOID_P MATCHES "8" AND NOT(FORCE32))
    SET(BIN_LIBROOT "lib64")
    SET(CMAKE_EXECUTABLE_SUFFIX ".bin.x86_64")
    SET(BIN_RPATH "\$ORIGIN/lib64")
    SET(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -include ${CMAKE_CURRENT_SOURCE_DIR}/symlower.h")
else()
    SET(BIN_LIBROOT "lib")
    SET(CMAKE_EXECUTABLE_SUFFIX ".bin.x86")
    SET(BIN_RPATH "\$ORIGIN/lib")
    SET(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -D_FILE_OFFSET_BITS=64")
endif()

ADD_DEFINITIONS(-DENABLE_BINRELOC)

set(CMAKE_SKIP_BUILD_RPATH TRUE)
set(CMAKE_BUILD_WITH_INSTALL_RPATH TRUE)
set(CMAKE_INSTALL_RPATH ${BIN_RPATH})
set(CMAKE_INSTALL_RPATH_USE_LINK_PATH FALSE)

SET(SOURCE
    kick.c
    binreloc.c
)

LINK_DIRECTORIES(
    ${CMAKE_SOURCE_DIR}/precompiled/${BIN_LIBROOT}
)

ADD_EXECUTABLE(kick
    ${SOURCE}
)
TARGET_LINK_LIBRARIES(kick
    mono-2.0
)

# SGEN version
ADD_EXECUTABLE(kick-sgen
    ${SOURCE}
)
TARGET_LINK_LIBRARIES(kick-sgen
    monosgen-2.0
)

