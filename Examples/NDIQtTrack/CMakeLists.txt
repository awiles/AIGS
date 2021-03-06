PROJECT(NDIQtTrack)

CMAKE_MINIMUM_REQUIRED(VERSION 2.8.0)

# allow cmake to search for libraries
IF(COMMAND cmake_policy)
  CMAKE_POLICY(SET CMP0003 NEW)
ENDIF(COMMAND cmake_policy)

# default build type
SET( CMAKE_BUILD_TYPE "RelWithDebInfo" )

# find and setup Qt4 for this project
FIND_PACKAGE(Qt REQUIRED)
#Qt Includes:
INCLUDE(${QT_USE_FILE})

# find VTK.
FIND_PACKAGE(CTK)
IF (CTK_USE_FILE)
  INCLUDE (${CTK_USE_FILE})
ELSE (CTK_USE_FILE)
  MESSAGE( FATAL_ERROR "This application requires CTK. One of these components is missing. Please verify configuration")
ENDIF (CTK_USE_FILE)

# find VTK.
FIND_PACKAGE(VTK)
IF (VTK_USE_FILE)
  INCLUDE (${VTK_USE_FILE})
ELSE (VTK_USE_FILE)
  MESSAGE( FATAL_ERROR "This application requires VTK. One of these components is missing. Please verify configuration")
ENDIF (VTK_USE_FILE)

INCLUDE_DIRECTORIES( 
    ${NDIQtTrack_SOURCE_DIR}
    ${NDIQtTrack_BINARY_DIR}
    ${TRACKING_SOURCE_DIR}
    ${vtkTrackerWidget_SOURCE_DIR}/utilities
    ${CTK_INCLUDE_DIRS}
    ${vtkTrackerWidget_SOURCE_DIR} )

LINK_DIRECTORIES( 
	${QT_LIBRARY_DIR}
  ${CTK_LIBRARY_DIR}
)

SET(NDIQtTrack_GUI_SRCS
    NDIQtTrackGUI.ui )
    
SET(NDIQtTrack_HDRS
    NDIQtTrackGUI.h 
    )
    
SET(NDIQtTrack_SRCS
    NDIQtTrack.cxx
    NDIQtTrackGUI.cxx )
    
QT4_WRAP_CPP( NDIQtTrack_GEN_MOC_SRCS ${NDIQtTrack_HDRS})
QT4_WRAP_UI( NDIQtTrack_GEN_UI_SRCS ${NDIQtTrack_GUI_SRCS})
QT4_WRAP_CPP( NDIQtTrack_GEN_MOC_UI_SRCS ${NDIQtTrack_GEN_UI_SRCS})

SET(NDIQtTrack_LIBS
    ${QT_LIBRARIES}
    ${CTK_LIBRARIES}
    QVTK
    vtkTrackerWidget )

IF(AIGS_USE_NDI)
  INCLUDE_DIRECTORIES(${VTKNDICAPI_SOURCE_DIR})
  INCLUDE_DIRECTORIES(${VTKNDICAPI_BINARY_DIR})
  SET(NDIQtTrack_LIBS ${NDIQtTrack_LIBS} vtkndicapi )
ENDIF(AIGS_USE_NDI)

ADD_EXECUTABLE( NDIQtTrack
    ${NDIQtTrack_HDRS}
    ${NDIQtTrack_SRCS}
    ${NDIQtTrack_GEN_UI_SRCS}
    ${NDIQtTrack_GEN_MOC_SRCS}
    ${NDIQtTrack_GEN_MOC_UI_SRCS} )
	
TARGET_LINK_LIBRARIES( NDIQtTrack
    ${NDIQtTrack_LIBS} )
    
IF( WIN32 )
SET_TARGET_PROPERTIES(NDIQtTrack PROPERTIES
  LINK_FLAGS "
    /NODEFAULTLIB:\"libc.lib\"
    /NODEFAULTLIB:\"libcmt.lib\"
    /NODEFAULTLIB:\"libcd.lib\"
    /NODEFAULTLIB:\"libcmtd.lib\" ")
ENDIF( WIN32 )

#--------------------
#INSTALL
#--------------------

# install the executable.
IF(WIN32)
	INSTALL(TARGETS NDIQtTrack RUNTIME DESTINATION bin COMPONENT Examples )
ELSEIF(UNIX)
	INSTALL(TARGETS NDIQtTrack RUNTIME DESTINATION . COMPONENT Examples )
ENDIF()

