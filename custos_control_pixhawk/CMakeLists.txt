cmake_minimum_required(VERSION 3.16)
project(custos_control_pixhawk)

if(NOT CMAKE_CXX_STANDARD)
  set(CMAKE_CXX_STANDARD 20)
endif()

find_package(ament_cmake REQUIRED)
find_package(rclcpp REQUIRED)
find_package(rclcpp_lifecycle REQUIRED)
find_package(lifecycle_msgs REQUIRED)
find_package(custos_common_msgs REQUIRED)
find_package(custos_control_msgs REQUIRED)

# Targets land here once the uXRCE-DDS vs MAVROS choice is made (see future ADR).

if(BUILD_TESTING)
  find_package(ament_lint_auto REQUIRED)
  ament_lint_auto_find_test_dependencies()
endif()

ament_package()
