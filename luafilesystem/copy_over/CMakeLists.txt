CMAKE_MINIMUM_REQUIRED(VERSION 3.22)

PROJECT("luafilesystem")

ENABLE_TESTING()
FIND_PACKAGE(Python3 REQUIRED COMPONENTS Interpreter)
SET(PYTHON_EXECUTABLE "${Python3_EXECUTABLE}")

FIND_PACKAGE(lua5.4 CONFIG REQUIRED)
GET_TARGET_PROPERTY(LUA_INCLUDE_DIR org.lua.lua::lua5.4 INTERFACE_INCLUDE_DIRECTORIES)

# Build the Lua module.
ADD_LIBRARY(TARGET_luafilesystem MODULE
            src/lfs.c)
TARGET_INCLUDE_DIRECTORIES(TARGET_luafilesystem
                           PRIVATE ${LUA_INCLUDE_DIR})
IF((${CMAKE_SYSTEM_NAME} STREQUAL "Windows") AND (${CMAKE_COMPILER_IS_GNUCC}))
	TARGET_LINK_LIBRARIES(TARGET_luafilesystem lua5.4)
ENDIF((${CMAKE_SYSTEM_NAME} STREQUAL "Windows") AND (${CMAKE_COMPILER_IS_GNUCC}))

# Set the name of the output file to "lfs".
SET_TARGET_PROPERTIES(TARGET_luafilesystem PROPERTIES PREFIX "" OUTPUT_NAME "lfs")

# On mingw link all compiler libraries static.
IF((${CMAKE_SYSTEM_NAME} STREQUAL "Windows") AND (${CMAKE_COMPILER_IS_GNUCC}))
	SET_PROPERTY(TARGET TARGET_luafilesystem PROPERTY LINK_FLAGS "-static -static-libgcc -static-libstdc++")
ENDIF((${CMAKE_SYSTEM_NAME} STREQUAL "Windows") AND (${CMAKE_COMPILER_IS_GNUCC}))

# Install the lua module.
INSTALL(TARGETS TARGET_luafilesystem
        EXPORT EXPORT_package
        LIBRARY DESTINATION lib
        ARCHIVE DESTINATION lib
        RUNTIME DESTINATION bin
        INCLUDES DESTINATION include)


# Add tests for this module.
IF((${CMAKE_SYSTEM_NAME} STREQUAL "Windows") AND (${CMAKE_COMPILER_IS_GNUCC}))
	# Here are the MinGW specific tests.
	ADD_TEST(NAME luafilesystem_MinGW_DLL_dependencies
	         COMMAND "${PYTHON_EXECUTABLE}" ${TEST_SCRIPT_FOLDER}/mingw_dll_dependencies.py -u lua5.1 -u lua5.2 -u lua5.3 -u lua5.4 $<TARGET_FILE:TARGET_luafilesystem>)
ENDIF((${CMAKE_SYSTEM_NAME} STREQUAL "Windows") AND (${CMAKE_COMPILER_IS_GNUCC}))
