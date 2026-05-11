#
#	MetaCall Library by Parra Studios
#	A library for providing a foreing function interface calls.
#
#	Copyright (C) 2016 - 2026 Vicente Eduardo Ferrer Garcia <vic798@gmail.com>
#
#	Licensed under the Apache License, Version 2.0 (the "License");
#	you may not use this file except in compliance with the License.
#	You may obtain a copy of the License at
#
#		http://www.apache.org/licenses/LICENSE-2.0
#
#	Unless required by applicable law or agreed to in writing, software
#	distributed under the License is distributed on an "AS IS" BASIS,
#	WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
#	See the License for the specific language governing permissions and
#	limitations under the License.
#

#
# CMake options
#

# CMake version
cmake_minimum_required(VERSION 3.15 FATAL_ERROR)

# Include cmake modules
list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake")
set(WriterCompilerDetectionHeaderFound NOTFOUND)

include(ExternalProject)
include(GetGitRevisionDescription)
include(Custom)

# Set policies
set_policy(CMP0028 NEW) 
set_policy(CMP0054 NEW) 
set_policy(CMP0042 NEW) 
set_policy(CMP0063 NEW) 
set_policy(CMP0135 NEW) 

#
# Project description and (meta) information
#
get_git_head_revision(GIT_REFSPEC GIT_SHA1)
string(SUBSTRING "${GIT_SHA1}" 0 12 GIT_REV)

set(META_PROJECT_NAME			"MetaCall")
set(META_PROJECT_DESCRIPTION	"A library for providing inter-language foreign function interface calls")
set(META_AUTHOR_ORGANIZATION	"MetaCall Inc.")
set(META_AUTHOR_DOMAIN			"https://metacall.io/")
set(META_AUTHOR_MAINTAINER		"vic798@gmail.com")

file(READ VERSION META_VERSION)
string(REPLACE "\n" "" META_VERSION ${META_VERSION})
string(REPLACE "." ";" META_VERSION_LIST ${META_VERSION})
list(GET META_VERSION_LIST 0 META_VERSION_MAJOR)
list(GET META_VERSION_LIST 1 META_VERSION_MINOR)
list(GET META_VERSION_LIST 2 META_VERSION_PATCH)
set(META_VERSION_REVISION		"${GIT_REV}")
set(META_NAME_VERSION			"${META_PROJECT_NAME} v${META_VERSION} (${META_VERSION_REVISION})")

#
# Project configuration options
#
option(BUILD_SHARED_LIBS		"Build shared instead of static libraries."					ON) 
option(OPTION_SELF_CONTAINED	"Create a self-contained install with all dependencies."	OFF) 
option(OPTION_BUILD_TESTS		"Build tests."												ON)
option(OPTION_BUILD_BENCHMARKS	"Build benchmarks."											OFF)
option(OPTION_BUILD_DOCS		"Build documentation."										OFF)
option(OPTION_BUILD_EXAMPLES	"Build examples."											ON)
option(OPTION_BUILD_CLI			"Build CLIs."												ON)
option(OPTION_BUILD_LOADERS		"Build loaders."											ON)

# [ОНОВЛЕНО] Використовуємо стандартну назву для C Loader
option(OPTION_BUILD_LOADERS_C   "Build C Foreign Function Interface library loader plugin." ON) 

option(OPTION_BUILD_EXTENSIONS	"Build extensions."											ON)
option(OPTION_BUILD_SCRIPTS		"Build scripts."											ON)
option(OPTION_BUILD_SERIALS		"Build serials."											ON)
option(OPTION_BUILD_DETOURS		"Build detours."											ON)
option(OPTION_BUILD_PORTS		"Build ports."												OFF)
option(OPTION_BUILD_LOG_PRETTY	"Build logs in a human readable format."					ON)
option(OPTION_BUILD_PIC			"Build with position independent code."						ON)
option(OPTION_BUILD_SECURITY	"Build with stack-smashing protection and source fortify."	ON)
option(OPTION_BUILD_GUIX		"Disable all build system unreproductible operations."		OFF)
option(OPTION_GIT_HOOKS			"Disable git hooks when running in CI/CD."					ON)
option(OPTION_FORK_SAFE			"Enable fork safety."										ON)
option(OPTION_THREAD_SAFE		"Enable thread safety."										OFF)
option(OPTION_COVERAGE			"Enable coverage."											OFF)
option(OPTION_MEMORY_TRACKER	"Enable memory tracking for reflect data."					ON)

