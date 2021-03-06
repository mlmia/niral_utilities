project(IntensityRescaler)

cmake_minimum_required(VERSION 2.8)
cmake_policy(VERSION 2.8)

find_package(ITK REQUIRED)
if(ITK_FOUND)
   include(${ITK_USE_FILE})
else()
   message(FATAL_ERROR, "ITK not found. Please set ITK_DIR.")
endif()

find_package(VTK REQUIRED)
if(VTK_FOUND)
  include(${VTK_USE_FILE})
else()
  message(FATAL_ERROR "VTK not found. Please set VTK_DIR.")
endif()

#include(CMake/qtSetup.cmake)
find_package( Qt REQUIRED )
#Find_Package( QT )
include(${QT_USE_FILE})

option(COMPILE_IntensityRescaler_GUI "Qt IntensityRescaler GUI support." OFF)

include_directories(
  ${QT_INCLUDE_DIR}
  ${GLUT_INCLUDE_DIR}
  Libraries/Display/Plot
  Libraries/Datatypes
  Libraries/IO/Qt
  Libraries/IO/Common
  Libraries/Algorithms
  Libraries/Display/Common
  Libraries/Display/Window2D
  ${CMAKE_CURRENT_SOURCE_DIR}
  ${CMAKE_CURRENT_BINARY_DIR}
  )

if( COMPILE_IntensityRescaler_GUI )

  if(WIN32)
    #set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} /subsystem:windows")
  endif()
  #Find_package( GLUT )
  #Find_package( opengl )
  include(CMake/glutSetup.cmake)

  include(${CMAKE_ROOT}/Modules/FindOpenGL.cmake)
  include(${CMAKE_ROOT}/Modules/FindGLU.cmake)

  set(IntensityRescaler_MOC_SRCS
     IntensityRescalerGUIControls.h
  )
  if(QT_WRAP_CPP)
    qt_wrap_cpp(IntensityRescaler IntensityRescalerMoced_SRCS ${IntensityRescaler_MOC_SRCS})
  endif()
  set( IntensityRescaler_GUI_CXX_SRCS 
    IntensityRescalerGUIControls.cxx
    BatchControls.cxx
  )
  set(IntensityRescaler_GUI_SRCS
    Batch.ui
    HistoGUI.ui
    IntensityRescalerGUI.ui
  )
  if(QT_WRAP_UI)
	  qt_wrap_ui(IntensityRescaler IntensityRescaler_HDRS  IntensityRescaler_SRCS  ${IntensityRescaler_GUI_SRCS} )
  endif()
  add_definitions(-DQT_DLL)
  add_definitions(-DQT_THREAD_SUPPORT)
  add_definitions( -DQT_GUI )

  set(OPEN_GL_LIBRARIES
      ${QT_GL_LIBRARY}
      ${OPENGL_glu_LIBRARY}
      ${OPENGL_LIBRARY}
      ${GLUT_LIBRARIES}
     )
else()
  add_definitions( -DQT_NO_GUI )
endif()
set(IntensityRescaler_SRCS
  ${IntensityRescalerMoced_SRCS}
  ${IntensityRescaler_GUI_CXX_SRCS}
  IntensityRescaler.cxx
  IntensityRescalerCommandLine.cxx
)

subdirs(Libraries)

add_executable(IntensityRescaler ${IntensityRescaler_SRCS})

target_link_libraries(IntensityRescaler
  Algorithms
  Display
  ${ITK_LIBRARIES}
  ${QT_QT_LIBRARY}
  ${QT_LIBRARIES}
  ${QT_QTMAIN_LIBRARY}
  ${OPEN_GL_LIBRARIES}
  ${VTK_LIBRARIES}
  vtkFiltering
 )

install( TARGETS IntensityRescaler RUNTIME DESTINATION bin )

