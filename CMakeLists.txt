cmake_minimum_required(VERSION 3.5)
project(rplidar_ros)

set(RPLIDAR_SDK_PATH "./sdk/")

FILE(GLOB RPLIDAR_SDK_SRC 
  "${RPLIDAR_SDK_PATH}/src/arch/win32/*.cpp"
  "${RPLIDAR_SDK_PATH}/src/hal/*.cpp"
  "${RPLIDAR_SDK_PATH}/src/*.cpp"
)

find_package(ament_cmake REQUIRED)
find_package(rclcpp REQUIRED)
find_package(sensor_msgs REQUIRED)
find_package(std_srvs REQUIRED)
find_package(rostime REQUIRED)

add_definitions(-DROS2)

include_directories(
  ${RPLIDAR_SDK_PATH}/include
  ${RPLIDAR_SDK_PATH}/src
  ${sensor_msgs_INCLUDE_DIRS}
  ${rclcpp_INCLUDE_DIRS} 
  ${std_srvs_INCLUDE_DIRS} 
  ${rostime_INCLUDE_DIRS}
  $ENV{BOOST_ROOT}
)

if(CMAKE_BUILD_TYPE MATCHES Debug)
set(BOOST_DEPENDENCIES
  $ENV{BOOST_LIBRARYDIR}/libboost_date_time-vc140-mt-gd-1_61.lib
  $ENV{BOOST_LIBRARYDIR}/libboost_thread-vc140-mt-gd-1_61.lib
  $ENV{BOOST_LIBRARYDIR}/libboost_system-vc140-mt-gd-1_61.lib
  $ENV{BOOST_LIBRARYDIR}/libboost_chrono-vc140-mt-gd-1_61.lib
)
else()
set(BOOST_DEPENDENCIES
  $ENV{BOOST_LIBRARYDIR}/libboost_date_time-vc140-mt-1_61.lib
  $ENV{BOOST_LIBRARYDIR}/libboost_thread-vc140-mt-1_61.lib
  $ENV{BOOST_LIBRARYDIR}/libboost_system-vc140-mt-1_61.lib
  $ENV{BOOST_LIBRARYDIR}/libboost_chrono-vc140-mt-1_61.lib
)
endif()

set(RPLIDAR_DEPENDENCIES
  ${rclcpp_LIBRARIES}
  ${sensor_msgs_LIBRARIES}
  ${std_srvs_LIBRARIES}
  ${rostime_LIBRARIES}
  ${BOOST_DEPENDENCIES}
)

add_executable(rplidarNode src/node.cpp ${RPLIDAR_SDK_SRC})
target_link_libraries(rplidarNode ${RPLIDAR_DEPENDENCIES})

add_executable(rplidarNodeClient src/client.cpp)
target_link_libraries(rplidarNodeClient ${RPLIDAR_DEPENDENCIES})

install(TARGETS rplidarNode rplidarNodeClient
  ARCHIVE DESTINATION lib
  LIBRARY DESTINATION lib
  RUNTIME DESTINATION bin
)

install(DIRECTORY launch rviz sdk
  DESTINATION share
  USE_SOURCE_PERMISSIONS
)
