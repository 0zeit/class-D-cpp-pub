set(PJNAME "ExampleName")

cmake_minimum_required(VERSION 3.28)
project(${PJNAME} VERSION 0.1.0 LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

option(${PROJECT_NAME}_BUILD_EXAMPLE "Build example projects" ${PROJECT_IS_TOP_LEVEL})
option(${PROJECT_NAME}_BUILD_TEST "Build tests" OFF)
option(${PROJECT_NAME}_BUILD_DOCS "Build documentation" OFF)
option(${PROJECT_NAME}_BUILD_SHARED "Build as shared library" OFF)

include(FetchContent)
include(GNUInstallDirs)
include(CMakePackageConfigHelpers)

file(GLOB_RECURSE SOURCES "src/*.cpp")
file(GLOB_RECURSE HEADERS "include/*.hpp" "include/*.h")

add_executable(${PROJECT_NAME} ${SOURCES} ${HEADERS})

target_include_directories(${PROJECT_NAME}
    PUBLIC
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
        $<INSTALL_INTERFACE:include>
    PRIVATE
        ${CMAKE_CURRENT_SOURCE_DIR}/src
)

target_compile_features(${PROJECT_NAME} PUBLIC cxx_std_20)

FetchContent_Declare(
    SFML
    GIT_REPOSITORY
    https://github.com/SFML/SFML.git
    GIT_BRANCH master
    GIT_SHALLOW ON
    EXCLUDE_FROM_ALL SYSTEM
)
FetchContent_MakeAvailable(SFML)

target_link_libraries(
    ${PROJECT_NAME}
    PUBLIC
    sfml-system sfml-window sfml-graphics sfml-audio sfml-network
)

if(MSVC)
    target_compile_options(${PROJECT_NAME} PRIVATE /W4)
    target_compile_definitions(${PROJECT_NAME} PRIVATE NOMINMAX WIN32_LEAN_AND_MEAN)
else()
    target_compile_options(${PROJECT_NAME} PRIVATE -Wall -Wextra -Wpedantic -Werror)
endif()

file(
    COPY ${CMAKE_CURRENT_SOURCE_DIR}/res
    DESTINATION ${CMAKE_CURRENT_BINARY_DIR}
)

install(
    DIRECTORY res/
    DESTINATION ${CMAKE_INSTALL_DATADIR}/${PROJECT_NAME}
)

install(TARGETS ${PROJECT_NAME}
    EXPORT ${PROJECT_NAME}Targets
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
    ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
)

install(DIRECTORY include/ DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})

write_basic_package_version_file(
    "${PROJECT_NAME}ConfigVersion.cmake"
    VERSION ${PROJECT_VERSION}
    COMPATIBILITY SameMajorVersion
)

configure_package_config_file(
    "${CMAKE_CURRENT_SOURCE_DIR}/cmake/${PROJECT_NAME}Config.cmake.in"
    "${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}Config.cmake"
    INSTALL_DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/${PROJECT_NAME}
)

install(FILES
    "${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}Config.cmake"
    "${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}ConfigVersion.cmake"
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/${PROJECT_NAME}
)

install(EXPORT ${PROJECT_NAME}Targets
    FILE ${PROJECT_NAME}Targets.cmake
    NAMESPACE ${PROJECT_NAME}::
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/${PROJECT_NAME}
)