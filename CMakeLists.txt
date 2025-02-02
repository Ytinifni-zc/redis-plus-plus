cmake_minimum_required(VERSION 3.0.0)

project(redis++)

set(CMAKE_CXX_FLAGS "-std=c++11 -Wall -W -Werror -fPIC")
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)

# TODO: TLS
set(_THIS_DIR ${PROJECT_SOURCE_DIR}) # PROJECT_SOURCE_DIR is changed later
list(APPEND REDIS_PLUS_PLUS_HEADER
        ${_THIS_DIR}/src
        ${_THIS_DIR}/src/sw/redis++/no_tls
        )

# hiredis dependency
option(REDIS_PLUS_PLUS_BUILD_HIREDIS_FROM_SUBMODULE "Build hiredis dependency from submodule instead of system" ON)

message(STATUS "REDIS_PLUS_PLUS_BUILD_HIREDIS_FROM_SUBMODULE Headers: ${REDIS_PLUS_PLUS_BUILD_HIREDIS_FROM_SUBMODULE}")
if (REDIS_PLUS_PLUS_BUILD_HIREDIS_FROM_SUBMODULE)
    set(HIREDIS_HEADER ${CMAKE_CURRENT_SOURCE_DIR})
    message(STATUS "hiRedis Headers: ${HIREDIS_HEADER}")
    add_subdirectory(hiredis)
else ()
    find_path(HIREDIS_HEADER hiredis)
    find_library(HIREDIS_LIB hiredis)
endif ()
message("HIREDIS_HEADER ${HIREDIS_HEADER}")

# redis-plus-plus
set(PROJECT_SOURCE_DIR ${PROJECT_SOURCE_DIR}/src/sw/redis++)
file(GLOB PROJECT_SOURCE_FILES "${PROJECT_SOURCE_DIR}/*.cpp")

set(STATIC_LIB redis++_static)
set(SHARED_LIB redis++)

add_library(${STATIC_LIB} STATIC ${PROJECT_SOURCE_FILES})
add_library(${SHARED_LIB} SHARED ${PROJECT_SOURCE_FILES})

target_include_directories(${STATIC_LIB} PUBLIC ${HIREDIS_HEADER} ${REDIS_PLUS_PLUS_HEADER})
target_include_directories(${SHARED_LIB} PUBLIC ${HIREDIS_HEADER} ${REDIS_PLUS_PLUS_HEADER})

if (REDIS_PLUS_PLUS_BUILD_HIREDIS_FROM_SUBMODULE)
    target_link_libraries(${SHARED_LIB} hiredis)
else ()
    target_link_libraries(${SHARED_LIB} ${HIREDIS_LIB})
endif ()

set_target_properties(${STATIC_LIB} PROPERTIES OUTPUT_NAME ${PROJECT_NAME})
set_target_properties(${SHARED_LIB} PROPERTIES OUTPUT_NAME ${PROJECT_NAME})

set_target_properties(${STATIC_LIB} PROPERTIES CLEAN_DIRECT_OUTPUT 1)
set_target_properties(${SHARED_LIB} PROPERTIES CLEAN_DIRECT_OUTPUT 1)

option(REDIS_PLUS_PLUS_BUILD_TESTS "Build redis-plus-plus tests" OFF)

if (REDIS_PLUS_PLUS_BUILD_TESTS)
    add_subdirectory(test)
endif ()

# Install static lib.
install(TARGETS ${STATIC_LIB}
        ARCHIVE DESTINATION lib)

# Install shared lib.
install(TARGETS ${SHARED_LIB}
        LIBRARY DESTINATION lib)

#Install headers.
set(HEADER_PATH "sw/redis++")
file(GLOB HEADERS "${PROJECT_SOURCE_DIR}/*.h*")
install(FILES ${HEADERS} DESTINATION ${CMAKE_INSTALL_PREFIX}/include/${HEADER_PATH})