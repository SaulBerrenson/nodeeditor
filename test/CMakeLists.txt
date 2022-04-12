find_package(Catch2 2.3.0 REQUIRED)

enable_testing()

if(NODE_EDITOR_QT5)
  find_package(Qt5 REQUIRED COMPONENTS Test )
  set(QT_TEST_TARGET Qt5::Test)
else()
  find_package(Qt6 REQUIRED COMPONENTS Test )
    set(QT_TEST_TARGET Qt::Test)
endif()

add_executable(test_nodes
  test_main.cpp
  src/TestDragging.cpp
  src/TestDataModelRegistry.cpp
  src/TestFlowScene.cpp
  src/TestNodeGraphicsObject.cpp
)

target_include_directories(test_nodes
  PRIVATE
    ../src
    ../include/internal
    include
)

target_link_libraries(test_nodes
  PRIVATE
    NodeEditor::nodes
    Catch2::Catch2
    ${QT_TEST_TARGET}
)

add_test(
  NAME test_nodes
  COMMAND
    $<TARGET_FILE:test_nodes>
    $<$<BOOL:${NODE_EDITOR_FORCE_TEST_COLOR}>:--use-colour=yes>
)