#Qt Libraries.
FOREACH(LIB QtCore QtGui QtNetwork QtXml QtTest QtSql QtWebKit phonon)
	IF(WIN32)
		INSTALL(FILES "${QT_BINARY_DIR}/${LIB}${QT_VERSION_MAJOR}.dll" DESTINATION bin COMPONENT Examples)
	ELSEIF(UNIX)
		INSTALL(FILES "${QT_LIBRARY_DIR}/lib${LIB}.so.${QT_VERSION_MAJOR}.${QT_VERSION_MINOR}.${QT_VERSION_PATCH}" DESTINATION . COMPONENT Examples)
		INSTALL(FILES "${QT_LIBRARY_DIR}/lib${LIB}.so.${QT_VERSION_MAJOR}.${QT_VERSION_MINOR}" DESTINATION . COMPONENT Examples)
		INSTALL(FILES "${QT_LIBRARY_DIR}/lib${LIB}.so" DESTINATION . COMPONENT Examples)
    ENDIF()
ENDFOREACH(LIB)

#CTK LIBRARIES
#TODO: ADD CTK LIBRARIES to the install.

#VTK Libraries
#Depending on how you compile vtk, you might need some other libraries: vtkParallel VPIC Cosmo
SET( NDIQtTrack_VTK_COMMON_LIBS 
		QVTK vtkalglib vtkCharts vtkCommon vtkDICOMParser vtkexoIIc vtkexpat vtkFiltering vtkfreetype vtkftgl
        vtkGraphics vtkHybrid vtkImaging vtkInfovis vtkIO vtkjpeg vtklibxml2 vtkmetaio vtkNetCDF vtkpng 
        vtkRendering vtksys vtktiff vtkverdict vtkViews vtkWidgets vtkzlib Cosmo VPIC vtkParallel)# vtkGenericFiltering vtksqlite )
        
SET( NDIQtTrack_VTK_COMMON_LIBS_UNVERSIONED vtkNetCDF_cxx )

IF(WIN32)
	FOREACH(LIB ${NDIQtTrack_VTK_COMMON_LIBS})
		INSTALL(FILES "${VTK_RUNTIME_LIBRARY_DIRS}/${LIB}.dll" DESTINATION bin COMPONENT Examples)
	ENDFOREACH(LIB)
    # unversioned libraries
	FOREACH(LIB ${NDIQtTrack_VTK_COMMON_LIBS_UNVERSIONED} )
		INSTALL(FILES "${VTK_RUNTIME_LIBRARY_DIRS}/${LIB}.dll" DESTINATION bin COMPONENT Examples)
	ENDFOREACH(LIB)
ELSEIF( UNIX )
	# versioned libraries.
    FOREACH(LIB ${NDIQtTrack_VTK_COMMON_LIBS} )
		INSTALL(FILES "${VTK_RUNTIME_LIBRARY_DIRS}/lib${LIB}.so.${VTK_VERSION_MAJOR}.${VTK_VERSION_MINOR}.${VTK_VERSION_PATCH}" DESTINATION . COMPONENT NDIQtTrack)
		INSTALL(FILES "${VTK_RUNTIME_LIBRARY_DIRS}/lib${LIB}.so.${VTK_VERSION_MAJOR}.${VTK_VERSION_MINOR}" DESTINATION . COMPONENT NDIQtTrack)	
		INSTALL(FILES "${VTK_RUNTIME_LIBRARY_DIRS}/lib${LIB}.so" DESTINATION . COMPONENT Examples)
	ENDFOREACH(LIB)
	# unversioned libraries
	FOREACH(LIB ${NDIQtTrack_VTK_COMMON_LIBS_UNVERSIONED} )
		INSTALL(FILES "${VTK_RUNTIME_LIBRARY_DIRS}/lib${LIB}.so" DESTINATION . COMPONENT Examples)
	ENDFOREACH(LIB)
ENDIF()

IF(WIN32)
  IF(AIGS_USE_3DG)
    GET_FILENAME_COMPONENT( ATC_LIBRARY_DIRS ${ATC_LIBRARY} PATH )
    GET_FILENAME_COMPONENT( ATC_LIBRARY_NAME_WE ${ATC_LIBRARY} NAME_WE )
    INSTALL( FILES "${ATC_LIBRARY_DIRS}/${ATC_LIBRARY_NAME_WE}.dll" DESTINATION bin COMPONENT Examples)
  ENDIF(AIGS_USE_3DG)
ENDIF(WIN32)
