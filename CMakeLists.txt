cmake_minimum_required (VERSION 3.0)
project(dinghy VERSION 0.1.0 LANGUAGES C)

include(GNUInstallDirs)

option(USE_WEBKIT_GTK "Use WebKitGTK+ instead of WPEWebKit" OFF)
option(BUILD_SHARED_LIBS "Build lib${PROJECT_NAME}core as a shared library" ON)

set(DEFAULT_APPID "" CACHE STRING "Default GApplication unique identifier")
if (DEFAULT_APPID AND NOT DEFAULT_APPID STREQUAL "")
	add_definitions(-DDY_LAUNCHER_DEFAULT_APPID="${DEFAULT_APPID}")
endif ()

if (BUILD_SHARED_LIBS)
	set(DINGHYCORE_COMPONENT "runtime")
else()
	set(DINGHYCORE_COMPONENT "development")
endif()

add_definitions(
	-DPROJECT_VERSION="${PROJECT_VERSION}"
	-DPROJECT_VERSION_MAJOR=${PROJECT_VERSION_MAJOR}
	-DPROJECT_VERSION_MINOR=${PROJECT_VERSION_MINOR}
	-DPROJECT_VERSION_PATCH=${PROJECT_VERSION_PATCH}
)

find_package(PkgConfig)

set(DINGHYCORE_EXTRA_SOURCES "")
if (USE_WEBKIT_GTK)
	pkg_search_module(WKGTK REQUIRED webkit2gtk-4.0)
	set(WEB_ENGINE_INCLUDE_DIRS ${WKGTK_INCLUDE_DIRS})
	set(WEB_ENGINE_LIBRARIES ${WKGTK_LIBRARIES})
	list(APPEND DINGHYCORE_EXTRA_SOURCES dy-gtk-utils.c)
	add_definitions(-DDY_WEBKIT_GTK=1)
else ()
	pkg_search_module(WPE REQUIRED wpe-webkit)
	set(WEB_ENGINE_INCLUDE_DIRS ${WPE_INCLUDE_DIRS})
	set(WEB_ENGINE_LIBRARIES ${WPE_LIBRARIES})
endif ()


# libdinghycore
set(DINGHYCORE_API_HEADERS
	dinghy.h
	dy-launcher.h
	dy-webkit-utils.h
)
add_library(dinghycore
	dy-launcher.c
	${DINGHYCORE_EXTRA_SOURCES}
)
set_property(TARGET dinghycore PROPERTY C_STANDARD 99)
target_include_directories(dinghycore PUBLIC ${WEB_ENGINE_INCLUDE_DIRS})
target_link_libraries(dinghycore ${WEB_ENGINE_LIBRARIES})

add_executable(dinghy dinghy.c)
set_property(TARGET dinghy PROPERTY C_STANDARD 99)
target_link_libraries(dinghy dinghycore)
		
install(TARGETS dinghy
	DESTINATION ${CMAKE_INSTALL_BINDIR}
	COMPONENT "runtime"
)
install(TARGETS dinghycore
	DESTINATION ${CMAKE_INSTALL_LIBDIR}
	COMPONENT ${DINGHYCORE_COMPONENT}
)
install(FILES ${DINGHYCORE_API_HEADERS}
	DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/dinghy
	COMPONENT "development"
)