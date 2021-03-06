PROJECT( companion )

SET( C9X_VERSION_MAJOR "2" )
SET( C9X_VERSION_MINOR "0" )
SET( C9X_VERSION_REVISION "14" )
SET( C9X_VERSION ${C9X_VERSION_MAJOR}.${C9X_VERSION_MINOR}.${C9X_VERSION_REVISION} )

CMAKE_MINIMUM_REQUIRED( VERSION 2.8 )

SET(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} ${CMAKE_CURRENT_SOURCE_DIR}/cmake)

MESSAGE( STATUS "Looking for XercesC " )
FIND_PACKAGE(XercesC)
IF ( XERCESC_FOUND )
    INCLUDE_DIRECTORIES( ${XERCESC_INCLUDE_DIR} )
    SET(LidarFormat_LIBRAIRIES ${LidarFormat_LIBRAIRIES} ${XERCESC_LIBRARIES})
ELSE()
    MESSAGE( FATAL_ERROR "XercesC not found! Please set XercesC path ..." )
ENDIF()

MESSAGE( STATUS "Looking for Xsd " )
FIND_PACKAGE(Xsd)
IF ( XSD_FOUND )
    INCLUDE_DIRECTORIES( ${XSD_INCLUDE_DIR} )
    MESSAGE("XSD Include Path: " ${XSD_INCLUDE_DIR} )
ELSE()
    MESSAGE( FATAL_ERROR "XSD not found! Please set XSD path ..." )
ENDIF()

MESSAGE( STATUS "Looking for SDL " )
FIND_PACKAGE(Sdl)
IF ( SDL_FOUND )
    INCLUDE_DIRECTORIES( ${SDL_INCLUDE_DIR} )
    MESSAGE("SDL Include Path: " ${SDL_INCLUDE_DIR} )
    ADD_DEFINITIONS( -DJOYSTICKS )
ELSE()
    MESSAGE( "SDL not found! Joysticks support will be disabled" )
ENDIF()

FIND_PACKAGE(Qt4 4.8.4 REQUIRED)
SET( QT_USE_QTNETWORK True )
SET( QT_USE_QTXML True )
INCLUDE( ${QT_USE_FILE} )

MESSAGE( STATUS "Looking for PHONON " )
FIND_PACKAGE(Phonon)
IF ( PHONON_FOUND )
    INCLUDE_DIRECTORIES( ${PHONO_INCLUDE_DIR} )
    MESSAGE("PHONON Include Path: " ${PHONON_INCLUDE_DIR} )
    ADD_DEFINITIONS( -DPHONON )
ELSE()
    MESSAGE( "PHONON not found!" )
ENDIF()

ADD_CUSTOM_COMMAND(
  COMMAND ${XSDCXX_EXECUTABLE} cxx-tree --root-element-all --generate-serialization --output-dir ${PROJECT_BINARY_DIR}/ ${PROJECT_SOURCE_DIR}/radio.xsd
  OUTPUT ${PROJECT_BINARY_DIR}/radio.hxx ${PROJECT_BINARY_DIR}/radio.cxx
  DEPENDS ${PROJECT_SOURCE_DIR}/radio.xsd
)

#if(EXT EQUAL FRSKY)
#add_definitions(-DEXT=FRSKY)
#endif() 

#IF(EXT EQUAL MAVLINK)
#add_definitions(-DEXT=MAVLINK)
#MESSAGE("EXT EQUAL MAVLINK--------------")
#endif() 

add_custom_command(OUTPUT ${PROJECT_SOURCE_DIR}/custom.h 
				   COMMAND echo "#define MAVLINK" > ${PROJECT_SOURCE_DIR}/custom.h)
#set_source_files_properties(./opentxSky9xsimulator.cpp
#    PROPERTIES COMPILE_DEFINITIONS MAVLINK=1;)