# Build type & Binary directory...
if(NOT CMAKE_BUILD_TYPE AND NOT CMAKE_CONFIGURATION_TYPES)
	set(CMAKE_BUILD_TYPE "Release" CACHE STRING "Define the type of build." FORCE)
	set_property(CACHE CMAKE_BUILD_TYPE PROPERTY STRINGS "Debug" "Release" "RelWithDebInfo" "MinSizeRel")
endif()

if ("${CMAKE_GENERATOR}" MATCHES "^(Visual Studio)")
	set(PROJECT_OUTPUT_DIR "${CMAKE_BINARY_DIR}/${CMAKE_BUILD_TYPE}")
else()
	set(PROJECT_OUTPUT_DIR "${CMAKE_BINARY_DIR}")
endif()

set_property(GLOBAL PROPERTY USE_FOLDERS ON)
set(IDE_FOLDER "")

project(${META_PROJECT_NAME} C CXX)

set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${PROJECT_BINARY_DIR})
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${PROJECT_BINARY_DIR})
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${PROJECT_BINARY_DIR})

file(WRITE "${PROJECT_BINARY_DIR}/VERSION" "${META_NAME_VERSION}")

include(GenerateExportHeader)

function(GENERATE_EXPORT_HEADER)
	set(target ${ARGV0})
	string(TOUPPER ${target} target_upper)
	set_target_properties(${target}
		PROPERTIES DEFINE_SYMBOL "${target_upper}_EXPORTS"
	)
	_GENERATE_EXPORT_HEADER(${ARGN})
endfunction()

include(Portability)
find_package(ClangFormat)

if(ClangFormat_FOUND)
	include(ClangDevTools)
else()
    message(WARNING "Linting disabled: clang-format executable not found")
endif()

include(CompileOptions)

if(OPTION_COVERAGE)
	if(NOT CMAKE_BUILD_TYPE STREQUAL "Debug")
		message(WARNING "OPTION_COVERAGE requires to be build with -DCMAKE_BUILD_TYPE=Debug, skipping coverage")
		set(OPTION_COVERAGE OFF)
	else()
		include(Coverage)
	endif()
endif()

string(TOLOWER ${META_PROJECT_NAME} project)

set(SYSTEM_DIR_INSTALL FALSE)
if("${CMAKE_INSTALL_PREFIX}" STREQUAL "/usr" OR "${CMAKE_INSTALL_PREFIX}" STREQUAL "/usr/local")
	set(SYSTEM_DIR_INSTALL TRUE)
endif()

if(UNIX AND SYSTEM_DIR_INSTALL OR OPTION_BUILD_GUIX)
	set(INSTALL_ROOT		"share/${project}")			
	set(INSTALL_CMAKE		"share/${project}/cmake")	
	set(INSTALL_EXAMPLES	"share/${project}")			
	set(INSTALL_DATA		"share/${project}")			
	set(INSTALL_BIN			"bin")						
	set(INSTALL_SHARED		"lib")						
	set(INSTALL_LIB			"lib")						
	set(INSTALL_INCLUDE		"include")					
	set(INSTALL_DOC			"share/doc/${project}")		
	set(INSTALL_SHORTCUTS	"share/applications")		
	set(INSTALL_ICONS		"share/pixmaps")			
	set(INSTALL_INIT		"/etc/init")				
else()
	set(INSTALL_ROOT		".")						
	set(INSTALL_CMAKE		"cmake")					
	set(INSTALL_EXAMPLES	".")						
	set(INSTALL_DATA		".")						
	set(INSTALL_BIN			".")						
	set(INSTALL_SHARED		"lib")						
	set(INSTALL_LIB			"lib")						
	set(INSTALL_INCLUDE		"include")					
	set(INSTALL_DOC			"doc")						
	set(INSTALL_SHORTCUTS	"misc")						
	set(INSTALL_ICONS		"misc")						
	set(INSTALL_INIT		"misc")						
