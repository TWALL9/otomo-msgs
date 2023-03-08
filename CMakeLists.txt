cmake_minimum_required(VERSION 3.0.2)
project(otomo_msgs)

add_compile_options(-std=c++14)

find_package(catkin REQUIRED COMPONENTS
  message_generation
  message_runtime
  std_msgs
  rospy
)

find_package(Protobuf REQUIRED)

add_message_files(
  FILES
  Joystick.msg
)

generate_messages(
  DEPENDENCIES
  std_msgs
)

set(PROTO_DIR ${PROJECT_SOURCE_DIR}/otomo-protobuf)
file(GLOB PROTO_FILES ${PROTO_DIR}/*.proto)
message(STATUS "Proto dir: ${PROTO_DIR}")
message(STATUS "Proto files: ${PROTO_FILES}")

set(PROTO_GEN_CPP_DIR ${CATKIN_DEVEL_PREFIX}/${CATKIN_PACKAGE_INCLUDE_DESTINATION})
set(PROTO_GEN_PY_DIR ${CATKIN_DEVEL_PREFIX}/${CATKIN_PACKAGE_PYTHON_DESTINATION})
file(MAKE_DIRECTORY ${PROTO_GEN_CPP_DIR})
file(MAKE_DIRECTORY ${PROTO_GEN_PY_DIR})

set(PROTO_GEN_CPP_FILES "")
set(PROTO_GEN_PY_FILES "")
foreach(PROTO_FILE ${PROTO_FILES})
  get_filename_component(PROTO_NAME ${PROTO_FILE} NAME_WE)
  list(APPEND PROTO_GEN_CPP_FILES ${PROTO_GEN_CPP_DIR}/${PROTO_NAME}.pb.h ${PROTO_GEN_CPP_DIR}/${PROTO_NAME}.pb.cc)
  list(APPEND PROTO_GEN_PY_FILES ${PROTO_GEN_PY_DIR}/${PROTO_NAME}_pb2.py)
endforeach(PROTO_FILE ${PROTO_FILES})

add_custom_command(
  OUTPUT ${PROTO_GEN_CPP_FILES} ${PROTO_GEN_PY_FILES}
  COMMAND ${PROTOBUF_PROTOC_EXECUTABLE} --proto_path=${PROTO_DIR} --cpp_out=${PROTO_GEN_CPP_DIR} --python_out=${PROTO_GEN_PY_DIR} ${PROTO_FILES}
  DEPENDS ${PROTOBUF_PROTOC_EXECUTABLE} ${PROTO_FILES}
  WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
)

set_source_files_properties(${PROTO_GEN_CPP_FILES} PROPERTIES GENERATED TRUE)
set_source_files_properties(${PROTO_GEN_PY_FILES} PROPERTIES GENERATED TRUE)

add_custom_target(
  ${PROJECT_NAME}_generate_headers
  DEPENDS ${PROTO_GEN_CPP_FILES} ${PROTO_GEN_PY_FILES}
)

catkin_package(
  DEPENDS Protobuf
  LIBRARIES ${PROJECT_NAME}_proto
)

include_directories(
  ${catkin_INCLUDE_DIRS}
)

# Declare a C++ library
add_library(
  ${PROJECT_NAME}_proto
  ${PROTO_GEN_CPP_FILES}
)

target_link_libraries(${PROJECT_NAME}_proto ${PROTOBUF_LIBRARY})
add_dependencies(${PROJECT_NAME}_proto ${PROJECT_NAME}_generate_headers)

# Mark libraries for installation
# See http://docs.ros.org/melodic/api/catkin/html/howto/format1/building_libraries.html
install(TARGETS ${PROJECT_NAME}_proto
  ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  RUNTIME DESTINATION ${CATKIN_GLOBAL_BIN_DESTINATION}
)

install(DIRECTORY ${PROTO_GEN_CPP_DIR}/
  DESTINATION ${CATKIN_PACKAGE_INCLUDE_DESTINATION}
  FILES_MATCHING PATTERN "*.h"
)

install(DIRECTORY ${PROTO_GEN_PY_DIR}/
  DESTINATION ${CATKIN_PACKAGE_PYTHON_DESTINATION}
  FILES_MATCHING PATTERN "*.py"
)