IF( MSVC )
  SET(CMAKE_CXX_FLAGS "/EHsc /LD")
  INCLUDE_DIRECTORIES( ${PROJECT_SOURCE_DIR}/winbuild C:/Programs/pthreads/Pre-built.2/include C:/Programs/msinttypes)
  LINK_DIRECTORIES( C:/Programs/pthreads/Pre-built.2/lib/ )
  SET( PTHREAD_LIBRARY pthreadVC2.lib )
ELSE( )
  SET( PTHREAD_LIBRARY pthread )
  ADD_DEFINITIONS( -Wall )
  LINK_DIRECTORIES( /usr/local/lib )
ENDIF( )

set(RADIO_SRC_DIRECTORY ${PROJECT_SOURCE_DIR}/../../radio/src)
set(SIMU_SRC_DIRECTORY ${PROJECT_SOURCE_DIR}/simulation)
set(COMPANION_SRC_DIRECTORY ${PROJECT_SOURCE_DIR})

add_custom_command(
  OUTPUT ${RADIO_SRC_DIRECTORY}/bitmaps/sticks.lbm
  COMMAND make lbm
  DEPENDS ${RADIO_SRC_DIRECTORY}/bitmaps/sticks_4x1.xbm
  WORKING_DIRECTORY ${RADIO_SRC_DIRECTORY}
)

add_custom_command(
  OUTPUT ${RADIO_SRC_DIRECTORY}/translations/en.h
  COMMAND make tra
  DEPENDS ${RADIO_SRC_DIRECTORY}/translations/en.h.txt
  WORKING_DIRECTORY ${RADIO_SRC_DIRECTORY}
)

add_custom_command(
  OUTPUT ${RADIO_SRC_DIRECTORY}/lua_exports.inc
  COMMAND make PCB=TARANIS lua_exports.cpp
  DEPENDS ${RADIO_SRC_DIRECTORY}/myeeprom.h
  WORKING_DIRECTORY ${RADIO_SRC_DIRECTORY}
)

include_directories(
  ${CMAKE_CURRENT_BINARY_DIR}
  ${PROJECT_SOURCE_DIR}/../..
  ${SIMU_SRC_DIRECTORY}
  ${RADIO_SRC_DIRECTORY}
  ${RADIO_SRC_DIRECTORY}/fonts/std
  ${RADIO_SRC_DIRECTORY}/lua/src
  ${XSD_INCLUDE_DIR}
)

add_subdirectory(modeledit)
add_subdirectory(simulation)

set(common_SRCS
  eeprominterface.cpp
  firmwares/th9x/th9xeeprom.cpp # TODO not needed
  firmwares/th9x/th9xinterface.cpp
  firmwares/er9x/er9xeeprom.cpp
  firmwares/er9x/er9xinterface.cpp
  firmwares/gruvin9x/gruvin9xeeprom.cpp
  firmwares/gruvin9x/gruvin9xinterface.cpp  
  firmwares/ersky9x/ersky9xeeprom.cpp
  firmwares/ersky9x/ersky9xinterface.cpp
  ${RADIO_SRC_DIRECTORY}/bitmaps/sticks.lbm
  ${RADIO_SRC_DIRECTORY}/translations/en.h
  ${RADIO_SRC_DIRECTORY}/lua_exports.inc
  firmwares/opentx/opentxeeprom.cpp
  firmwares/opentx/open9xStockeeprom.cpp
  firmwares/opentx/open9xGruvin9xeeprom.cpp
  firmwares/opentx/open9xSky9xeeprom.cpp
  firmwares/opentx/opentxinterface.cpp
  firmwares/opentx/opentxTaranisSimulator.cpp
  firmwares/opentx/opentxSky9xsimulator.cpp
  firmwares/opentx/opentxGruvin9xsimulator.cpp
  firmwares/opentx/opentxM128simulator.cpp
  firmwares/opentx/opentxM64simulator.cpp
  file.cpp
  appdata.cpp
)

qt4_wrap_cpp(common_SRCS)

