cmake_minimum_required(VERSION 3.1)
project(libnbt++
    VERSION 2.5)

# supported configure options
option(NBT_BUILD_SHARED "Build shared libraries" OFF)
option(NBT_USE_ZLIB "Build additional zlib stream functionality" ON)
option(NBT_BUILD_TESTS "Build the unit tests. Requires CxxTest." ON)

if(CMAKE_PROJECT_NAME STREQUAL PROJECT_NAME AND NOT DEFINED CMAKE_CXX_STANDARD)
  set(CMAKE_CXX_STANDARD 11)
endif()

if(NOT DEFINED CMAKE_CXX_STANDARD_REQUIRED)
  set(CMAKE_CXX_STANDARD_REQUIRED ON)
endif()

# hide this from includers.
set(BUILD_SHARED_LIBS ${NBT_BUILD_SHARED})

include(GenerateExportHeader)

set(NBT_SOURCES
    src/endian_str.cpp
    src/tag.cpp
    src/tag_array.cpp
    src/tag_compound.cpp
    src/tag_list.cpp
    src/tag_string.cpp
    src/value.cpp
    src/value_initializer.cpp

    src/io/stream_reader.cpp
    src/io/stream_writer.cpp

    src/text/json_formatter.cpp)

set(NBT_SOURCES_Z
    src/io/izlibstream.cpp
    src/io/ozlibstream.cpp)

if(NBT_USE_ZLIB)
    find_package(ZLIB REQUIRED)
    list(APPEND NBT_SOURCES ${NBT_SOURCES_Z})
endif()

add_library(nbt++ ${NBT_SOURCES})
target_include_directories(nbt++ PUBLIC include ${CMAKE_CURRENT_BINARY_DIR})

if(NBT_USE_ZLIB)
    target_link_libraries(nbt++ ${ZLIB_LIBRARY})
    target_include_directories(nbt++ PUBLIC ${ZLIB_INCLUDE_DIRS})
    target_compile_definitions(nbt++ PUBLIC "-DNBT_HAVE_ZLIB")
endif()
generate_export_header(nbt++ BASE_NAME nbt)

if(${BUILD_SHARED_LIBS})
    set_target_properties(nbt++ PROPERTIES
        CXX_VISIBILITY_PRESET hidden
        VISIBILITY_INLINES_HIDDEN 1)
endif()

if(NBT_BUILD_TESTS)
    enable_testing()
    add_subdirectory(test)
endif()
