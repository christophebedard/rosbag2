cmake_minimum_required(VERSION 3.5)
project(rosbag2_storage_sqlite3)

# Default to C99
if(NOT CMAKE_C_STANDARD)
  set(CMAKE_C_STANDARD 99)
endif()

# Default to C++17
if(NOT CMAKE_CXX_STANDARD)
  set(CMAKE_CXX_STANDARD 17)
  set(CMAKE_CXX_STANDARD_REQUIRED ON)
endif()

if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Wall -Wextra -Wpedantic)
endif()

# Windows supplies macros for min and max by default. We should only use min and max from stl
if(WIN32)
  add_definitions(-DNOMINMAX)
endif()

find_package(ament_cmake REQUIRED)
find_package(ament_cmake_python REQUIRED)
find_package(pluginlib REQUIRED)
find_package(rcpputils REQUIRED)
find_package(rcutils REQUIRED)
find_package(rosbag2_storage REQUIRED)
find_package(sqlite3_vendor REQUIRED)
find_package(SQLite3 REQUIRED)  # provided by sqlite3_vendor
find_package(yaml_cpp_vendor REQUIRED)
find_package(yaml-cpp REQUIRED)

ament_python_install_package(ros2bag_sqlite3_cli)

add_library(${PROJECT_NAME} SHARED
  src/rosbag2_storage_sqlite3/sqlite_wrapper.cpp
  src/rosbag2_storage_sqlite3/sqlite_storage.cpp
  src/rosbag2_storage_sqlite3/sqlite_statement_wrapper.cpp)

target_link_libraries(${PROJECT_NAME} PUBLIC
  rosbag2_storage::rosbag2_storage
  rcpputils::rcpputils
  rcutils::rcutils
  SQLite::SQLite3
)
target_link_libraries(${PROJECT_NAME} PRIVATE
  pluginlib::pluginlib
  yaml-cpp::yaml-cpp
)

target_include_directories(${PROJECT_NAME}
  PUBLIC
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
  $<INSTALL_INTERFACE:include/${PROJECT_NAME}>
)

# Causes the visibility macros to use dllexport rather than dllimport,
# which is appropriate when building the dll but not consuming it.
target_compile_definitions(${PROJECT_NAME} PRIVATE
  ROSBAG2_STORAGE_DEFAULT_PLUGINS_BUILDING_DLL)

pluginlib_export_plugin_description_file(rosbag2_storage plugin_description.xml)

install(
  DIRECTORY include/
  DESTINATION include/${PROJECT_NAME})

install(
  TARGETS ${PROJECT_NAME}
  ARCHIVE DESTINATION lib
  LIBRARY DESTINATION lib
  RUNTIME DESTINATION bin)

# Export old-style CMake variables
ament_export_include_directories("include/${PROJECT_NAME}")
ament_export_libraries(${PROJECT_NAME})

ament_export_dependencies(rosbag2_storage rcpputils rcutils sqlite3_vendor SQLite3)

if(BUILD_TESTING)
  find_package(ament_cmake_gmock REQUIRED)
  find_package(ament_lint_auto REQUIRED)
  find_package(rosbag2_test_common REQUIRED)
  find_package(std_msgs REQUIRED)
  ament_lint_auto_find_test_dependencies()

  set(TEST_LINK_LIBRARIES
    ${PROJECT_NAME}
    rosbag2_storage::rosbag2_storage
    rosbag2_test_common::rosbag2_test_common
    rcutils::rcutils
  )

  ament_add_gmock(test_sqlite_wrapper
    test/rosbag2_storage_sqlite3/test_sqlite_wrapper.cpp
    WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR})
  if(TARGET test_sqlite_wrapper)
    target_link_libraries(test_sqlite_wrapper ${TEST_LINK_LIBRARIES})
  endif()

  ament_add_gmock(test_sqlite_storage
    test/rosbag2_storage_sqlite3/test_sqlite_storage.cpp
    WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR})
  if(TARGET test_sqlite_storage)
    target_link_libraries(test_sqlite_storage ${TEST_LINK_LIBRARIES})
  endif()

  ament_add_gmock(test_sqlite_topic_filter
    test/rosbag2_storage_sqlite3/test_sqlite_topic_filter.cpp
    WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR})
  if(TARGET test_sqlite_topic_filter)
    target_link_libraries(test_sqlite_topic_filter ${TEST_LINK_LIBRARIES} ${std_msgs_TARGETS})
  endif()
endif()

ament_package()