add_library(common ${common_SRCS})

set(companion_SRCS
  hexinterface.cpp
  flashinterface.cpp
  # xmlinterface.cpp
  # ${PROJECT_BINARY_DIR}/radio.cxx
  helpers.cpp
  mdichild.cpp
  generaledit.cpp
  modelslist.cpp
  mountlist.cpp 
  avroutputdialog.cpp
  apppreferencesdialog.cpp
  fwpreferencesdialog.cpp
  burnconfigdialog.cpp
  comparedialog.cpp
  contributorsdialog.cpp
  customizesplashdialog.cpp
  burndialog.cpp
  printdialog.cpp
  fusesdialog.cpp
  logsdialog.cpp
  downloaddialog.cpp
  splashlibrary.cpp
  mainwindow.cpp
  companion.cpp
  radionotfound.cpp
  qcustomplot.cpp
  wizarddata.cpp
  wizarddialog.cpp
)

set(companion_MOC_HDRS
  avroutputdialog.h
  apppreferencesdialog.h
  fwpreferencesdialog.h
  burnconfigdialog.h
  comparedialog.h
  printdialog.h
  fusesdialog.h
  logsdialog.h
  contributorsdialog.h
  customizesplashdialog.h
  splashlibrary.h
  splashlabel.h
  burndialog.h
  downloaddialog.h
  generaledit.h
  modelslist.h
  mdichild.h
  mainwindow.h
  radionotfound.h
  qcustomplot.h
  qautocombobox.h
  helpers.h
  wizarddialog.h
)
     
set(companion_UIS
  mdichild.ui
  avroutputdialog.ui
  comparedialog.ui
  fusesdialog.ui
  logsdialog.ui
  apppreferencesdialog.ui
  fwpreferencesdialog.ui
  burnconfigdialog.ui
  downloaddialog.ui
  generaledit.ui
  printdialog.ui
  contributorsdialog.ui
  customizesplashdialog.ui
  splashlibrary.ui
  burndialog.ui
  radionotfound.ui
)

set(companion_RCS
  companion.qrc
  ${CMAKE_CURRENT_BINARY_DIR}/translations.qrc
)

MACRO (TODAY RESULT)
    IF (WIN32)
        EXECUTE_PROCESS(COMMAND "cmd" " /C date /T" OUTPUT_VARIABLE ${RESULT})
        string(REGEX REPLACE "(..)/(..)/(....).*" "\\1.\\2.\\3" ${RESULT} ${${RESULT}})
    ELSEIF(UNIX)
        EXECUTE_PROCESS(COMMAND "date" "+%d/%m/%Y" OUTPUT_VARIABLE ${RESULT})
        string(REGEX REPLACE "(..)/(..)/(....).*" "\\1.\\2.\\3" ${RESULT} ${${RESULT}})
    ELSE (WIN32)
        MESSAGE(SEND_ERROR "date not implemented")
        SET(${RESULT} 00.00.0000)
    ENDIF (WIN32)
ENDMACRO (TODAY)

MACRO (NOW RESULT)
    IF (WIN32)
        EXECUTE_PROCESS(COMMAND "cmd" " /C time /T" OUTPUT_VARIABLE ${RESULT})
        string(REGEX REPLACE "(..):(..).*" "\\1:\\2:00" ${RESULT} ${${RESULT}})
    ELSEIF(UNIX)
        EXECUTE_PROCESS(COMMAND "date" "+%H:%M:%S" OUTPUT_VARIABLE ${RESULT})
        string(REGEX REPLACE "(..):(..):(..).*" "\\1:\\2:\\3" ${RESULT} ${${RESULT}})
    ELSE (WIN32)
        MESSAGE(SEND_ERROR "time not implemented")
        SET(${RESULT} 00:00:00)
    ENDIF (WIN32)
ENDMACRO (NOW)

TODAY(DATE)
NOW(TIME)

