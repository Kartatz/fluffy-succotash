cmake_minimum_required(VERSION 3.13)

project(
	SparkleC
	VERSION 0.4
	DESCRIPTION "Tiny CLI tool to download courses from Hotmart"
	HOMEPAGE_URL "https://github.com/Kartatz/SparkleC"
	LANGUAGES C
)

set(ENABLE_COMMOFFCRYPTO OFF)
set(ENABLE_GNUTLS OFF)
set(ENABLE_MBEDTLS OFF)
set(ENABLE_OPENSSL OFF)
set(ENABLE_WINDOWS_CRYPTO OFF)
set(LIBZIP_DO_INSTALL OFF)
set(SHARED_LIB_VERSIONNING OFF)
set(BUILD_EXAMPLES OFF)
set(BUILD_DOC OFF)
set(ENABLE_FDOPEN OFF)
set(ENABLE_BZIP2 OFF)
set(ENABLE_LZMA OFF)
set(ENABLE_ZSTD OFF)

set(CMAKE_POLICY_DEFAULT_CMP0077 NEW)
set(CMAKE_POLICY_DEFAULT_CMP0048 NEW)

add_subdirectory(submodules/zlib EXCLUDE_FROM_ALL)

set(ZLIB_INCLUDE_DIRS "${CMAKE_SOURCE_DIR}/submodules/zlib")
set(ZLIB_LIBRARY "${CMAKE_SOURCE_DIR}/submodules/zlib/libz${CMAKE_SHARED_LIBRARY_SUFFIX}")

add_subdirectory(submodules/libzip EXCLUDE_FROM_ALL)

add_custom_command(
	OUTPUT libz
	PRE_BUILD
	COMMAND true
    DEPENDS zlib
)

add_custom_target(
	ensure_zlib
	ALL
    DEPENDS libz
)

set_target_properties(
	zip
	PROPERTIES
	IMPORTED_LOCATION ${ZLIB_LIBRARY}
)

add_executable(
	sparklec
	src/main.c
)

if (APPLE)
	foreach(property BUILD_RPATH INSTALL_RPATH)
		set_target_properties(
			sparklec
			PROPERTIES
			${property} "@loader_path/../lib:@loader_path"
		)
	endforeach()
elseif (UNIX)
	foreach(property BUILD_RPATH INSTALL_RPATH)
		set_target_properties(
			sparklec
			PROPERTIES
			${property} "$ORIGIN/../lib:$ORIGIN"
		)
	endforeach()
endif()

if (APPLE)
	foreach(property BUILD_RPATH INSTALL_RPATH)
		set_target_properties(
			zip
			PROPERTIES
			${property} "@loader_path"
		)
	endforeach()
elseif (UNIX)
	foreach(property BUILD_RPATH INSTALL_RPATH)
		set_target_properties(
			zip
			PROPERTIES
			${property} "$ORIGIN"
		)
	endforeach()
endif()

foreach(property RUNTIME_OUTPUT_DIRECTORY LIBRARY_OUTPUT_DIRECTORY)
	foreach(target zlib zip)
		set_target_properties(
			${target}
			PROPERTIES
			${property} $<TARGET_FILE_DIR:sparklec>
		)
	endforeach()
endforeach()

foreach(target zlib zip)
	set_target_properties(
		${target}
		PROPERTIES
		NO_SONAME 1
	)
endforeach()

if (CMAKE_C_COMPILER_ID STREQUAL "GNU")
	target_link_options(
		sparklec
		BEFORE
		PRIVATE
		-Wl,-rpath-link,$<TARGET_FILE_DIR:sparklec>
	)
endif()

target_link_libraries(
	sparklec
	zip
)

foreach(target sparklec zlib zip)
	install(
		TARGETS ${target}
		RUNTIME DESTINATION bin
		LIBRARY DESTINATION lib NAMELINK_SKIP
	)
endforeach()
