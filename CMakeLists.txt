cmake_minimum_required(VERSION 2.8.3)
project(serial)

set(PACKAGE_VERSION 1.2.1)

include(CTest)

if(APPLE)
    find_library(IOKIT_LIBRARY IOKit)
    find_library(FOUNDATION_LIBRARY Foundation)
endif()

## Sources
set(serial_SRCS
    src/serial.cc
    include/serial/serial.h
    include/serial/v8stdint.h
)
if(APPLE)
    # If OSX
    list(APPEND serial_SRCS src/impl/unix.cc)
    list(APPEND serial_SRCS src/impl/list_ports/list_ports_osx.cc)
elseif(UNIX)
    # If unix
    list(APPEND serial_SRCS src/impl/unix.cc)
    list(APPEND serial_SRCS src/impl/list_ports/list_ports_linux.cc)
else()
    # If windows
    list(APPEND serial_SRCS src/impl/win.cc)
    list(APPEND serial_SRCS src/impl/list_ports/list_ports_win.cc)
endif()

## Add serial library
add_library(${PROJECT_NAME} SHARED ${serial_SRCS})
if(APPLE)
    target_link_libraries(${PROJECT_NAME} ${FOUNDATION_LIBRARY} ${IOKIT_LIBRARY})
elseif(UNIX)
    target_link_libraries(${PROJECT_NAME} rt pthread)
else()
    target_link_libraries(${PROJECT_NAME} setupapi)
endif()

## Uncomment for example
add_executable(serial_example examples/serial_example.cc)
add_dependencies(serial_example ${PROJECT_NAME})
target_link_libraries(serial_example ${PROJECT_NAME})

## Include headers
include_directories(include)

set(TARGETS_EXPORT_NAME "${PROJECT_NAME}-targets")

include(CMakePackageConfigHelpers)

## Generate CMake package version file
set(VERSION_CONFIG_FILE
    "${CMAKE_CURRENT_BINARY_DIR}/generated/${PROJECT_NAME}Config-version.cmake"
)
write_basic_package_version_file(
    "${VERSION_CONFIG_FILE}"
    VERSION "${PACKAGE_VERSION}"
    COMPATIBILITY SameMajorVersion
)

## Generate CMake configuration file
set(INCLUDE_DIRS "include")
set(LIBRARIES ${PROJECT_NAME})
set(CONFIG_FILE
    "${CMAKE_CURRENT_BINARY_DIR}/generated/${PROJECT_NAME}Config.cmake"
)
configure_package_config_file(
    "config.cmake.in"
    "${CONFIG_FILE}"
    INSTALL_DESTINATION "share/${PROJECT_NAME}/cmake"
    PATH_VARS
        INCLUDE_DIRS
        LIBRARIES
)

## Install library
install(TARGETS ${PROJECT_NAME}
    EXPORT "${TARGETS_EXPORT_NAME}"
    ARCHIVE DESTINATION "lib"
    LIBRARY DESTINATION "lib"
    RUNTIME DESTINATION "bin"
)

## Install headers
install(FILES include/serial/serial.h include/serial/v8stdint.h
    DESTINATION "include/${PROJECT_NAME}"
)

## Install export file
install(EXPORT "${TARGETS_EXPORT_NAME}"
    DESTINATION "share/${PROJECT_NAME}/cmake"
)

## Install CMake config files
install(FILES "${VERSION_CONFIG_FILE}" "${CONFIG_FILE}"
    DESTINATION "share/${PROJECT_NAME}/cmake"
)

## Install package.xml
install(FILES package.xml
  DESTINATION "share/${PROJECT_NAME}"
)

## Tests
if(BUILD_TESTING)
    add_subdirectory(tests)
endif()
