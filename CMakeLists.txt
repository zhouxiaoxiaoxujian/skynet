cmake_minimum_required(VERSION 3.7)

project(skynet)

set(CMAKE_SHARED_LIBRARY_PREFIX "")
set(CMAKE_SHARED_LIBRARY_SUFFIX ".so")

add_subdirectory(3rd/lua 3rd/lua)

include_directories(skynet-src 3rd/lua)

if (APPLE)
    add_compile_options(-DNOUSE_JEMALLOC)
    link_libraries(-Wl,-undefined,dynamic_lookup)
elseif(${CMAKE_SYSTEM_NAME} STREQUAL "Linux")
	add_custom_target(libjemalloc ./autogen.sh --with-jemalloc-prefix=je_ --disable-valgrind
		COMMAND make
		WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}/3rd/jemalloc) 
    include_directories(3rd/jemalloc/include/jemalloc)
    link_libraries(-lpthread -lm)
endif (APPLE)


#copy examples

add_custom_target(examples cp -r ${CMAKE_SOURCE_DIR}/examples ${CMAKE_BINARY_DIR})

#file(COPY examples DESTINATION ${CMAKE_BINARY_DIR})

#C service

set(cservice gate harbor logger snlua)

foreach(service ${cservice})

    add_library(${service} SHARED service-src/service_${service}.c)

    set_target_properties(${service}
            PROPERTIES
            LIBRARY_OUTPUT_DIRECTORY cservice)
endforeach(service)

#Lua services
add_custom_target(luaservice cp -r ${CMAKE_SOURCE_DIR}/service ${CMAKE_BINARY_DIR})

#Lua c lib

set(clib lualib-src/lua-skynet.c lualib-src/lua-seri.c
        lualib-src/lua-socket.c
        lualib-src/lua-mongo.c
        lualib-src/lua-netpack.c
        lualib-src/lua-memory.c
        lualib-src/lua-profile.c
        lualib-src/lua-multicast.c
        lualib-src/lua-cluster.c
        lualib-src/lua-sharedata.c
        lualib-src/lua-stm.c
        lualib-src/lua-mysqlaux.c
        lualib-src/lua-datasheet.c
        lualib-src/lua-debugchannel.c)


add_library(lua-skynet SHARED ${clib})

set_target_properties(lua-skynet
        PROPERTIES
        LIBRARY_OUTPUT_DIRECTORY luaclib
        LIBRARY_OUTPUT_NAME skynet)

#client lua c lib
add_library(lua-client SHARED lualib-src/lua-clientsocket.c lualib-src/lua-crypt.c lualib-src/lsha1.c)

set_target_properties(lua-client
        PROPERTIES
        LIBRARY_OUTPUT_DIRECTORY luaclib
        LIBRARY_OUTPUT_NAME client)

#lepg lua c lib
aux_source_directory(3rd/lpeg lpeg-src)

add_library(lua-lpeg SHARED ${lpeg-src})

set_target_properties(lua-lpeg
        PROPERTIES
        LIBRARY_OUTPUT_DIRECTORY luaclib
        LIBRARY_OUTPUT_NAME lpeg)

#sproto lua c lib
aux_source_directory(lualib-src/sproto sproto-src)

add_library(lua-sproto SHARED ${sproto-src})

set_target_properties(lua-sproto
        PROPERTIES
        LIBRARY_OUTPUT_DIRECTORY luaclib
        LIBRARY_OUTPUT_NAME sproto)

#bson lua c lib
add_library(lua-bson SHARED lualib-src/lua-bson.c)

set_target_properties(lua-bson
        PROPERTIES
        LIBRARY_OUTPUT_DIRECTORY luaclib
        LIBRARY_OUTPUT_NAME bson)

#md5 lua c lib
aux_source_directory(3rd/lua-md5 md5-src)

add_library(lua-md5 SHARED ${md5-src})

set_target_properties(lua-md5
        PROPERTIES
        LIBRARY_OUTPUT_DIRECTORY luaclib
        LIBRARY_OUTPUT_NAME md5)

#Lua lua lib
add_custom_target(lualib cp -r ${CMAKE_SOURCE_DIR}/lualib ${CMAKE_BINARY_DIR})

#skynet executable

aux_source_directory(skynet-src skynet-src)

add_executable(skynet ${skynet-src})

target_link_libraries(skynet liblua)

if (${CMAKE_SYSTEM_NAME} STREQUAL "Linux")
	target_link_libraries(skynet ${CMAKE_SOURCE_DIR}/3rd/jemalloc/lib/libjemalloc_pic.a -lpthread -lm -ldl)		
endif (${CMAKE_SYSTEM_NAME} STREQUAL "Linux")

add_dependencies(skynet liblua lua-skynet lua-bson lua-client lualib ${cservice} luaservice  lua-lpeg lua-sproto lua-md5)

add_dependencies(skynet examples)
