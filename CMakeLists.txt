cmake_minimum_required(VERSION 3.2)

project(craft)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

string(COMPARE EQUAL "${CMAKE_BUILD_TYPE}" "RelWithDebInfo" ENABLE_DEBUGGING)
string(COMPARE EQUAL "${CMAKE_BUILD_TYPE}" "Debug" ENABLE_DEBUGGING2)

if(ENABLE_DEBUGGING OR ENABLE_DEBUGGING2)
    message(INFO ": Debugging enabled for ${PROJECT_NAME}")
    add_compile_definitions("DEBUGGING")
endif()

file(GLOB SOURCE_FILES src/*.c)

add_executable(
    craft
    ${SOURCE_FILES}
    deps/glad/src/glad_3_0.c
    deps/lodepng/lodepng.c
    deps/noise/noise.c
    deps/sqlite/sqlite3.c
    deps/tinycthread/tinycthread.c
    deps/dearimgui/imgui.cpp
    deps/dearimgui/imgui_demo.cpp
    deps/dearimgui/imgui_draw.cpp
    deps/dearimgui/imgui_tables.cpp
    deps/dearimgui/imgui_widgets.cpp
    deps/dearimgui/backends/imgui_impl_opengl3.cpp
    deps/dearimgui/backends/imgui_impl_sdl3.cpp)

include_directories(deps/glad/include)
include_directories(deps/lodepng)
include_directories(deps/noise)
include_directories(deps/sqlite)
include_directories(deps/tinycthread)
include_directories(deps/dearimgui)

find_package(OpenGL REQUIRED)

# Find/Configure SDL3
find_package(SDL3 QUIET)
if(SDL3_FOUND)
    message(STATUS "Using SDL3 via find_package")
endif()

# Check current project for SDL3
if(NOT SDL3_FOUND AND EXISTS "${CMAKE_CURRENT_LIST_DIR}/SDL/CMakeLists.txt")
    add_subdirectory(SDL)
    message(STATUS "Using SDL3 via add_subdirectory")
    set(SDL3_FOUND TRUE)
endif()

# If all else fails, download SDL3
if(NOT SDL3_FOUND)
    include(FetchContent)
    set(SDL_SHARED TRUE CACHE BOOL "Build a SDL shared library (if available)")
    set(SDL_STATIC TRUE CACHE BOOL "Build a SDL static library (if available)")
    FetchContent_Declare(
        SDL
        GIT_REPOSITORY https://github.com/libsdl-org/SDL.git
        GIT_TAG main  # Replace this with a particular git tag or git hash
        GIT_SHALLOW TRUE
        GIT_PROGRESS TRUE
    )
    message(STATUS "Using SDL3 via FetchContent")
    FetchContent_MakeAvailable(SDL)
    set_property(DIRECTORY "${CMAKE_CURRENT_BINARY_DIR}/_deps/sdl-src" PROPERTY EXCLUDE_FROM_ALL TRUE)
endif()

if(MINGW)
    set(CMAKE_LIBRARY_PATH ${CMAKE_LIBRARY_PATH}
        "C:/Program Files/CURL/lib" "C:/Program Files (x86)/CURL/lib")
    set(CMAKE_INCLUDE_PATH ${CMAKE_INCLUDE_PATH}
        "C:/Program Files/CURL/include" "C:/Program Files (x86)/CURL/include")
endif()

# find_package(CURL REQUIRED)
# include_directories(${CURL_INCLUDE_DIR})

if(APPLE)
    target_link_libraries(craft OpenGL::GL SDL3::SDL3)
endif()

if(UNIX)
    target_link_libraries(craft dl OpenGL::GL SDL3::SDL3)
endif()

if(MINGW)
    target_link_libraries(craft ws2_32.lib OpenGL::GL SDL3::SDL3)
endif()
