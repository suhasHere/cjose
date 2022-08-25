cmake_minimum_required(VERSION 3.12)

project(cjose
  VERSION 0.1
  LANGUAGES C
)

###
### Global Config
###
set_property(GLOBAL PROPERTY USE_FOLDERS ON)
set(PACKAGE_VERSION 0.6.1)
configure_file(${PROJECT_SOURCE_DIR}/include/cjose/version.h.in ${PROJECT_SOURCE_DIR}/include/cjose/version.h @ONLY)

# External libraries
find_package(OpenSSL 1.1 REQUIRED)
pkg_check_modules(JANSSON jansson>=2.3)
if (JANSSON_FOUND)
    set(JANSSON_PKG libjansson)
    include_directories(${JANSSON_INCLUDE_DIRS})
    link_directories(${JANSSON_LIBRARY_DIRS})
else (JANSSON_FOUND)
    message(FATAL_ERROR "libjansson >=2.3 not found")
endif (JANSSON_FOUND)

###
### Library Config
###

set(LIB_NAME "${PROJECT_NAME}")

file(GLOB_RECURSE LIB_HEADERS CONFIGURE_DEPENDS "${CMAKE_CURRENT_SOURCE_DIR}/include/*.h")
file(GLOB_RECURSE LIB_SOURCES CONFIGURE_DEPENDS "${CMAKE_CURRENT_SOURCE_DIR}/src/*.c")

add_library(${LIB_NAME} ${LIB_HEADERS} ${LIB_SOURCES})
target_link_libraries(${LIB_NAME} 
                    PUBLIC
                        OpenSSL::Crypto
                        jansson)
set_target_properties(${LIB_NAME} 
    PROPERTIES
        C_STANDARD 11
        C_STANDARD_REQUIRED YES
        C_EXTENSIONS YES)

target_include_directories(${LIB_NAME}
  PUBLIC
    $<INSTALL_INTERFACE:include>
    $<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}/include>
    $<INSTALL_INTERFACE:include/${PROJECT_NAME}-${PROJECT_VERSION}>
  PRIVATE
    ${OPENSSL_INCLUDE_DIR}
    ${JANSSON_INCLUDE_DIRS}
)

target_compile_options(${LIB_NAME} PRIVATE
    $<$<OR:$<C_COMPILER_ID:Clang>,$<C_COMPILER_ID:AppleClang>,$<C_COMPILER_ID:GNU>>: -Wpedantic -Wextra -Wall>
    $<$<C_COMPILER_ID:MSVC>: >)

target_link_directories(${LIB_NAME} PUBLIC ${JANSSON_LIBRARY_DIRS}) 