CONFIGURE_FILE(${CMAKE_CURRENT_SOURCE_DIR}/version.h.in ${CMAKE_CURRENT_BINARY_DIR}/version.h @ONLY)
CONFIGURE_FILE(${CMAKE_CURRENT_SOURCE_DIR}/translations.qrc.in ${CMAKE_CURRENT_BINARY_DIR}/translations.qrc @ONLY)
CONFIGURE_FILE(${CMAKE_CURRENT_SOURCE_DIR}/firmwares/opentx/stamp-opentx.h.in ${CMAKE_CURRENT_BINARY_DIR}/stamp-opentx.h @ONLY)

IF( WIN32 )
  SET( companion_SRCS ${companion_SRCS} icon.rc )
  IF ( NOT MSVC )
    SET(CMAKE_EXE_LINKER_FLAGS -mwindows)
  ENDIF( )
ENDIF( )

INCLUDE_DIRECTORIES( ${CMAKE_BINARY_DIR} )
INCLUDE_DIRECTORIES( ${CMAKE_SOURCE_DIR} )

SET( LANGUAGES he pl pt ru de fr it sv cs fi nl)
FOREACH( language ${LANGUAGES} )
  SET( companion_TS ${companion_TS} translations/companion_${language}.ts )
ENDFOREACH( language )

set( companion_SRCS ${companion_SRCS}  )
QT4_WRAP_UI( companion_SRCS ${companion_UIS} )
QT4_WRAP_CPP( companion_SRCS ${companion_MOC_HDRS} )
QT4_ADD_TRANSLATION( companion_QM ${companion_TS}  )
QT4_ADD_RESOURCES( companion_SRCS ${companion_RCS} )

ADD_DEFINITIONS( -DQT_TRANSLATIONS_DIR="${QT_TRANSLATIONS_DIR}" )

IF( APPLE )
    ADD_EXECUTABLE( ${PROJECT_NAME} MACOSX_BUNDLE ${companion_SRCS} ${companion_QM} )
    # OS X compiler no longer automatically includes /Library/Frameworks in search path
    SET( CMAKE_EXE_LINKER_FLAGS -F/Library/Frameworks )
ELSE( )
    ADD_EXECUTABLE( ${PROJECT_NAME} WIN32 ${companion_SRCS} ${companion_QM} )
ENDIF( )

TARGET_LINK_LIBRARIES( ${PROJECT_NAME} modeledit simulation common ${QT_LIBRARIES} ${QT_QTMAIN_LIBRARY} ${XERCESC_LIBRARY} ${PTHREAD_LIBRARY} ${SDL_LIBRARY} ${PHONON_LIBS} )

############# Standalone simu ###############

set(TH9X_CHECKOUT_DIRECTORY ${PROJECT_BINARY_DIR}/firmwares/th9x)
add_custom_command(
  OUTPUT ${TH9X_CHECKOUT_DIRECTORY}/th9xsimulator.cpp
  COMMAND ${CMAKE_COMMAND} -E make_directory ${TH9X_CHECKOUT_DIRECTORY}
  COMMAND cd ${TH9X_CHECKOUT_DIRECTORY} && svn checkout http://th9x.googlecode.com/svn/trunk/src@285 .
  COMMAND ${CMAKE_COMMAND} -E copy ${CMAKE_CURRENT_SOURCE_DIR}/firmwares/th9x/th9xsimulator.cpp ${TH9X_CHECKOUT_DIRECTORY}
  DEPENDS ${CMAKE_CURRENT_SOURCE_DIR}/firmwares/th9x/th9xsimulator.cpp
)

