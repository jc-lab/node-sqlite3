cmake_minimum_required (VERSION 2.8)
project(node-sqlite3)

set(SRC_PATH ${CMAKE_CURRENT_SOURCE_DIR}/src)

if(MSVC)
	set(SQLITE_SRC_DIR "${CMAKE_SOURCE_DIR}/windows/sqlcipher")
	set(OPENSSL_INC_DIR "${CMAKE_SOURCE_DIR}/windows/openssl-prebuilt-winx86/include")
	set(OPENSSL_LIB_DIR "${CMAKE_SOURCE_DIR}/windows/openssl-prebuilt-winx86/msvc_lib/${MSVC_TOOLSET}/lib32")
	set(LIBEAY_NAME "libeay32.lib")
	if(TARGET_IS_DEBUG)
		set(BINDING_OUT_DIR "${CMAKE_SOURCE_DIR}/windows/install-base-dir-debug/bin32")
	else()
		set(BINDING_OUT_DIR "${CMAKE_SOURCE_DIR}/windows/install-base-dir-release/bin32")
	endif()
else()
	set(SQLITE_SRC_DIR "${CMAKE_SOURCE_DIR}/linux/sqlcipher")
endif()


file(GLOB_RECURSE SRC_FILES
	${SRC_PATH}/*.c
	${SRC_PATH}/*.h
	${SRC_PATH}/*.cc
	${SQLITE_SRC_DIR}/src/sqlite3.c
)

set(BINDING_GYP_SRC_LIST "")
set(BINDING_GCC_PREINCLUDE_FILE ${CMAKE_CURRENT_SOURCE_DIR}/src/gcc-preinclude.h)

string (REGEX REPLACE "([^;]+)" "\"\\1\"" BINDING_GYP_SRC_LIST "${SRC_FILES}")
string (REPLACE ";" "," BINDING_GYP_SRC_LIST "${BINDING_GYP_SRC_LIST}")

configure_file(${CMAKE_CURRENT_SOURCE_DIR}/binding.gyp.in ${CMAKE_CURRENT_BINARY_DIR}/binding.gyp @ONLY)

if(MSVC)
	add_custom_command(
		OUTPUT node_modules.md5
		COMMAND ${CMAKE_COMMAND} -E copy ${CMAKE_CURRENT_SOURCE_DIR}/package.json ${CMAKE_CURRENT_SOURCE_DIR}/package-lock.json ${CMAKE_CURRENT_BINARY_DIR}
		COMMAND call npm install --ignore-scripts
		COMMAND ${CMAKE_COMMAND} -E env "PATH=${BUILD_TOOLS_BIN};$ENV{PATH}" npm.cmd run node_modules_postinstall
		WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}
		DEPENDS ${CMAKE_CURRENT_BINARY_DIR}/bindings.js ${CMAKE_CURRENT_BINARY_DIR}/package.json
	)
else()
	add_custom_command(
		OUTPUT node_modules.md5
		COMMAND ${CMAKE_COMMAND} -E copy ${CMAKE_CURRENT_SOURCE_DIR}/package.json ${CMAKE_CURRENT_SOURCE_DIR}/package-lock.json ${CMAKE_CURRENT_BINARY_DIR}
		COMMAND npm install --ignore-scripts
		COMMAND ${CMAKE_COMMAND} -E env "PATH=${BUILD_TOOLS_BIN}:$ENV{PATH}" npm run node_modules_postinstall
		WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}
		DEPENDS ${CMAKE_CURRENT_BINARY_DIR}/bindings.js ${CMAKE_CURRENT_BINARY_DIR}/package.json
	)
endif()

add_custom_target(${PROJECT_NAME}-init
	DEPENDS node_modules.md5
)

if(TARGET_IS_DEBUG)
	add_custom_command(
		OUTPUT test.node
		COMMAND call npm run node-gyp -- --arch=ia32 --target=${TARGET_ELECTRON_VERSION} --dist-url=https://electronjs.org/headers -j 4 --thin=yes configure
		COMMAND call npm run node-gyp -- -j 4 --debug --arch=ia32 build
		WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}
		DEPENDS ${PROJECT_NAME}-init ${SRC_FILES}
	)
else()
	add_custom_command(
		OUTPUT test.node
		COMMAND npm run node-gyp -- --target=${TARGET_ELECTRON_VERSION} --dist-url=https://electronjs.org/headers -j 4 --thin=yes configure
		COMMAND npm run node-gyp -- -j 4 --release --arch=ia32 build
		WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}
		DEPENDS ${PROJECT_NAME}-init ${SRC_FILES}
	)
endif()

add_custom_target(${PROJECT_NAME}
	SOURCES  ${SRC_FILES}
	DEPENDS test.node
)

set_property(TARGET ${PROJECT_NAME} PROPERTY ZMANB_TARGET_FILE ${ZMANB_TARGET_FILE})

#file(MAKE_DIRECTORY ${FRONT_TESTING_PATH})
#configure_file(${CMAKE_CURRENT_SOURCE_DIR}/front/testing-env.js.in ${FRONT_TESTING_PATH}/testing-env.js)
