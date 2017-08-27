#Change this if you need to target a specific CMake version
cmake_minimum_required(VERSION 3.4)
include("cget/cget/cget.cmake")

# Enable debug symbols by default
# must be done before project() statement
if(NOT CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE Debug CACHE STRING "Choose the type of build (Debug or Release)" FORCE)
endif()
# (you can also set it on the command line: -D CMAKE_BUILD_TYPE=Release)

project(Game)

file(GLOB_RECURSE Game_SOURCES "src/*.cpp")
file(GLOB_RECURSE Game_HEADERS "src/*.h")

set (Game_INCLUDE_DIRS "")
foreach (_headerFile ${Game_HEADERS})
    get_filename_component(_dir ${_headerFile} PATH)
    list (APPEND Game_INCLUDE_DIRS ${_dir})
endforeach()
list(REMOVE_DUPLICATES Game_INCLUDE_DIRS)

add_subdirectory(box2d)


# Set version information in a config.h file
set(Game_VERSION_MAJOR 1)
set(Game_VERSION_MINOR 0)
configure_file(
  "${PROJECT_SOURCE_DIR}/config.h.in"
  "${PROJECT_BINARY_DIR}/config.h"
  )
include_directories("${PROJECT_BINARY_DIR}")

# Define sources and executable
set(GAME_PROJECT_NAME "_Result")
set(GAME_EXECUTABLE_NAME "pong")
# set 
set_property(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR} PROPERTY VS_STARTUP_PROJECT ${GAME_PROJECT_NAME})
include_directories(${Game_INCLUDE_DIRS} "${PROJECT_SOURCE_DIR}/box2d/include")
add_executable(${GAME_PROJECT_NAME} ${Game_SOURCES})
set_target_properties(${GAME_PROJECT_NAME} PROPERTIES OUTPUT_NAME ${GAME_EXECUTABLE_NAME})

target_link_libraries(${GAME_PROJECT_NAME} box2d)

# Detect and add SFML
set(CMAKE_MODULE_PATH "${CMAKE_SOURCE_DIR}/cmake_modules" ${CMAKE_MODULE_PATH})
#Find any version 2.X of SFML
#See the FindSFML.cmake file for additional details and instructions
set(SFML_ROOT "${CMAKE_SOURCE_DIR}/cget")
find_package(SFML 2 REQUIRED network audio graphics window system)
if(SFML_FOUND)
  include_directories(${SFML_INCLUDE_DIR})
  target_link_libraries(${GAME_PROJECT_NAME} ${SFML_LIBRARIES} ${SFML_DEPENDENCIES})
endif()

# COPY SFML DLLs to folder with exe
add_custom_command(TARGET _Result POST_BUILD
COMMAND ${CMAKE_COMMAND} -E copy_directory
    "${PROJECT_SOURCE_DIR}/cget/bin"
    $<TARGET_FILE_DIR:_Result>)

#copy resources to CWD
add_custom_command(TARGET _Result POST_BUILD
    COMMAND ${CMAKE_COMMAND} -E copy_directory
        "${PROJECT_SOURCE_DIR}/resources"
        "${PROJECT_BINARY_DIR}/resources")


# Install target
install(TARGETS ${GAME_PROJECT_NAME} DESTINATION .)
install(DIRECTORY "${PROJECT_SOURCE_DIR}/cget/bin/" DESTINATION .)
install(DIRECTORY "${CMAKE_SOURCE_DIR}/resources/" DESTINATION resources)

# CPack packaging
include(InstallRequiredSystemLibraries)
set(CPACK_RESOURCE_FILE_LICENSE "${CMAKE_SOURCE_DIR}/LICENSE.md")
set(CPACK_RESOURCE_FILE_README "${CMAKE_SOURCE_DIR}/README.md")
set(CPACK_PACKAGE_DESCRIPTION_FILE "${CMAKE_SOURCE_DIR}/README.md")
set(CPACK_RESOURCE_FILE_WELCOME "${CMAKE_SOURCE_DIR}/WELCOME.md")

set(CPACK_PACKAGE_INSTALL_DIRECTORY "PONG GAME")
set(CPACK_PACKAGE_NAME "PONG")
set(CPACK_PACKAGE_VENDOR "soltys")
set(CPACK_PACKAGE_CONTACT "pawel@soltysiak.it")

set(CPACK_PACKAGE_VERSION_MAJOR "${Game_VERSION_MAJOR}")
set(CPACK_PACKAGE_VERSION_MINOR "${Game_VERSION_MINOR}")
include(CPack)