set(ER9X_CHECKOUT_DIRECTORY ${PROJECT_BINARY_DIR}/firmwares/er9x)
add_custom_command(
  OUTPUT ${ER9X_CHECKOUT_DIRECTORY}/er9xsimulator.cpp
  COMMAND ${CMAKE_COMMAND} -E make_directory ${ER9X_CHECKOUT_DIRECTORY}
  COMMAND cd ${ER9X_CHECKOUT_DIRECTORY} && svn checkout http://er9x.googlecode.com/svn/trunk/src@790 .
  COMMAND ${CMAKE_COMMAND} -E copy ${CMAKE_CURRENT_SOURCE_DIR}/firmwares/er9x/er9xsimulator.cpp ${ER9X_CHECKOUT_DIRECTORY}
  DEPENDS ${CMAKE_CURRENT_SOURCE_DIR}/firmwares/er9x/er9xsimulator.cpp
)

set(ERSKY9X_CHECKOUT_DIRECTORY ${PROJECT_BINARY_DIR}/firmwares/ersky9x)
add_custom_command(
  OUTPUT ${ERSKY9X_CHECKOUT_DIRECTORY}/ersky9xsimulator.cpp
  COMMAND ${CMAKE_COMMAND} -E make_directory ${ERSKY9X_CHECKOUT_DIRECTORY}
  COMMAND cd ${ERSKY9X_CHECKOUT_DIRECTORY} && svn checkout http://ersky9x.googlecode.com/svn/trunk/src@170 .
  COMMAND ${CMAKE_COMMAND} -E copy ${CMAKE_CURRENT_SOURCE_DIR}/firmwares/ersky9x/ersky9xsimulator.cpp ${ERSKY9X_CHECKOUT_DIRECTORY}
  DEPENDS ${CMAKE_CURRENT_SOURCE_DIR}/firmwares/ersky9x/ersky9xsimulator.cpp
)

set(simu_SRCS
  ${TH9X_CHECKOUT_DIRECTORY}/th9xsimulator.cpp
  ${ER9X_CHECKOUT_DIRECTORY}/er9xsimulator.cpp
  ${ERSKY9X_CHECKOUT_DIRECTORY}/ersky9xsimulator.cpp
  ${RADIO_SRC_DIRECTORY}/bitmaps/sticks.lbm
  ${RADIO_SRC_DIRECTORY}/translations/en.h
  ${RADIO_SRC_DIRECTORY}/lua_exports.inc
  modeledit/node.cpp
  modeledit/edge.cpp # TODO not needed
  simulator.cpp
)

set(simu_MOC_HDRS
  modeledit/node.h
)

set( simu_SRCS ${simu_SRCS} )
qt4_wrap_cpp(simu_SRCS ${simu_MOC_HDRS} )
qt4_add_resources(simu_SRCS companion.qrc) # TODO not needed

if(WIN32)
  set(simu_SRCS ${simu_SRCS} icon.rc)
endif()

if(APPLE)
  add_executable(simulator MACOSX_BUNDLE ${simu_SRCS})
else()
  add_executable(simulator WIN32 ${simu_SRCS})
endif()

if(SDL_FOUND)
  target_link_libraries(simulator simulation common ${QT_LIBRARIES} ${PTHREAD_LIBRARY} ${SDL_LIBRARY} ${PHONON_LIBS})
else()
  target_link_libraries(simulator simulation common ${QT_LIBRARIES} ${QT_QTMAIN_LIBRARY} ${PTHREAD_LIBRARY} ${SDL_LIBRARY} ${PHONON_LIBS})
endif()

############# Packaging ####################

