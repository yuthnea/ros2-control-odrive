cmake_minimum_required(VERSION 3.5)
project(diffbot_odrive_hardware_interface)

# Default to C99
if(NOT CMAKE_C_STANDARD)
  set(CMAKE_C_STANDARD 99)
endif()

# Default to C++14
if(NOT CMAKE_CXX_STANDARD)
  set(CMAKE_CXX_STANDARD 14)
endif()

if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Wall -Wextra -Wpedantic)
endif()

# find dependencies
find_package(ament_cmake REQUIRED)
find_package(hardware_interface REQUIRED)
find_package(pluginlib REQUIRED)
find_package(rclcpp REQUIRED)
find_package(PkgConfig REQUIRED)
pkg_search_module(LIBUSB1 REQUIRED libusb-1.0)

if(LIBUSB1_FOUND)
  include_directories(SYSTEM ${LIBUSB1_INCLUDE_DIRS})
else()
  message( FATAL_ERROR "Failed to find libusb-1.0" )
endif()

add_library(
  odrive_usb
  SHARED
  diffbot_odrive_hardware_interface/odrive_usb.cpp
)
target_include_directories(
  odrive_usb
  PRIVATE
  include
)
target_link_libraries(
  odrive_usb
  ${LIBUSB1_LIBRARIES}
)
add_library(
  ${PROJECT_NAME}
  SHARED
  diffbot_odrive_hardware_interface/odrive_hardware_interface.cpp
)
target_include_directories(
  ${PROJECT_NAME}
  PRIVATE
  include
)
target_link_libraries(
  ${PROJECT_NAME}
  odrive_usb
)
ament_target_dependencies(
  ${PROJECT_NAME}
  hardware_interface
  pluginlib
  rclcpp
)
pluginlib_export_plugin_description_file(hardware_interface diffbot_odrive_hardware_interface.xml)
install(
  TARGETS ${PROJECT_NAME} odrive_usb
  DESTINATION lib
)


# uncomment the following section in order to fill in
# further dependencies manually.
# find_package(<dependency> REQUIRED)

if(BUILD_TESTING)
  find_package(ament_lint_auto REQUIRED)
  # the following line skips the linter which checks for copyrights
  # uncomment the line when a copyright and license is not present in all source files
  #set(ament_cmake_copyright_FOUND TRUE)
  # the following line skips cpplint (only works in a git repo)
  # uncomment the line when this package is not in a git repo
  #set(ament_cmake_cpplint_FOUND TRUE)
  ament_lint_auto_find_test_dependencies()
endif()

ament_package()
