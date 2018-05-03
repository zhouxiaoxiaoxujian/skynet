CMAKE_MINIMUM_REQUIRED(VERSION 3.7)

set_property(GLOBAL PROPERTY C_STANDARD 99)

set(CMAKE_STATIC_LIBRARY_PREFIX "")

project(lua)

if (APPLE)
    set(CFLAGS "-DLUA_USE_MACOSX")
    set(LDFLAGS "-lreadline")
elseif(${CMAKE_SYSTEM_NAME} STREQUAL "Linux")
    set(CFLAGS "-DLUA_USE_LINUX")
    set(LDFLAGS "-lreadline -ldl -lm")
endif ()

include_directories(../../skynet-src)

add_compile_options(${CFLAGS})


#lua library

set(skynet-src ../../skynet-src)

aux_source_directory(. lua-src)

add_library(liblua STATIC ${lua-src})

#lua
add_executable(lua lua.c)

#luac
add_executable(luac luac.c)

target_link_libraries(lua liblua ${LDFLAGS})
target_link_libraries(luac liblua ${LDFLAGS})

