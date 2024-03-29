cmake_minimum_required(VERSION 3.0.2)
project(cob_fiducials)

set(catkin_RUN_PACKAGES 
  cob_object_detection_msgs
  cob_vision_utils
  cv_bridge
  diagnostic_msgs
  diagnostic_updater
  image_transport
  roscpp
  sensor_msgs
  std_msgs
  std_srvs
  tf
  visualization_msgs
)

set(catkin_BUILD_PACKAGES
  cmake_modules
  ${catkin_RUN_PACKAGES}
)

find_package(catkin REQUIRED COMPONENTS
  ${catkin_BUILD_PACKAGES}
)

find_package(OpenCV REQUIRED)
find_package(TinyXML REQUIRED)
find_package(Boost REQUIRED COMPONENTS filesystem)

catkin_package(
INCLUDE_DIRS
  common/include
LIBRARIES
  ${PROJECT_NAME}
CATKIN_DEPENDS
  ${catkin_RUN_PACKAGES}
DEPENDS
  Boost
  OpenCV
  TinyXML
)


###########
## Build ##
###########
include_directories(
  common/include
  ${catkin_INCLUDE_DIRS}
  ${OpenCV_INCLUDE_DIRS}
  ${TinyXML_INCLUDE_DIRS}
  ${Boost_INCLUDE_DIRS}
)

set(project_CPP_FILES
  common/src/AbstractFiducialModel.cpp
  common/src/aruco/FiducialModelAruco.cpp
  common/src/aruco/arucofidmarkers.cpp
  common/src/aruco/board.cpp
  common/src/aruco/boarddetector.cpp
  common/src/aruco/cameraparameters.cpp
  common/src/aruco/cvdrawingutils.cpp
  common/src/aruco/marker.cpp
  common/src/aruco/markerdetector.cpp
  common/src/pi/FiducialModelPi.cpp
)

add_library(${PROJECT_NAME} 
  ${project_CPP_FILES}
)
add_dependencies(${PROJECT_NAME} ${catkin_EXPORTED_TARGETS})
target_link_libraries(${PROJECT_NAME}
  ${catkin_LIBRARIES}
  ${OpenCV_LIBS}
  ${TinyXML_LIBRARIES}
  ${Boost_LIBRARIES}
)
set_target_properties(${PROJECT_NAME} PROPERTIES COMPILE_FLAGS "-D__LINUX__ -DBOOST_FILESYSTEM_VERSION=2")

add_executable(fiducials
  ros/src/fiducials.cpp
)
add_dependencies(fiducials ${catkin_EXPORTED_TARGETS})
target_link_libraries(fiducials
  ${PROJECT_NAME}
  ${catkin_LIBRARIES}
  ${OpenCV_LIBS}
  ${TinyXML_LIBRARIES}
  ${Boost_LIBRARIES}
)
set_target_properties(fiducials PROPERTIES COMPILE_FLAGS "-D__LINUX__ -DBOOST_FILESYSTEM_VERSION=2")

#############
## Install ##
#############
install(TARGETS ${PROJECT_NAME} fiducials
  ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  RUNTIME DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
)

install(DIRECTORY common/include/${PROJECT_NAME}/
  DESTINATION ${CATKIN_PACKAGE_INCLUDE_DESTINATION}
  FILES_MATCHING PATTERN "*.h"
)

install(DIRECTORY ros/launch
  DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}/ros
)

install(DIRECTORY common/files
  DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}/common
)

catkin_install_python(PROGRAMS
  ros/scripts/fiducials_detector_service_client.py
  DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
)
