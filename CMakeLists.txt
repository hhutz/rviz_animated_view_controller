cmake_minimum_required(VERSION 2.8.3)
project(rviz_animated_view_controller)

find_package(cmake_modules REQUIRED)
find_package(Eigen3 REQUIRED)
find_package(Boost REQUIRED system filesystem)

# Ogre
pkg_check_modules(OGRE OGRE)

find_package(OpenGL REQUIRED)

find_package(catkin REQUIRED COMPONENTS
 rviz
 pluginlib
 view_controller_msgs
 rqt_gui     # for Qt setup
 qt_gui_cpp  # for Qt setup
)

# Qt stuff
if("${qt_gui_cpp_USE_QT_MAJOR_VERSION} " STREQUAL "5 ")
  find_package(Qt5Widgets REQUIRED)
  if (Qt5_POSITION_INDEPENDENT_CODE)
    SET(CMAKE_POSITION_INDEPENDENT_CODE ON)
  endif()
  include_directories(SYSTEM "${Qt5Widgets_INCLUDE_DIRS}")
else()
  find_package(Qt4 COMPONENTS QtCore QtGui REQUIRED)
  include(${QT_USE_FILE})
  include_directories(SYSTEM "${QT_INCLUDE_DIR}")
endif()
add_definitions(-DQT_NO_KEYWORDS)

catkin_package(
  INCLUDE_DIRS include
  LIBRARIES ${PROJECT_NAME}
  CATKIN_DEPENDS view_controller_msgs
  )

set(MOC_HEADERS
  include/${PROJECT_NAME}/rviz_animated_view_controller.h
)
if("${qt_gui_cpp_USE_QT_MAJOR_VERSION} " STREQUAL "5 ")
  qt5_wrap_cpp(MOC_FILES ${MOC_HEADERS})
else()
  qt4_wrap_cpp(MOC_FILES ${MOC_HEADERS})
endif()

include_directories(include
                    ${catkin_INCLUDE_DIRS}
                    ${CMAKE_CURRENT_BINARY_DIR})
include_directories(SYSTEM
  ${Boost_INCLUDE_DIRS}
  ${QT_INCLUDE_DIR}
  ${OGRE_INCLUDE_DIRS}
  ${OPENGL_INCLUDE_DIR}
)
add_library(${PROJECT_NAME}
  src/rviz_animated_view_controller.cpp
  ${MOC_FILES})
target_link_libraries(${PROJECT_NAME} ${QT_LIBRARIES} ${OGRE_LIBRARIES}
                                      ${catkin_LIBRARIES} ${Boost_LIBRARIES})
add_dependencies(${PROJECT_NAME} view_controller_msgs_generate_messages_cpp)

install(DIRECTORY include/${PROJECT_NAME}/
        DESTINATION ${CATKIN_PACKAGE_INCLUDE_DESTINATION} PATTERN ".svn" EXCLUDE)

install(FILES plugin_description.xml
        DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION})
install(DIRECTORY launch DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION})

install(TARGETS ${PROJECT_NAME}
   ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
   LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
   RUNTIME DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
 )

