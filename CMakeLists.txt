cmake_minimum_required(VERSION 3.10)
project(kinect_ros2)

if(NOT CMAKE_CXX_STANDARD)
  set(CMAKE_CXX_STANDARD 14)
endif()

if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Wall -Wextra -Wpedantic)
endif()

include_directories(include)

# find dependencies
find_package(ament_cmake REQUIRED)
find_package(rclcpp_components REQUIRED)
find_package(ament_index_cpp REQUIRED)
find_package(rclcpp REQUIRED)
# find_package(Threads REQUIRED)

find_package(libfreenect REQUIRED)
find_package(OpenCV REQUIRED)
find_package(sensor_msgs REQUIRED)
find_package(cv_bridge REQUIRED)
find_package(image_transport REQUIRED)
find_package(depth_image_proc REQUIRED)
find_package(camera_info_manager REQUIRED)

add_library(kinect_ros2_component SHARED src/kinect_ros2_component.cpp)
target_compile_definitions(kinect_ros2_component
  PRIVATE "COMPOSITION_BUILDING_DLL")

target_link_libraries(kinect_ros2_component
  PUBLIC
    rclcpp::rclcpp
    sensor_msgs::sensor_msgs
    cv_bridge::cv_bridge
    camera_info_manager::camera_info_manager
    rclcpp_components::component
    ament_index_cpp::ament_index_cpp
    image_transport::image_transport
    ${OpenCV_LIBRARIES}
    freenect)

rclcpp_components_register_nodes(kinect_ros2_component "kinect_ros2::KinectRosComponent")

add_executable(kinect_ros2_node src/kinect_ros2_node.cpp)

target_link_libraries(kinect_ros2_node 
  PUBLIC
    kinect_ros2_component
    rclcpp::rclcpp
    sensor_msgs::sensor_msgs)

install(TARGETS
  kinect_ros2_component
  ARCHIVE DESTINATION lib
  LIBRARY DESTINATION lib
  RUNTIME DESTINATION bin)

install(TARGETS
  kinect_ros2_node
  DESTINATION lib/${PROJECT_NAME})

install(
  DIRECTORY cfg launch rviz
  DESTINATION share/${PROJECT_NAME}
)

ament_package()
