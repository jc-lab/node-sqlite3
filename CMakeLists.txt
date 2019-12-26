cmake_minimum_required(VERSION 2.8.8 FATAL_ERROR)
cmake_policy(VERSION 2.8.8)
project(node-sqlite3 C CXX ASM)

set(NODE_GYP_MODULE_NAME node-sqlite3)

if("${CMAKE_BUILD_TYPE}" STREQUAL "Debug")
    set(TARGET_IS_DEBUG 1)
else()
    set(TARGET_IS_DEBUG 0)
endif()

set(builddir "${CMAKE_CURRENT_BINARY_DIR}")

set(CMAKE_C_OUTPUT_EXTENSION_REPLACE 1)
set(CMAKE_CXX_OUTPUT_EXTENSION_REPLACE 1)

set(CMAKE_NINJA_FORCE_RESPONSE_FILE 1)

list(APPEND sqlite3__cxx_srcs
        "${CMAKE_CURRENT_SOURCE_DIR}/src/backup.cc"
        "${CMAKE_CURRENT_SOURCE_DIR}/src/database.cc"
        "${CMAKE_CURRENT_SOURCE_DIR}/src/node_sqlite3.cc"
        "${CMAKE_CURRENT_SOURCE_DIR}/src/statement.cc"
        "${NODE_GYP_PATH}/src/win_delay_load_hook.cc")
list(APPEND sqlite3__other_srcs
        "${CMAKE_CURRENT_SOURCE_DIR}/src/async.h"
        "${CMAKE_CURRENT_SOURCE_DIR}/src/backup.h"
        "${CMAKE_CURRENT_SOURCE_DIR}/src/database.h"
        "${CMAKE_CURRENT_SOURCE_DIR}/src/gcc-preinclude.h"
        "${CMAKE_CURRENT_SOURCE_DIR}/src/macros.h"
        "${CMAKE_CURRENT_SOURCE_DIR}/src/statement.h"
        "${CMAKE_CURRENT_SOURCE_DIR}/src/threading.h")
add_library(${PROJECT_NAME} MODULE ${SQLITE3_SOURCES} ${sqlite3__cxx_srcs} ${sqlite3__other_srcs})
if(MSVC)
    set_target_properties(${PROJECT_NAME} PROPERTIES LINK_FLAGS "/DELAYLOAD:node.exe")
endif()
set_target_properties(${PROJECT_NAME} PROPERTIES EXCLUDE_FROM_ALL "FALSE")
set_source_files_properties(${sqlite3__other_srcs} PROPERTIES HEADER_FILE_ONLY "TRUE")
set_target_properties(${PROJECT_NAME} PROPERTIES LIBRARY_OUTPUT_DIRECTORY "${builddir}")
set_target_properties(${PROJECT_NAME} PROPERTIES PREFIX "")
set_target_properties(${PROJECT_NAME} PROPERTIES LIBRARY_OUTPUT_NAME "sqlite3")
set_target_properties(${PROJECT_NAME} PROPERTIES SUFFIX ".node")
set_source_files_properties(${builddir}/sqlite3.node PROPERTIES GENERATED "TRUE")
list(APPEND sqlite3__include_dirs
        "${NODE_GYP_CACHE}/include/node"
        "${NODE_GYP_CACHE}/src"
        "${NODE_GYP_CACHE}/deps/openssl/config"
        "${NODE_GYP_CACHE}/deps/openssl/openssl/include"
        "${NODE_GYP_CACHE}/deps/uv/include"
        "${NODE_GYP_CACHE}/deps/zlib"
        "${NODE_GYP_CACHE}/deps/v8/include"
        ${SQLITE3_INCLUDE_DIRS}
        ${NAN_INCLUDE_DIRS})
set_property(TARGET ${PROJECT_NAME} APPEND PROPERTY INCLUDE_DIRECTORIES ${sqlite3__include_dirs})

if(TARGET_IS_DEBUG)
    set_target_properties(${PROJECT_NAME} PROPERTIES COMPILE_DEFINITIONS "NODE_GYP_MODULE_NAME=${NODE_GYP_MODULE_NAME};USING_UV_SHARED=1;USING_V8_SHARED=1;V8_DEPRECATION_WARNINGS=1;V8_DEPRECATION_WARNINGS;V8_IMMINENT_DEPRECATION_WARNINGS;WIN32;_CRT_SECURE_NO_DEPRECATE;_CRT_NONSTDC_NO_DEPRECATE;_HAS_EXCEPTIONS=0;OPENSSL_THREADS;SQLITE_HAS_CODEC=1;SQLCIPHER_CRYPTO_OPENSSL=1;SQLITE_THREADSAFE=1;HAVE_USLEEP=1;SQLITE_ENABLE_FTS3=1;SQLITE_ENABLE_FTS4=1;SQLITE_ENABLE_FTS5=1;SQLITE_ENABLE_JSON1=1;SQLITE_ENABLE_RTREE=1;BUILDING_NODE_EXTENSION;HOST_BINARY=\"node\";DEBUG;_DEBUG;V8_ENABLE_CHECKS;")
    set_target_properties(${PROJECT_NAME} PROPERTIES COMPILE_FLAGS "-include ${CMAKE_CURRENT_SOURCE_DIR}/src/gcc-preinclude.h -g -O0 ")
else()
    set_target_properties(${PROJECT_NAME} PROPERTIES COMPILE_DEFINITIONS "NODE_GYP_MODULE_NAME=${NODE_GYP_MODULE_NAME};USING_UV_SHARED=1;USING_V8_SHARED=1;V8_DEPRECATION_WARNINGS=1;V8_DEPRECATION_WARNINGS;V8_IMMINENT_DEPRECATION_WARNINGS;WIN32;_CRT_SECURE_NO_DEPRECATE;_CRT_NONSTDC_NO_DEPRECATE;_HAS_EXCEPTIONS=0;OPENSSL_THREADS;SQLITE_HAS_CODEC=1;SQLCIPHER_CRYPTO_OPENSSL=1;SQLITE_THREADSAFE=1;HAVE_USLEEP=1;SQLITE_ENABLE_FTS3=1;SQLITE_ENABLE_FTS4=1;SQLITE_ENABLE_FTS5=1;SQLITE_ENABLE_JSON1=1;SQLITE_ENABLE_RTREE=1;BUILDING_NODE_EXTENSION;HOST_BINARY=\"node\";NDEBUG;")
    set_target_properties(${PROJECT_NAME} PROPERTIES COMPILE_FLAGS "-include ${CMAKE_CURRENT_SOURCE_DIR}/src/gcc-preinclude.h -O3 ")
endif()

target_link_libraries(${PROJECT_NAME}
        kernel32.lib
        user32.lib
        gdi32.lib
        winspool.lib
        comdlg32.lib
        advapi32.lib
        shell32.lib
        ole32.lib
        oleaut32.lib
        uuid.lib
        odbc32.lib
        DelayImp.lib
        ${NODE_LIB}
        ${SQLITE3_LIBRARIES}
        )

if(SQLCIPHER_CRYPTO_OPENSSL)
    find_package(OpenSSL REQUIRED)
    target_link_libraries(${PROJECT_NAME}
        OpenSSL::Crypto
    )
endif()