cmake_minimum_required(VERSION 3.8)
project(world_info)

# Default to C++14
if(NOT CMAKE_CXX_STANDARD)
  set(CMAKE_CXX_STANDARD 14)
endif()

if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Wall -Wextra -Wpedantic)
endif()

# find dependencies
find_package(ament_cmake REQUIRED)
find_package(rclcpp REQUIRED)
find_package(rclcpp_components REQUIRED)
find_package(world_info_msgs REQUIRED)
find_package(tf2_ros REQUIRED)
find_package(tf2_geometry_msgs REQUIRED)
find_package(sensor_msgs REQUIRED)
find_package(Eigen3 REQUIRED)
find_package(OpenCV REQUIRED)
find_package(OpenVINO REQUIRED)
find_package(cv_bridge REQUIRED)
find_package(image_transport REQUIRED)

add_executable(world_info src/world_info.cpp)
ament_target_dependencies(world_info rclcpp world_info_msgs tf2_ros tf2_geometry_msgs)

add_library(qrcode SHARED src/qrcode.cpp)
ament_target_dependencies(qrcode rclcpp rclcpp_components world_info_msgs sensor_msgs image_transport Eigen3 OpenCV cv_bridge tf2_ros)
target_link_libraries(qrcode ${ZBAR_LIBRARIES} zbar)
rclcpp_components_register_node(qrcode
PLUGIN "world_info::DetectQR"
EXECUTABLE qrcode_node)

add_library(aruco SHARED src/aruco.cpp)
ament_target_dependencies(aruco rclcpp rclcpp_components world_info_msgs sensor_msgs image_transport Eigen3 OpenCV cv_bridge tf2_ros)
rclcpp_components_register_node(aruco
PLUGIN "world_info::DetectAruco"
EXECUTABLE aruco_node)

add_library(rs_depth SHARED src/rs_depth.cpp)
ament_target_dependencies(rs_depth rclcpp rclcpp_components sensor_msgs image_transport OpenCV cv_bridge world_info_msgs)
rclcpp_components_register_node(rs_depth
PLUGIN "world_info::RSDepth"
EXECUTABLE rs_depth_node)

add_library(hazmat SHARED src/hazmat.cpp)
ament_target_dependencies(hazmat rclcpp rclcpp_components world_info_msgs sensor_msgs image_transport OpenCV cv_bridge tf2_ros)
target_link_libraries(hazmat $ENV{INTEL_OPENVINO_DIR}/runtime/lib/intel64/libopenvino.so)
target_include_directories(hazmat PUBLIC $ENV{INTEL_OPENVINO_DIR}/runtime/include $ENV{INTEL_OPENVINO_DIR}/runtime/include/ie)
rclcpp_components_register_node(hazmat
PLUGIN "world_info::DetectHazmat"
EXECUTABLE hazmat_node)

add_library(hazmat_seg SHARED src/hazmat_seg.cpp)
ament_target_dependencies(hazmat_seg rclcpp rclcpp_components world_info_msgs sensor_msgs image_transport OpenCV cv_bridge)
target_link_libraries(hazmat_seg $ENV{INTEL_OPENVINO_DIR}/runtime/lib/intel64/libopenvino.so)
target_include_directories(hazmat_seg PUBLIC $ENV{INTEL_OPENVINO_DIR}/runtime/include $ENV{INTEL_OPENVINO_DIR}/runtime/include/ie)
rclcpp_components_register_node(hazmat_seg
PLUGIN "world_info::DetectHazmatSeg"
EXECUTABLE hazmat_seg_node)

add_library(babyface SHARED src/babyface.cpp)
ament_target_dependencies(babyface rclcpp rclcpp_components world_info_msgs sensor_msgs image_transport OpenCV cv_bridge tf2_ros)
target_link_libraries(babyface $ENV{INTEL_OPENVINO_DIR}/runtime/lib/intel64/libopenvino.so)
target_include_directories(babyface PUBLIC $ENV{INTEL_OPENVINO_DIR}/runtime/include $ENV{INTEL_OPENVINO_DIR}/runtime/include/ie)
rclcpp_components_register_node(babyface
PLUGIN "world_info::DetectBabyface"
EXECUTABLE babyface_node)

add_library(firenet SHARED src/firenet.cpp)
ament_target_dependencies(firenet rclcpp rclcpp_components world_info_msgs sensor_msgs image_transport OpenCV cv_bridge)
target_link_libraries(firenet $ENV{INTEL_OPENVINO_DIR}/runtime/lib/intel64/libopenvino.so)
target_include_directories(firenet PUBLIC $ENV{INTEL_OPENVINO_DIR}/runtime/include $ENV{INTEL_OPENVINO_DIR}/runtime/include/ie)
rclcpp_components_register_node(firenet
PLUGIN "world_info::DetectFirenet"
EXECUTABLE firenet_node)

add_library(door_seg SHARED src/door_seg.cpp)
ament_target_dependencies(door_seg rclcpp rclcpp_components world_info_msgs sensor_msgs image_transport OpenCV cv_bridge)
target_link_libraries(door_seg $ENV{INTEL_OPENVINO_DIR}/runtime/lib/intel64/libopenvino.so)
target_include_directories(door_seg PUBLIC $ENV{INTEL_OPENVINO_DIR}/runtime/include $ENV{INTEL_OPENVINO_DIR}/runtime/include/ie)
rclcpp_components_register_node(door_seg
PLUGIN "world_info::DetectDoorSeg"
EXECUTABLE door_seg_node)

install(DIRECTORY launch
  DESTINATION share/${PROJECT_NAME})

install(DIRECTORY weights
  DESTINATION share/${PROJECT_NAME})

install(TARGETS
  qrcode
  aruco
  babyface
  firenet
  hazmat
  hazmat_seg
  door_seg
  rs_depth
  ARCHIVE DESTINATION lib
  LIBRARY DESTINATION lib
  RUNTIME DESTINATION bin
  DESTINATION lib/${PROJECT_NAME})

install(TARGETS
  world_info
  DESTINATION lib/${PROJECT_NAME})

ament_package()
