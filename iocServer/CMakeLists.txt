cmake_minimum_required(VERSION 3.16)
project(ioc-server)

set(CMAKE_CXX_STANDARD 17)

set(INSTALL_${PROJECT_NAME} ON)

if(MSVC)
    add_compile_options(/W4 /WX)
else()
    add_compile_options(-Wall -Wextra -pedantic -Werror)
endif()

include(GNUInstallDirs)

if(DEFINED ENV{CMAKE_INSTALL_PREFIX})
    set(CMAKE_INSTALL_PREFIX $ENV{CMAKE_INSTALL_PREFIX})
endif()

if(DEFINED ENV{BOOST_ROOT})
    set(BOOST_ROOT $ENV{BOOST_ROOT})
endif()

if(DEFINED ENV{Boost_DEBUG})
    set(Boost_DEBUG $ENV{Boost_DEBUG})
endif()

if(NOT DEFINED PACKAGE_NAME)
    message(STATUS "Searching for ioc-container")
    find_package(ioc-container REQUIRED)
endif()

set(THREADS_PREFER_PTHREAD_FLAG ON)
find_package(Threads REQUIRED)

find_package(Boost REQUIRED)

add_library(${PROJECT_NAME} src/main.cpp src/construction/AppBuilder.cpp src/construction/AppBuilder.h src/Server.cpp src/Server.h src/ThreadRunner.cpp src/ThreadRunner.h)

target_include_directories(${PROJECT_NAME} PUBLIC $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
        $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}> PUBLIC ${Boost_INCLUDE_DIR})

target_link_libraries(${PROJECT_NAME} PRIVATE Threads::Threads)

if(DEFINED PACKAGE_NAME)
    target_link_libraries(${PROJECT_NAME} PUBLIC ${PACKAGE_NAME}::ioc-container)
else()
    target_link_libraries(${PROJECT_NAME} PUBLIC ioc-container::ioc-container)
endif()

if(DEFINED PACKAGE_NAME)
    message(STATUS "Exporting target to ${PACKAGE_NAME}Targets")

    add_library(${PACKAGE_NAME}::${PROJECT_NAME} ALIAS ${PROJECT_NAME})

    install(TARGETS ${PACKAGE_NAME}::${PROJECT_NAME}
            EXPORT ${PACKAGE_NAME}Targets)

    install(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/include/mabiphmo TYPE INCLUDE)
else()
    if(${INSTALL_${PROJECT_NAME}})
        message(STATUS "Installing StandAlone")

        install(TARGETS ${PROJECT_NAME}
                EXPORT ${PROJECT_NAME}Targets
                LIBRARY DESTINATION lib COMPONENT Runtime
                ARCHIVE DESTINATION lib COMPONENT Development
                RUNTIME DESTINATION bin COMPONENT Runtime
                PUBLIC_HEADER DESTINATION include COMPONENT Development
                BUNDLE DESTINATION bin COMPONENT Runtime)

        include(CMakePackageConfigHelpers)
        configure_package_config_file(
                "${PROJECT_NAME}Config.cmake.in"
                "${PROJECT_NAME}Config.cmake"
                INSTALL_DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/${PROJECT_NAME}
                PATH_VARS
                CMAKE_INSTALL_LIBDIR
        )

        write_basic_package_version_file(
                ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}ConfigVersion.cmake
                VERSION 1.0.0
                COMPATIBILITY SameMajorVersion
        )

        install(EXPORT ${PROJECT_NAME}Targets
                FILE ${PROJECT_NAME}Targets.cmake
                NAMESPACE ${PROJECT_NAME}::
                DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/${PROJECT_NAME})

        install(FILES "${PROJECT_BINARY_DIR}/${PROJECT_NAME}Config.cmake"
                "${PROJECT_BINARY_DIR}/${PROJECT_NAME}ConfigVersion.cmake"
                DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/${PROJECT_NAME})

        install(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/include/mabiphmo TYPE INCLUDE)
    else()
        message(STATUS "Not installing")
    endif()
endif()

enable_testing()

add_subdirectory(test)