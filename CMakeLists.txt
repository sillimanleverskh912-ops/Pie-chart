cmake_minimum_required(VERSION 3.16)

project(Pie_Chart_Control VERSION 0.1 LANGUAGES CXX)

set(CMAKE_AUTOUIC ON)
set(CMAKE_AUTOMOC ON)
set(CMAKE_AUTORCC ON)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# Qt Creator kits set CMAKE_PREFIX_PATH; generic IDE shells often do not.
if(DEFINED ENV{Qt6_DIR})
    get_filename_component(_qt6_prefix "$ENV{Qt6_DIR}/../../.." ABSOLUTE)
    if(EXISTS "${_qt6_prefix}/lib/cmake/Qt6/Qt6Config.cmake")
        list(PREPEND CMAKE_PREFIX_PATH "${_qt6_prefix}")
    endif()
endif()
if(DEFINED ENV{Qt5_DIR})
    get_filename_component(_qt5_prefix "$ENV{Qt5_DIR}/../../.." ABSOLUTE)
    if(EXISTS "${_qt5_prefix}/lib/cmake/Qt5/Qt5Config.cmake")
        list(PREPEND CMAKE_PREFIX_PATH "${_qt5_prefix}")
    endif()
endif()
if(DEFINED ENV{QTDIR})
    list(PREPEND CMAKE_PREFIX_PATH "$ENV{QTDIR}")
endif()
# Windows: if Qt is still missing, try common kit folders (do not mix MSVC with MinGW Qt).
if(WIN32)
    set(_qt_glob_matched FALSE)
    if(MSVC)
        set(_qt_kit_globs
            "F:/QtCreater/*/msvc2022_64"
            "F:/QtCreater/*/msvc2019_64"
            "C:/Qt/*/msvc2022_64"
            "C:/Qt/*/msvc2019_64"
            "D:/Qt/*/msvc2022_64"
            "D:/Qt/*/msvc2019_64")
    else()
        set(_qt_kit_globs
            "F:/QtCreater/*/mingw_64"
            "C:/Qt/*/mingw_64"
            "D:/Qt/*/mingw_64")
    endif()
    foreach(_qt_glob IN LISTS _qt_kit_globs)
        if(_qt_glob_matched)
            break()
        endif()
        file(GLOB _qt_dirs "${_qt_glob}")
        if(NOT _qt_dirs)
            continue()
        endif()
        list(SORT _qt_dirs COMPARE NATURAL ORDER DESCENDING)
        foreach(_qt_d IN LISTS _qt_dirs)
            if(EXISTS "${_qt_d}/lib/cmake/Qt6/Qt6Config.cmake")
                list(PREPEND CMAKE_PREFIX_PATH "${_qt_d}")
                set(_qt_glob_matched TRUE)
                break()
            endif()
        endforeach()
    endforeach()
endif()

find_package(QT NAMES Qt6 Qt5 REQUIRED COMPONENTS Widgets)
find_package(Qt${QT_VERSION_MAJOR} REQUIRED COMPONENTS Widgets)

set(CHART_CORE_SOURCES
        widget.cpp
        widget.h
        VarCountData.h
        PieChartConfig.cpp
        PieChartConfig.h
        PieChartConfigDialog.cpp
        PieChartConfigDialog.h
        SinglePieChart.cpp
        SinglePieChart.h
)

set(PROJECT_SOURCES
        main.cpp
        ${CHART_CORE_SOURCES}
)

if(${QT_VERSION_MAJOR} GREATER_EQUAL 6)
    qt_add_executable(Pie_Chart_Control
        MANUAL_FINALIZATION
        ${PROJECT_SOURCES}
    )
# Define target properties for Android with Qt 6 as:
#    set_property(TARGET Pie_Chart_Control APPEND PROPERTY QT_ANDROID_PACKAGE_SOURCE_DIR
#                 ${CMAKE_CURRENT_SOURCE_DIR}/android)
# For more information, see https://doc.qt.io/qt-6/qt-add-executable.html#target-creation
else()
    if(ANDROID)
        add_library(Pie_Chart_Control SHARED
            ${PROJECT_SOURCES}
        )
# Define properties for Android with Qt 5 after find_package() calls as:
#    set(ANDROID_PACKAGE_SOURCE_DIR "${CMAKE_CURRENT_SOURCE_DIR}/android")
    else()
        add_executable(Pie_Chart_Control
            ${PROJECT_SOURCES}
        )
    endif()
endif()

target_link_libraries(Pie_Chart_Control PRIVATE Qt${QT_VERSION_MAJOR}::Widgets)

if(MSVC)
    target_compile_options(Pie_Chart_Control PRIVATE /Zc:__cplusplus)
endif()

# Qt for iOS sets MACOSX_BUNDLE_GUI_IDENTIFIER automatically since Qt 6.1.
# If you are developing for iOS or macOS you should consider setting an
# explicit, fixed bundle identifier manually though.
if(${QT_VERSION} VERSION_LESS 6.1.0)
  set(BUNDLE_ID_OPTION MACOSX_BUNDLE_GUI_IDENTIFIER com.example.Pie_Chart_Control)
endif()
set_target_properties(Pie_Chart_Control PROPERTIES
    ${BUNDLE_ID_OPTION}
    MACOSX_BUNDLE_BUNDLE_VERSION ${PROJECT_VERSION}
    MACOSX_BUNDLE_SHORT_VERSION_STRING ${PROJECT_VERSION_MAJOR}.${PROJECT_VERSION_MINOR}
    MACOSX_BUNDLE TRUE
    WIN32_EXECUTABLE TRUE
)

include(GNUInstallDirs)
install(TARGETS Pie_Chart_Control
    BUNDLE DESTINATION .
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
)

if(QT_VERSION_MAJOR EQUAL 6)
    qt_finalize_executable(Pie_Chart_Control)
endif()

# ---------- 饼图展示与配置测试程序（定时模拟数据 + 独立配置持久化键）----------

set(PIECHART_DISPLAY_TEST_SOURCES
    PieChartDisplayTest/main.cpp
    PieChartDisplayTest/TestHarnessWindow.cpp
    PieChartDisplayTest/TestHarnessWindow.h
)

if(${QT_VERSION_MAJOR} GREATER_EQUAL 6)
    qt_add_executable(PieChartDisplayTest
        MANUAL_FINALIZATION
        ${CHART_CORE_SOURCES}
        ${PIECHART_DISPLAY_TEST_SOURCES}
    )
else()
    add_executable(PieChartDisplayTest
        ${CHART_CORE_SOURCES}
        ${PIECHART_DISPLAY_TEST_SOURCES}
    )
endif()

target_link_libraries(PieChartDisplayTest PRIVATE Qt${QT_VERSION_MAJOR}::Widgets)

target_include_directories(PieChartDisplayTest PRIVATE ${CMAKE_CURRENT_SOURCE_DIR})

if(MSVC)
    target_compile_options(PieChartDisplayTest PRIVATE /Zc:__cplusplus)
endif()

set_target_properties(PieChartDisplayTest PROPERTIES
    WIN32_EXECUTABLE TRUE
)

if(${QT_VERSION_MAJOR} GREATER_EQUAL 6)
    qt_finalize_executable(PieChartDisplayTest)
endif()

install(TARGETS PieChartDisplayTest
    BUNDLE DESTINATION .
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
)
