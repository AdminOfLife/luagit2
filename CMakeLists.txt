#
# Lua bindings for libgit2
#
cmake_minimum_required(VERSION 2.8)

project(lua-git2 C)

set(BUILD_SHARED_LIBS TRUE)

set(CMAKE_MODULE_PATH ${CMAKE_SOURCE_DIR}/cmake)

set(INSTALL_CMOD share/lua/cmod CACHE PATH "Directory to install Lua binary modules (configure lua via LUA_CPATH)")
set(LUA_NATIVE_OBJECTS_PATH ../LuaNativeObjects CACHE PATH
				"Directory to LuaNativeObjects bindings generator.")

set(COMMON_CFLAGS)
set(COMMON_LDFLAGS)
set(COMMON_LIBS)

## Lua 5.1.x
include(FindLua51)
if(!${LUA51_FOUND})
    message(FATAL_ERROR "The FindLua51 module could not find lua :-(")
endif()
set(COMMON_LIBS "${COMMON_LIBS};${LUA_LIBRARIES}")

## LibGit2
include(FindPkgConfig)
pkg_search_module(GIT2 REQUIRED libgit2)
set(COMMON_CFLAGS "${COMMON_CFLAGS} ${GIT2_CFLAGS}")
set(COMMON_LDFLAGS "${COMMON_LDFLAGS} ${GIT2_LDFLAGS}")
set(COMMON_LIBS "${COMMON_LIBS};${GIT2_LIBRARIES}")

## LuaNativeObjects
include(LuaNativeObjects)

include_directories(${CMAKE_CURRENT_SOURCE_DIR}
                    ${LUA_INCLUDE_DIR})

## LuaGit2
set(LUA_GIT2_SRC
	git2.nobj.lua
)

# Generate Lua bindings.
GenLuaNativeObjects(LUA_GIT2_SRC)

add_library(lua-git2 MODULE ${LUA_GIT2_SRC})
target_link_libraries(lua-git2 ${COMMON_LIBS})
set_target_properties(lua-git2 PROPERTIES PREFIX "")
set_target_properties(lua-git2 PROPERTIES COMPILE_FLAGS "${COMMON_CFLAGS}")
set_target_properties(lua-git2 PROPERTIES OUTPUT_NAME git2)

install(TARGETS lua-git2
        DESTINATION "${INSTALL_CMOD}")
