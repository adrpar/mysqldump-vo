cmake_minimum_required (VERSION 2.6)
project (mysqldump_vo)

###########################################################
#### uncomment this, if you want to specify own mysql path
###########################################################
#set(MYSQL_PATH "/opt/mysql")
set(MYSQL_SOURCES_PATH "/usr/src/mysql/mysql-5.5/")

if(MYSQL_PATH)
    set(MYSQL_CONFIG "${MYSQL_PATH}/bin/mysql_config")
else()
    find_file(MYSQL_CONFIG mysql_config)
endif()

execute_process(COMMAND ${MYSQL_CONFIG} --include OUTPUT_VARIABLE MYSQL_INCLUDE_DIR)
execute_process(COMMAND ${MYSQL_CONFIG} --libs OUTPUT_VARIABLE MYSQL_LIBRARIES)
execute_process(COMMAND ${MYSQL_CONFIG} --plugindir OUTPUT_VARIABLE MYSQL_PLUGIN_DIR)

STRING(REGEX REPLACE "\n" "" MYSQL_INCLUDE_DIR ${MYSQL_INCLUDE_DIR})
STRING(REGEX REPLACE "\n" "" MYSQL_LIBRARIES ${MYSQL_LIBRARIES})
STRING(REGEX REPLACE "\n" "" MYSQL_PLUGIN_DIR ${MYSQL_PLUGIN_DIR})

set(CMAKE_BUILD_TYPE Debug)

add_definitions(-DDBUG_OFF)

#add_definitions(-D__VALIDATE_DEBUG__)
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${MYSQL_INCLUDE_DIR} ${MYSQL_LIBRARIES} -fPIC -fno-exceptions -fno-rtti")
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${MYSQL_INCLUDE_DIR} ${MYSQL_LIBRARIES} -fPIC -fno-exceptions")

include_directories ("${PROJECT_SOURCE_DIR}" "${MYSQL_SOURCES_PATH}/include" "${MYSQL_SOURCES_PATH}/client" "${MYSQL_SOURCES_PATH}")

file(GLOB FILES_SRC "${PROJECT_SOURCE_DIR}/src/*.h" "${PROJECT_SOURCE_DIR}/src/*.cpp" "${PROJECT_SOURCE_DIR}/src/*.cc" "${PROJECT_SOURCE_DIR}/src/*.c")
set(MYSQL_SRC "${MYSQL_SOURCES_PATH}/mysys/hash.c" "${MYSQL_SOURCES_PATH}/mysys/string.c" "${MYSQL_SOURCES_PATH}/libmysql/get_password.c" "${MYSQL_SOURCES_PATH}/mysys/my_delete.c" "${MYSQL_SOURCES_PATH}/mysys/mf_unixpath.c" "${MYSQL_SOURCES_PATH}/mysys/base64.c" "${MYSQL_SOURCES_PATH}/strings/llstr.c"  "${MYSQL_SOURCES_PATH}/sql-common/my_user.c")

add_executable (mysqldump_vo ${FILES_SRC}  ${MYSQL_SRC})

TARGET_LINK_LIBRARIES(mysqldump_vo mysqlclient)

SET_TARGET_PROPERTIES (mysqldump_vo PROPERTIES HAS_CXX TRUE)

INSTALL(TARGETS mysqldump_vo RUNTIME DESTINATION bin)
