cmake_minimum_required(VERSION 3.14)

project(uWebsockets)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED True)

set(USKT_NAME uSockets)
////

option(WITH_BORINGSSL "Enable BoringSSL support" OFF)
option(WITH_OPENSSL "Enable OpenSSL support" ON)
option(WITH_WOLFSSL "Enable WolfSSL support" OFF)
option(WITH_IO_URING "Enable io_uring support" OFF)
option(WITH_LIBUV "Enable libuv support" OFF)
option(WITH_ASIO "Enable ASIO support" OFF)
option(WITH_GCD "Enable GCD support" OFF)
option(WITH_ASAN "Enable AddressSanitizer" OFF)
option(WITH_QUIC "Enable QUIC support" OFF)

if(WITH_LIBUV)
    find_package(LibUV REQUIRED)
endif()

if(WITH_OPENSSL OR WITH_BORINGSSL)
    find_package(OpenSSL REQUIRED)

elseif(WITH_WOLFSSL)
    find_package(WolfSSL REQUIRED)
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -DLIBUS_USE_WOLFSSL")

else()

    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -DLIBUS_NO_SSL")
endif()

file(GLOB_RECURSE USKT_SOURCES
    uSockets/src/*.c
    uSockets/src/eventing/*.c
    uSockets/src/crypto/*.c
    uSockets/src/io_uring/*.c
)
if(WITH_OPENSSL OR WITH_BORINGSSL)
    file(GLOB_RECURSE CRYPTO_SOURCES uSockets/src/crypto/*.cpp)
endif()

if(WITH_ASIO)
    list(APPEND USKT_SOURCES uSockets/src/eventing/asio.cpp)
endif()

add_library(${USKT_NAME} STATIC ${USKT_SOURCES} ${CRYPTO_SOURCES})

target_include_directories(${USKT_NAME} PUBLIC uSockets/src)

if(WITH_LIBUV)
    target_compile_definitions(${USKT_NAME} PUBLIC LIBUS_USE_LIBUV)
    target_link_libraries(${USKT_NAME} PUBLIC libuv::uv)
endif()

if(WITH_ASIO)
    target_compile_definitions(${USKT_NAME} PUBLIC LIBUS_USE_ASIO)
endif()

if(WITH_IO_URING)
    target_compile_definitions(${USKT_NAME} PUBLIC LIBUS_USE_IO_URING)
    target_link_libraries(${USKT_NAME} PUBLIC /usr/lib/liburing.a)
endif()

if(WITH_OPENSSL)
    target_link_libraries(${USKT_NAME} PUBLIC OpenSSL::SSL OpenSSL::Crypto)
    target_compile_definitions(${USKT_NAME} PRIVATE LIBUS_USE_OPENSSL)
endif()
///

find_package(ZLIB)

file(GLOB_RECURSE UWS_INC "src/*.h")

add_library(${PROJECT_NAME} INTERFACE ${UWS_INC})

target_link_libraries(${PROJECT_NAME} INTERFACE uSockets)
target_include_directories(${PROJECT_NAME} INTERFACE src)

if(ZLIB_FOUND)
    target_link_libraries(${PROJECT_NAME} INTERFACE ZLIB::ZLIB)
else()
    target_compile_definitions(${PROJECT_NAME} INTERFACE UWS_NO_ZLIB)
endif()

