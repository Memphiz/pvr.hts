PROJECT(pvr.hts)

CMAKE_MINIMUM_REQUIRED(VERSION 2.6)

include(AddonVersion.cmake)
addon_version(pvr.hts HTS)

set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} ${PROJECT_SOURCE_DIR})

ENABLE_LANGUAGE(CXX)

OPTION(PACKAGE_ZIP "Package Zip file?" OFF)

IF(WIN32)
  SET(BINDING_FILE ${XBMC_BINDINGS}.zip)
  message (STATUS "downloading XBMC bindings: " ${BINDING_FILE})
  file(DOWNLOAD http://mirrors.xbmc.org/build-deps/win32/${BINDING_FILE} ${CMAKE_BINARY_DIR}/downloads/${BINDING_FILE} STATUS STATUSLIST SHOW_PROGRESS)
  LIST(GET STATUSLIST 0 VALUE)
  IF(${VALUE} STRGREATER "0")
    LIST(GET STATUSLIST 1 VALUE)
    message (STATUS "failed to download XBMC bindings: " ${VALUE})
  ENDIF(${VALUE} STRGREATER "0")
  execute_process(
   COMMAND ${CMAKE_COMMAND} -E tar xzf ${CMAKE_BINARY_DIR}/downloads/${BINDING_FILE}
  )
  FIND_PATH(XBMC_INCLUDE_DIR xbmc/xbmc_pvr_dll.h ${CMAKE_BINARY_DIR}/include)
  INCLUDE_DIRECTORIES(${XBMC_INCLUDE_DIR})
  ADD_DEFINITIONS(-DTARGET_WINDOWS)
ELSE(WIN32)
  FIND_PATH(XBMC_INCLUDE_DIR xbmc/xbmc_pvr_dll.h)
  FIND_PACKAGE(xbmcplatform REQUIRED)
  INCLUDE_DIRECTORIES(${xbmcplatform_INCLUDE_DIRS}
                      ${XBMC_INCLUDE_DIR}
                      ${PROJECT_SOURCE_DIR}/lib)
  ADD_DEFINITIONS(-DTARGET_LINUX -DUSE_DEMUX)
ENDIF(WIN32)

SET(SOURCES src/CircBuffer.cpp
            src/client.cpp
            src/GUIDialogTranscode.cpp
            src/HTSPConnection.cpp
            src/HTSPData.cpp
            src/HTSPDemux.cpp)

ADD_SUBDIRECTORY(lib/libhts)

ADD_LIBRARY(pvr.hts ${SOURCES})
ADD_DEPENDENCIES(pvr.hts hts)

IF(NOT WIN32)
  TARGET_LINK_LIBRARIES(pvr.hts ${xbmcplatform_LIBRARIES})
  set(CMAKE_SHARED_LINKER_FLAGS_RELEASE "-s")
  set(CMAKE_MODULE_LINKER_FLAGS_RELEASE "-s")
ENDIF(NOT WIN32)
SET_TARGET_PROPERTIES(pvr.hts PROPERTIES VERSION ${HTS_VERSION}
                                         SOVERSION 13.0
                                         PREFIX "")

IF(PACKAGE_ZIP)
  # Pack files together to create an archive
  INSTALL(DIRECTORY ${PROJECT_NAME} DESTINATION ./)
  IF(WIN32)
    INSTALL(PROGRAMS ${CMAKE_BINARY_DIR}/pvr.hts.dll DESTINATION pvr.hts)
  ELSE(WIN32)
    INSTALL(TARGETS pvr.hts DESTINATION pvr.hts)
  ENDIF(WIN32)
  SET(CPACK_GENERATOR "ZIP")
  SET(CPACK_INCLUDE_TOPLEVEL_DIRECTORY OFF)
  SET(CPACK_PACKAGE_FILE_NAME ${CMAKE_PROJECT_NAME}-${HTS_VERSION})
  INCLUDE(CPack)
ELSE(PACKAGE_ZIP)
  INSTALL(TARGETS pvr.hts DESTINATION lib/xbmc/addons/pvr.hts)
  INSTALL(DIRECTORY pvr.hts DESTINATION share/xbmc/addons)
ENDIF(PACKAGE_ZIP)
