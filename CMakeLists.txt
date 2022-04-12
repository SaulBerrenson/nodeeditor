cmake_minimum_required(VERSION 3.7.2)
project(NodeEditor CXX)

include(CMakeDependentOption)

#-------------------------------------------------------------------------------------------------------------------------------------------
# Options

get_directory_property(_has_parent PARENT_DIRECTORY)

option(NODE_EDITOR_QT5 "Use Qt5" ON)
option(NODE_EDITOR_FORCE_TEST_COLOR "Force colorized unit test output" OFF)
cmake_dependent_option(NODE_EDITOR_BUILD_TESTING "Build tests" ON "NOT BUILD_TESTING" OFF)
cmake_dependent_option(NODE_EDITOR_BUILD_EXAMPLES "Build examples" ON "NOT BUILD_EXAMPLES" OFF)
cmake_dependent_option(NODE_EDITOR_BUILD_SHARED_LIBS "Build as shared library" ON "NOT BUILD_SHARED_LIBS" OFF)

#-------------------------------------------------------------------------------------------------------------------------------------------
# Global options

set(CMAKE_CXX_STANDARD 14)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_POSITION_INDEPENDENT_CODE ON)
set(CMAKE_DISABLE_IN_SOURCE_BUILD ON)
set(CMAKE_DISABLE_SOURCE_CHANGES  ON)
set(OpenGL_GL_PREFERENCE LEGACY)

if(NODE_EDITOR_DEVELOPER_DEFAULTS)
  set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${PROJECT_BINARY_DIR}/bin$<0:>)
  set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${PROJECT_BINARY_DIR}/bin$<0:>)
  set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${PROJECT_BINARY_DIR}/lib$<0:>)
endif()

#-------------------------------------------------------------------------------------------------------------------------------------------
# find packages

if(NODE_EDITOR_QT5)
  find_package(Qt5 REQUIRED 
    COMPONENTS
      Core
      Widgets
      Gui
      OpenGL
  )
  set(QT_TARGETS Qt5::Core Qt5::Widgets Qt5::Gui Qt5::OpenGL)
else()
  find_package(Qt6
    COMPONENTS
    Core
    Widgets
    Gui
    OpenGL
  )
  set(QT_TARGETS Qt::Core Qt::Widgets Qt::Gui Qt::OpenGL)
endif()

add_subdirectory(external)

#-------------------------------------------------------------------------------------------------------------------------------------------
# Add resources

if(NODE_EDITOR_QT5)
  qt5_add_resources(RESOURCES ./resources/resources.qrc)
else()
  qt_add_resources(RESOURCES ./resources/resources.qrc)
endif()

#-------------------------------------------------------------------------------------------------------------------------------------------
# AUTOMOC

set(CMAKE_AUTOMOC ON)

#-------------------------------------------------------------------------------------------------------------------------------------------
# Search source files

file(GLOB_RECURSE NODES_SOURCES_FILES LIST_DIRECTORIES FALSE "src/*.cpp")
file(GLOB_RECURSE NODES_HEADERS_FILES LIST_DIRECTORIES FALSE "include/*")

#-------------------------------------------------------------------------------------------------------------------------------------------
# Library build

if (NODE_EDITOR_BUILD_SHARED_LIBS)
  set(NODE_EDITOR_SHARED_OR_STATIC SHARED)
else ()
  set(NODE_EDITOR_SHARED_OR_STATIC STATIC)
endif()

add_library(nodes ${NODE_EDITOR_SHARED_OR_STATIC}
  ${NODES_SOURCES_FILES}
  ${NODES_HEADERS_FILES}
  ${RESOURCES}
)
add_library(NodeEditor::nodes ALIAS nodes)

target_include_directories(nodes
  PUBLIC
    $<INSTALL_INTERFACE:include>
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
  PRIVATE
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/src>
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include/nodes/internal>
)

target_link_libraries(nodes PUBLIC ${QT_TARGETS})

target_compile_definitions(nodes
  PUBLIC
    NODE_EDITOR_SHARED
  PRIVATE
    NODE_EDITOR_EXPORTS
    QT_NO_KEYWORDS
)

#-------------------------------------------------------------------------------------------------------------------------------------------
# MOC

file(GLOB_RECURSE HEADERS_TO_MOC include/nodes/internal/*.hpp)

qt_wrap_cpp(nodes_moc
  ${HEADERS_TO_MOC}
  TARGET nodes
  OPTIONS --no-notes # Don't display a note for the headers which don't produce a moc_*.cpp
)

target_sources(nodes PRIVATE ${nodes_moc})

#-------------------------------------------------------------------------------------------------------------------------------------------
# Examples

if(NODE_EDITOR_BUILD_EXAMPLES)
  add_subdirectory(examples)
endif()

#-------------------------------------------------------------------------------------------------------------------------------------------
# Tests

if(NODE_EDITOR_BUILD_TESTING)
  add_subdirectory(test)
endif()

#-------------------------------------------------------------------------------------------------------------------------------------------
# Install

include(GNUInstallDirs)
include(CMakePackageConfigHelpers)

set(INSTALL_CONFIGDIR ${CMAKE_INSTALL_LIBDIR}/cmake/NodeEditor)

install(TARGETS nodes
  EXPORT NodeEditorTargets
  LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
  ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
  RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
)

install(DIRECTORY include/ DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})

install(EXPORT NodeEditorTargets
  FILE NodeEditorTargets.cmake
  NAMESPACE NodeEditor::
  DESTINATION ${INSTALL_CONFIGDIR}
)

configure_package_config_file(${CMAKE_CURRENT_LIST_DIR}/cmake/NodeEditorConfig.cmake.in
  ${CMAKE_CURRENT_BINARY_DIR}/NodeEditorConfig.cmake
  INSTALL_DESTINATION ${INSTALL_CONFIGDIR}
)

install(FILES
  ${CMAKE_CURRENT_BINARY_DIR}/NodeEditorConfig.cmake
  DESTINATION ${INSTALL_CONFIGDIR}
)
