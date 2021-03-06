project(xdebug)
cmake_minimum_required(VERSION 2.8.9)

set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} ${xdebug_SOURCE_DIR}/cmake/)

find_package(KDE4 REQUIRED)
find_package(KDevPlatform 1.5.60 REQUIRED)

set(VERSION_MAJOR 1)
set(VERSION_MINOR 5)
set(VERSION_PATCH 60)

configure_file(
  "${CMAKE_CURRENT_SOURCE_DIR}/version.h.cmake"
  "${CMAKE_CURRENT_BINARY_DIR}/version.h"
  @ONLY
)

remove_definitions( -DKDE_DEFAULT_DEBUG_AREA=24000 )
add_definitions( -DKDE_DEFAULT_DEBUG_AREA=9046 )

if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID STREQUAL "Clang" OR CMAKE_CXX_COMPILER_ID STREQUAL "Intel")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")
endif()

enable_testing()

find_path( EXECUTEBROWSER_INCLUDE_DIR iexecutebrowserplugin.h
 PATH_SUFFIXES kdevexecutebrowser )

include_directories(
    ${KDEVPLATFORM_INCLUDE_DIR}
    ${KDE4_INCLUDES}
    ${CMAKE_CURRENT_BINARY_DIR}
    ${CMAKE_CURRENT_SOURCE_DIR}
    ${EXECUTEBROWSER_INCLUDE_DIR}
)

########### next target ###############

set(kdevxdebugcore_PART_SRCS
    connection.cpp
    debugsession.cpp
    debugjob.cpp
    framestackmodel.cpp
    breakpointcontroller.cpp
    variablecontroller.cpp
    variable.cpp
    stringhelpers.cpp
)

set(kdevxdebug_PART_SRCS
    xdebugplugin.cpp
    launchconfig.cpp
    launchconfigurationpage.cpp
)
kde4_add_ui_files(kdevxdebug_PART_SRCS launchconfigurationpage.ui)

kde4_add_plugin(kdevxdebug ${kdevxdebug_PART_SRCS} ${kdevxdebugcore_PART_SRCS})

target_link_libraries(kdevxdebug
    ${KDEVPLATFORM_INTERFACES_LIBRARIES}
    ${KDEVPLATFORM_DEBUGGER_LIBRARIES}
    ${KDEVPLATFORM_OUTPUTVIEW_LIBRARIES}
    ${KDEVPLATFORM_UTIL_LIBRARIES}
)

install(TARGETS kdevxdebug DESTINATION ${PLUGIN_INSTALL_DIR})


########### next target ###############

set(connectiontest_SRCS
    tests/connectiontest.cpp
)

kde4_add_unit_test(connectiontest ${connectiontest_SRCS} ${kdevxdebugcore_PART_SRCS})

target_link_libraries(connectiontest
    ${QT_QTTEST_LIBRARY}
    ${KDEVPLATFORM_INTERFACES_LIBRARIES}
    ${KDEVPLATFORM_TESTSHELL_LIBRARIES}
    ${KDEVPLATFORM_DEBUGGER_LIBRARIES}
    ${KDEVPLATFORM_TESTS_LIBRARIES}
    ${KDEVPLATFORM_OUTPUTVIEW_LIBRARIES}
    ${KDEVPLATFORM_UTIL_LIBRARIES}
)


########### install files ###############
configure_file(kdevxdebug.desktop.cmake ${CMAKE_CURRENT_BINARY_DIR}/kdevxdebug.desktop)
install(FILES ${CMAKE_CURRENT_BINARY_DIR}/kdevxdebug.desktop DESTINATION ${SERVICES_INSTALL_DIR})