endif()

# Set runtime path
set(CMAKE_SKIP_BUILD_RPATH				FALSE)	

# [ВАЖЛИВО] Вмикаємо RPATH під час збірки для портативності дистрибутивів
set(CMAKE_BUILD_WITH_INSTALL_RPATH		TRUE)	
set(CMAKE_INSTALL_RPATH_USE_LINK_PATH	TRUE)	

if(SYSTEM_DIR_INSTALL)
	SET(CMAKE_INSTALL_RPATH "${CMAKE_INSTALL_PREFIX}/${INSTALL_LIB}")
else()
	if(APPLE)
		set(CMAKE_INSTALL_RPATH "@loader_path;@loader_path/${INSTALL_LIB}")
	else()
		set(CMAKE_INSTALL_RPATH "$ORIGIN:$ORIGIN/${INSTALL_LIB}")
	endif()
endif()

set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

if(OPTION_BUILD_TESTS OR OPTION_BUILD_BENCHMARKS)
	enable_testing()
endif()

add_subdirectory(source)
add_subdirectory(docs)
add_subdirectory(deploy)

set(EXPORT_MODULE_NAMES
	version preprocessor environment format threading log memory portability
	adt filesystem reflect dynlink detour serial configuration loader metacall
)

set(PROJECT_CONFIGURATION "metacall-config.cmake")
set(PROJECT_CONFIGURATION_VERSION "metacall-config-version.cmake")
set(PROJECT_CONFIGURATION_PATH "${PROJECT_BINARY_DIR}/${PROJECT_CONFIGURATION}")
set(PROJECT_CONFIGURATION_VERSION_PATH "${PROJECT_BINARY_DIR}/${PROJECT_CONFIGURATION_VERSION}")

configure_file("${CMAKE_SOURCE_DIR}/${PROJECT_CONFIGURATION}.in" "${PROJECT_CONFIGURATION_PATH}" @ONLY)
configure_file("${CMAKE_SOURCE_DIR}/${PROJECT_CONFIGURATION_VERSION}.in" "${PROJECT_CONFIGURATION_VERSION_PATH}" @ONLY)

install(FILES "${PROJECT_BINARY_DIR}/VERSION" DESTINATION ${INSTALL_ROOT} COMPONENT runtime)
install(FILES ${PROJECT_CONFIGURATION_PATH} DESTINATION ${INSTALL_ROOT} COMPONENT dev)
install(FILES ${PROJECT_CONFIGURATION_VERSION_PATH} DESTINATION ${INSTALL_ROOT} COMPONENT dev)
install(FILES AUTHORS	DESTINATION ${INSTALL_ROOT} COMPONENT runtime)
install(FILES LICENSE	DESTINATION ${INSTALL_ROOT} COMPONENT runtime)
install(FILES README.md	DESTINATION ${INSTALL_ROOT} COMPONENT runtime)

if(OPTION_GIT_HOOKS)
	message(STATUS "Installing git hooks at ${CMAKE_CURRENT_SOURCE_DIR}")
	execute_process(
		COMMAND git config --local core.hooksPath githooks/${PROJECT_OS_FAMILY}
		WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
	)
endif()

set(CPACK_PACKAGE_NAME "${META_PROJECT_NAME}")
set(CPACK_PACKAGE_VENDOR "${META_AUTHOR_ORGANIZATION}")
set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "${META_PROJECT_DESCRIPTION}")
set(CPACK_PACKAGE_VERSION_MAJOR "${META_VERSION_MAJOR}")
set(CPACK_PACKAGE_VERSION_MINOR "${META_VERSION_MINOR}")
set(CPACK_PACKAGE_VERSION_PATCH "${META_VERSION_PATCH}")
set(CPACK_PACKAGE_FILE_NAME "${META_PROJECT_NAME}-${META_VERSION}-${CMAKE_SYSTEM_NAME}-${CMAKE_SYSTEM_PROCESSOR}")

if(WIN32)
	set(CPACK_GENERATOR "ZIP")
elseif(APPLE)
	set(CPACK_GENERATOR "TGZ")
else()
	set(CPACK_GENERATOR "TGZ")
endif()

include(CPack)