IF(${CMAKE_SYSTEM_NAME} MATCHES "Linux")
    INSTALL( TARGETS ${PROJECT_NAME} DESTINATION ${CMAKE_INSTALL_PREFIX}/bin )
    INSTALL( FILES companion.desktop DESTINATION share/applications )
    INSTALL( FILES images/linuxicons/16x16/companion.png    DESTINATION /usr/share/icons/hicolor/16x16/apps )
    INSTALL( FILES images/linuxicons/22x22/companion.png    DESTINATION /usr/share/icons/hicolor/22x22/apps )
    INSTALL( FILES images/linuxicons/24x24/companion.png    DESTINATION /usr/share/icons/hicolor/24x24/apps )
    INSTALL( FILES images/linuxicons/32x32/companion.png    DESTINATION /usr/share/icons/hicolor/32x32/apps )
    INSTALL( FILES images/linuxicons/48x48/companion.png    DESTINATION /usr/share/icons/hicolor/48x48/apps )
    INSTALL( FILES images/linuxicons/128x128/companion.png  DESTINATION /usr/share/icons/hicolor/128x128/apps )
    INSTALL( FILES images/linuxicons/256x256/companion.png  DESTINATION /usr/share/icons/hicolor/256x256/apps )
    INSTALL( FILES images/linuxicons/512x512/companion.png  DESTINATION /usr/share/icons/hicolor/512x512/apps )
    INSTALL( FILES images/linuxicons/scalable/companion.svg DESTINATION /usr/share/icons/hicolor/scalable/apps )
    INSTALL( FILES ../targets/linux/45-companion-taranis.rules DESTINATION /etc/udev/rules.d/ )
    # Linux specific code
    SET(OperatingSystem "Linux")
ELSE (${CMAKE_SYSTEM_NAME} MATCHES "Linux")
    INSTALL( TARGETS ${PROJECT_NAME} DESTINATION ${QT_BINARY_DIR} )
ENDIF(${CMAKE_SYSTEM_NAME} MATCHES "Linux")

IF( WIN32 )
  IF( MSVC )
    SET( NSI_IN_FILE companion-vs.nsi.in )  
  ELSE( )
    SET( NSI_IN_FILE companion-msys.nsi.in )
  ENDIF( )

  SET( SYSDIR "$ENV{windir}/system32" )  
  CONFIGURE_FILE(${CMAKE_CURRENT_SOURCE_DIR}/../targets/windows/${NSI_IN_FILE} ${CMAKE_CURRENT_BINARY_DIR}/companion.nsi @ONLY)     

  FIND_PROGRAM(NSIS_EXE makensis.exe PATHS
    "C:/Program Files/NSIS"
    "C:/Program Files (x86)/NSIS"    
    "C:/Programs/NSIS"
  )
  
  ADD_CUSTOM_COMMAND(OUTPUT ${PROJECT_BINARY_DIR}/companionInstall.exe
                     COMMAND "${NSIS_EXE}" ARGS companion.nsi
                     DEPENDS ${PROJECT_NAME} simulator ${PROJECT_BINARY_DIR}/companion.nsi
                     COMMENT "Windows NSIS Installer")

  ADD_CUSTOM_TARGET( installer
    DEPENDS ${PROJECT_BINARY_DIR}/companionInstall.exe
    SOURCES ${PROJECT_BINARY_DIR}/companion.nsi
  )
ENDIF( )

