cmake_minimum_required(VERSION 3.11.4 FATAL_ERROR)
project(baton LANGUAGES C)

find_package(OpenSSL REQUIRED)

set(IRODS_CLIENT_C_INCLUDES_ROOT /home/kory/dev/prog/cpp/irods_client_library_c)
set(IRODS_CLIENT_C_LIBS_ROOT /home/kory/dev/prog/cpp/build_irods_client_library_c)

#
# Compile baton shared library.
#

set(BATON_COMMON baton)

add_library(${BATON_COMMON} SHARED ${CMAKE_SOURCE_DIR}/src/baton.c
                                   ${CMAKE_SOURCE_DIR}/src/compat_checksum.c
                                   ${CMAKE_SOURCE_DIR}/src/error.c
                                   ${CMAKE_SOURCE_DIR}/src/json.c
                                   ${CMAKE_SOURCE_DIR}/src/json_query.c
                                   ${CMAKE_SOURCE_DIR}/src/list.c
                                   ${CMAKE_SOURCE_DIR}/src/log.c
                                   ${CMAKE_SOURCE_DIR}/src/operations.c
                                   ${CMAKE_SOURCE_DIR}/src/query.c
                                   ${CMAKE_SOURCE_DIR}/src/read.c
                                   ${CMAKE_SOURCE_DIR}/src/utilities.c
                                   ${CMAKE_SOURCE_DIR}/src/write.c)

target_compile_definitions(${BATON_COMMON}
                           PRIVATE
                           RODS_REL_VERSION="rods4.3.0"
                           RODS_API_VERSION="d"
                           IRODS_VERSION_MAJOR=4
                           IRODS_VERSION_MINOR=3
                           IRODS_VERSION_PATCHLEVEL=0
                           ACCESS_NULL="null"
                           ACCESS_READ_OBJECT="read object"
                           ACCESS_MODIFY_OBJECT="modify object"
                           ACCESS_OWN="own")

target_compile_options(${BATON_COMMON} PRIVATE -Wall -Wextra)

target_include_directories(${BATON_COMMON}
                           PRIVATE
                           ${CMAKE_SOURCE_DIR}/src
                           ${IRODS_CLIENT_C_INCLUDES_ROOT}/include/api
                           ${IRODS_CLIENT_C_INCLUDES_ROOT}/include/core
                           ${IRODS_CLIENT_C_INCLUDES_ROOT}/include/hasher
                           ${OPENSSL_INCLUDE_DIR}
                           /opt/irods-externals/jansson2.7-0/include)

target_link_libraries(${BATON_COMMON}
                      PRIVATE
                      ${IRODS_CLIENT_C_LIBS_ROOT}/libirods_client_c.so
                      /opt/irods-externals/jansson2.7-0/lib/libjansson.so)

#
# Compile baton commands.
#

set(EXECUTABLES baton-chmod
                baton-do
                baton-get
                baton-list
                baton-metamod
                baton-metaquery
                baton-put
                baton-specificquery)

foreach(EXECUTABLE ${EXECUTABLES})
    add_executable(${EXECUTABLE} ${CMAKE_SOURCE_DIR}/src/${EXECUTABLE}.c)

    target_compile_definitions(${EXECUTABLE}
                               PRIVATE
                               RODS_REL_VERSION="rods4.3.0"
                               RODS_API_VERSION="d"
                               IRODS_VERSION_MAJOR=4
                               IRODS_VERSION_MINOR=3
                               IRODS_VERSION_PATCHLEVEL=0
                               ACCESS_NULL="null"
                               ACCESS_READ_OBJECT="read object"
                               ACCESS_MODIFY_OBJECT="modify object"
                               ACCESS_OWN="own")

    target_compile_options(${EXECUTABLE} PRIVATE -Wall)

    target_include_directories(${EXECUTABLE}
                               PRIVATE
                               ${CMAKE_SOURCE_DIR}/src
                               ${IRODS_CLIENT_C_INCLUDES_ROOT}/include/api
                               ${IRODS_CLIENT_C_INCLUDES_ROOT}/include/core
                               ${IRODS_CLIENT_C_INCLUDES_ROOT}/include/hasher
                               ${OPENSSL_INCLUDE_DIR}
                               /opt/irods-externals/jansson2.7-0/include)

    target_link_libraries(${EXECUTABLE}
                          PRIVATE
                          ${BATON_COMMON}
                          ${IRODS_CLIENT_C_LIBS_ROOT}/libirods_client_c.so
                          /opt/irods-externals/jansson2.7-0/lib/libjansson.so)
endforeach()

