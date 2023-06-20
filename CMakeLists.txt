cmake_minimum_required(VERSION 3.8)
project(themis)

include_directories(src)
include_directories(include)

SET (OPENSSL_USE_STATIC_LIBS TRUE)
IF(MSVC)
	SET(OPENSSL_MSVC_STATIC_RT TRUE)
ENDIF()
find_package(OpenSSL REQUIRED)
IF(OPENSSL_FOUND)
	MESSAGE(STATUS "Found openssl version ${OPENSSL_VERSION}")

	add_definitions(-DCRYPTO_ENGINE_PATH=openssl)
    include_directories(${OPENSSL_INCLUDE_DIR})
	set(cryptolib crypto)
	if(WIN32)
		set(cryptolib ${LIB_EAY_RELEASE} ${SSL_EAY_RELEASE} CRYPT32.lib) 
	ENDIF()
ENDIF(OPENSSL_FOUND)

if (NO_NIST_STS)
add_definitions(-DNO_NIST_STS=1)
endif()
add_definitions(-DSOTER_EXPORT)
add_definitions(-DTHEMIS_EXPORT)

file(GLOB SOTER_SOURCE_FILES src/soter/*.c src/soter/openssl/*.c src/soter/ed25519/*)
add_library(soter ${SOTER_SOURCE_FILES})
add_library(soter_shared SHARED ${SOTER_SOURCE_FILES})
set_property(TARGET soter PROPERTY CXX_STANDARD 98)
set_property(TARGET soter_shared PROPERTY CXX_STANDARD 98)
#set_target_properties(soter_shared PROPERTIES OUTPUT_NAME soter)

target_link_libraries(soter ${cryptolib})
target_link_libraries(soter_shared ${cryptolib})

if(MSVC)
	target_link_libraries(soter wsock32 ws2_32)
    target_link_libraries(soter_shared wsock32 ws2_32)
endif()

#target_compile_definitions(soter PRIVATE "DNIST_STS_EXE_PATH=tests/soter/nist-sts")
#target_compile_options(soter PRIVATE -std=gnu99)

#target_compile_definitions(soter_shared PRIVATE "DNIST_STS_EXE_PATH=tests/soter/nist-sts")
#target_compile_options(soter_shared PRIVATE -std=gnu99)

file(GLOB_RECURSE THEMIS_SOURCE_FILES src/themis/*.c)
add_library(themis ${THEMIS_SOURCE_FILES})
add_library(themis_shared SHARED ${THEMIS_SOURCE_FILES})
#set_target_properties(themis_shared PROPERTIES OUTPUT_NAME themis)
target_link_libraries(themis soter)
target_link_libraries(themis_shared soter)

file(GLOB SOTER_TEST_SOURCE tests/soter/*.c tests/common/*.c )
file(GLOB THEMIS_TEST_SOURCE tests/themis/*.c tests/common/*.c)

add_executable(soter_test ${SOTER_TEST_SOURCE} )
target_include_directories(soter_test PRIVATE tests)
target_link_libraries(soter_test soter)

add_executable(themis_test ${THEMIS_TEST_SOURCE})
target_include_directories(themis_test PRIVATE tests)
target_link_libraries(themis_test themis)

#file(GLOB THEMISPP_SOURCE src/wrappers/themis/themispp/*.hpp)
#add_library(themispp ${THEMISPP_SOURCE})
#target_link_libraries(themispp themis)
#set_target_properties(themispp PROPERTIES LINKER_LANGUAGE CXX)

file(GLOB THEMISPP_TEST_SOURCE tests/themispp/*.hpp tests/themispp/*.cpp tests/common/*)
add_executable(themispp_test ${THEMISPP_TEST_SOURCE})
target_include_directories(themispp_test PUBLIC tests src/wrappers/themis)
target_link_libraries(themispp_test themis)
set_target_properties(themispp_test PROPERTIES LINKER_LANGUAGE CXX)
