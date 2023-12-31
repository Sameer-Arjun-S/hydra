cmake_minimum_required(VERSION 3.8)
project(hydra)

set(CMAKE_EXPORT_COMPILE_COMMANDS ON)
execute_process(COMMAND "rm" "-f" "${CMAKE_SOURCE_DIR}/compile_commands.json")
execute_process(COMMAND "ln" "-s" "-r" "${CMAKE_BINARY_DIR}/compile_commands.json" "${CMAKE_SOURCE_DIR}/")

if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Wall -Wextra -Wpedantic)
endif()

find_package(ament_cmake REQUIRED)
find_package(rclcpp REQUIRED)
find_package(std_msgs REQUIRED)
find_package(tf2 REQUIRED)
find_package(tf2_ros REQUIRED)
find_package(geometry_msgs REQUIRED)
find_package(sensor_msgs REQUIRED)
find_package(nav_msgs REQUIRED)

include_directories(include)

add_executable(x_publisher src/main.cpp)
ament_target_dependencies(x_publisher tf2 tf2_ros rclcpp std_msgs sensor_msgs geometry_msgs nav_msgs)
add_library(library src/publisher.cpp src/master.cpp src/custom_trajectory.cpp)
target_include_directories(library PUBLIC "${CMAKE_CURRENT_SOURCE_DIR}/include")
ament_target_dependencies(library tf2 tf2_ros rclcpp std_msgs sensor_msgs geometry_msgs nav_msgs)


target_link_directories(x_publisher PUBLIC ${CMAKE_CURRENT_BINARY_DIR})
target_link_libraries(x_publisher library)

install(TARGETS
  x_publisher
  DESTINATION lib/${PROJECT_NAME})

install(DIRECTORY
launch
worlds
models
DESTINATION share/${PROJECT_NAME}
)

if(COVERAGE)
  set(CMAKE_BUILD_TYPE "Debug")
  set(CMAKE_CXX_FLAGS "--coverage")
  set(CMAKE_CXX_OUTPUT_EXTENSION_REPLACE "ON")
  set(ARG_CONFIG_FILE "${PROJECT_SOURCE_DIR}/uncrustify_google_cpp_allman.cfg"
    CACHE STRING "uncrustify config file")
endif()

if(BUILD_TESTING)
  find_package(ament_cmake_gtest REQUIRED)
  find_package(ament_lint_auto REQUIRED)

  # the following lines skip linters
  set(ament_cmake_cppcheck_FOUND TRUE)
  set(ament_cmake_copyright_FOUND TRUE)
  set(ament_cmake_cpplint_FOUND TRUE)
  set(ament_cmake_flake8_FOUND TRUE)
  set(ament_cmake_uncrustify_FOUND TRUE)
  set(ament_cmake_xmllint_FOUND TRUE)
  set(ament_cmake_lint_cmake_FOUND TRUE)
  ament_lint_auto_find_test_dependencies()

  ament_lint_auto_find_test_dependencies()
  set(TESTFILES
    test/main.cpp
    test/test.cpp
    test/publisher_test.cpp
    src/custom_trajectory.cpp
    include/custom_trajectory.hpp
    )
  ament_add_gtest(test_${PROJECT_NAME} ${TESTFILES})
  ament_target_dependencies(test_${PROJECT_NAME} tf2 tf2_ros rclcpp std_msgs sensor_msgs geometry_msgs nav_msgs)

  install(TARGETS
    test_${PROJECT_NAME}
    DESTINATION lib/${PROJECT_NAME})
endif()

## Install some arbitrary executables
install(PROGRAMS      # want to make the file executable (ie. chmod a+x)
  scripts/generate_coverage_report.bash
  DESTINATION lib/${PROJECT_NAME})

ament_package()