IF( APPLE )
    SET(MACOSX_BUNDLE_ICON_FILE iconmac.icns)
  IF( CMAKE_GENERATOR STREQUAL Xcode )
    SET( WORK_DIR ${CMAKE_BINARY_DIR}/Debug )
  ELSE( )
    SET( WORK_DIR ${PROJECT_BINARY_DIR} )
  ENDIF()
  
  ADD_CUSTOM_TARGET( addframeworks ALL 
                     COMMAND mkdir -p companion.app/Contents/Frameworks
                     COMMAND mkdir -p companion.app/Contents/Resources
                     COMMAND mkdir -p companion.app/Contents/translations                     
                     COMMAND rsync ${PROJECT_SOURCE_DIR}/images/${MACOSX_BUNDLE_ICON_FILE} companion.app/Contents/Resources/${MACOSX_BUNDLE_ICON_FILE} # Copy the icon files
                     COMMAND rsync -r --exclude=.svn ${PROJECT_SOURCE_DIR}/../targets/mac/qt_menu.nib companion.app/Contents/Resources/
                     COMMAND rsync -r /Library/Frameworks/SDL.framework companion.app/Contents/Frameworks/
                     COMMAND rsync ${PROJECT_BINARY_DIR}/*.qm companion.app/Contents/translations
                     COMMAND ${QT_BINARY_DIR}/macdeployqt companion.app
                     COMMAND install_name_tool -change @rpath/SDL.framework/Versions/A/SDL @executable_path/../Frameworks/SDL.framework/Versions/A/SDL companion.app/Contents/MacOS/companion
                     WORKING_DIRECTORY ${WORK_DIR}
                     DEPENDS ${PROJECT_NAME} )
                        
  ADD_CUSTOM_TARGET( makedmg
                     COMMAND ${QT_BINARY_DIR}/macdeployqt companion.app -dmg -no-plugins
                     WORKING_DIRECTORY ${WORK_DIR}
                     DEPENDS addframeworks )

ENDIF( )

ADD_CUSTOM_TARGET( stamp
                   COMMAND ${CMAKE_COMMAND} -E copy ${CMAKE_BINARY_DIR}/version.h ${CMAKE_SOURCE_DIR}/../companion.stamp 
                   DEPENDS ${CMAKE_BINARY_DIR}/version.h )

IF(WIN32)
  ADD_CUSTOM_TARGET( translations
                   WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}
                   COMMAND lupdate ${CMAKE_SOURCE_DIR} -ts ${companion_TS} )
ELSE( )
  ADD_CUSTOM_TARGET( translations
                   WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}
                   COMMAND lupdate-qt4 ${CMAKE_SOURCE_DIR} -ts ${companion_TS} )
ENDIF( )

SET(CPACK_PACKAGE_NAME "COMPANION")
SET(CPACK_PACKAGE_DESCRIPTION_SUMMARY "Models and settings editor for the OpenTx open source firmware")
string(TOLOWER "${CPACK_PACKAGE_NAME}" CPACK_PACKAGE_NAME_LOWERCASE)
find_program(DPKG_PROGRAM dpkg DOC "dpkg program of Debian-based systems")
if(DPKG_PROGRAM)
  SET(CPACK_GENERATOR "DEB")
  execute_process(
    COMMAND ${DPKG_PROGRAM} --print-architecture
    OUTPUT_VARIABLE CPACK_DEBIAN_PACKAGE_ARCHITECTURE
    OUTPUT_STRIP_TRAILING_WHITESPACE
  )
  SET(CPACK_PACKAGE_FILE_NAME "${CPACK_PACKAGE_NAME_LOWERCASE}_${C9X_VERSION}_${CPACK_DEBIAN_PACKAGE_ARCHITECTURE}")
  SET(CPACK_DEBIAN_PACKAGE_MAINTAINER "Bertrand Songis <bsongis-nosp@m-gmail.com>")
  SET(CPACK_DEBIAN_PACKAGE_VERSION ${C9X_VERSION})
  SET(CPACK_DEBIAN_PACKAGE_VERSION_MAJOR ${C9X_VERSION_MAJOR})
  SET(CPACK_DEBIAN_PACKAGE_VERSION_MINOR ${C9X_VERSION_MINOR})
  SET(CPACK_DEBIAN_ARCHITECTURE $(CMAKE_SYSTEM_PROCESSOR))
  SET(CPACK_DEBIAN_PACKAGE_SHLIBDEPS ON)
else(DPKG_PROGRAM)
  SET(CPACK_GENERATOR "RPM")
  SET(CPACK_RPM_PACKAGE_VERSION ${C9X_VERSION})
  SET(CPACK_RPM_PACKAGE_DESCRIPTION ${CPACK_PACKAGE_DESCRIPTION_SUMMARY})
  SET(CPACK_PACKAGE_FILE_NAME "${CPACK_PACKAGE_NAME_LOWERCASE}-${C9X_VERSION}-${CMAKE_SYSTEM_PROCESSOR}")
endif(DPKG_PROGRAM) 

INCLUDE(CPack)
