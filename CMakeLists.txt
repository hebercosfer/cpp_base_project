# This is the top-level CMakeLists.txt file for this CPP Base Project.
#
# Pass the following variables to CMake to control the build:
#
# -DBUILD_TESTING=[true|false]
#  Build the testing tree.
#  Default=true

# -DDEVELOPER_MODE=[true|false]
#  Build the application as developer mode
#  Default=false

# -DENABLE_SANITIZERS=[true|false]
#  Build the application with sanitizer support
#  Default=false

# -DENABLE_BUILD_3RDPARTY=[true|false]
#  Build the 3rd party libraries, usually submodules
#  Default=true

# -DBUILD_COVERAGE=[true|false]
#  Build with coverage support
#  Default=false

# -DGENERATE_DOCS=[true|false]
#  Generate the API documentation.  Enables the 'api_docs' build target.
#  Default=false
cmake_minimum_required(VERSION 3.16)

if(NOT EXISTS ${CMAKE_BINARY_DIR}/CMakeCache.txt)
    set(CMAKE_FIRST_CONFIGURATION TRUE)
else()
    set(CMAKE_FIRST_CONFIGURATION FALSE)
endif()

if(POLICY CMP0003)
    cmake_policy(SET CMP0003 NEW) #linking to libraries with full path
endif()
if(POLICY CMP0020)
    cmake_policy(SET CMP0020 NEW) #static linking to qtmain.lib on Windows
endif()
if(POLICY CMP0048)
    cmake_policy(SET CMP0048 NEW) #project command manages VERSION option
endif()

# Dynamic CMake properties to be used on Naming and Versioning of the project
file(STRINGS cmake.properties ConfigContents)
foreach(name_and_value ${ConfigContents})
    # Strip leading spaces
    string(REGEX REPLACE "^[ ]+" "" name_and_value ${name_and_value})
    # Find variable name
    string(REGEX MATCH "^[^=]+" Name ${name_and_value})
    # Find the value
    string(REPLACE "${Name}=" "" Value ${name_and_value})
    # Set the variable
    set(${Name} "${Value}")
endforeach()

project(
    ${PROJECT_NAME}
    LANGUAGES C CXX
    VERSION ${PROJECT_VERSION}
    DESCRIPTION ${PROJECT_NAME_LONG}
)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
# CMake handle the Qt Objects Moc generation
set(CMAKE_AUTOMOC ON)
# CMake handle the Qt resource file generation
set(CMAKE_AUTORCC ON)

if(APPLE)
    set(CMAKE_MACOSX_RPATH ON)
endif()

if(MSVC)
    set(CMAKE_DEBUG_POSTFIX d)
endif()

# CMake helper for generation of Export Header for Dynamic libs (useful for multi-platform project)
include(GenerateExportHeader)
# Define GNU standard installation directories
include(GNUInstallDirs)

# Helper Cmake files are included on the cmake folder
list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_LIST_DIR}/cmake")

set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)

# Configure where to install the Header files
set(BASEPROJECT_INCLUDE_INSTALL_DIR "${CMAKE_INSTALL_INCLUDEDIR}/CppBaseProject")
set(CMAKECONFIG_INSTALL_DIR "${CMAKE_INSTALL_LIBDIR}/cmake/CppBaseProject")

###############
# Enabling the Qt Library to be used on the project
option(USING_QT6 "Build against Qt6" OFF)

if(USING_QT6)
    set(QT_VERSION_MAJOR 6)
    set(QT_REQUIRED_VERSION "6.3.0")
else()
    set(QT_VERSION_MAJOR 5)
    set(QT_REQUIRED_VERSION "5.15.2")
endif()

find_package(
    Qt${QT_VERSION_MAJOR} ${QT_REQUIRED_VERSION}
    COMPONENTS Widgets
    REQUIRED
)
#########

###############
# Enabling the Boost Library on the project
find_package(Boost REQUIRED)
##########

option(ENABLE_BUILD_3RDPARTY "Build 3rdparty submodules" OFF)

option(DEVELOPER_MODE "Switch on Developer mode" OFF)

option(DEVELOPER_MODE "Build unity tests" OFF)

option(ENABLE_SANITIZERS "Enable sanitizers" OFF)

option(BUILD_COVERAGE "Build the project with gcov support" OFF)

option(UPDATE_TRANSLATIONS "Scan source for new translatable strings" OFF)

option(GENERATE_DOCS "Generate API Documentation system" OFF)

option(QT_FORCE_ASSERTS "Force asserts. Asserts will be enabled regardless of build type configuration." OFF)

option(INSTALL_PYTHON_MODULES "Install the required Python modules using Pip" ON)

if(ENABLE_SANITIZERS)
    if(NOT MSVC)
        set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fsanitize=address,undefined")
    else()
        set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /fsanitize=address")
    endif()
endif()

add_definitions(-DQT_NO_NARROWING_CONVERSIONS_IN_CONNECT)
add_definitions(-DQT_NO_CAST_TO_ASCII)
add_definitions(-DQT_DEPRECATED_WARNINGS_SINCE=0x060000)
add_definitions(-DQT_NO_CAST_FROM_ASCII)
add_definitions(-DQT_DISABLE_DEPRECATED_BEFORE=0x060000)
add_definitions(-DQT_USE_QSTRINGBUILDER)

if(BUILD_COVERAGE)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fprofile-arcs -ftest-coverage")
    set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -lgcov")
endif()

if(QT_FORCE_ASSERTS)
    add_definitions(-DQT_FORCE_ASSERTS)
endif()

if(CMAKE_FIRST_CONFIGURATION AND INSTALL_PYTHON_MODULES)
    find_package(
        Python
        COMPONENTS Interpreter
        REQUIRED
    )

    # detect virtualenv and set Pip args accordingly
    if(DEFINED ENV{VIRTUAL_ENV} OR DEFINED ENV{CONDA_PREFIX})
        set(_pip_args)
    else()
        set(_pip_args "--user")
    endif()

    execute_process(
        COMMAND ${Python_EXECUTABLE} -m pip install ${_pip_args} -r "${CMAKE_CURRENT_LIST_DIR}/requirements-dev.txt"
    )
    # Automate the pre-commit installation
    execute_process(COMMAND pre-commit install WORKING_DIRECTORY ${CMAKE_CURRENT_LIST_DIR})
endif()

#Function that adds the "-Werror" compiler option to the specified target
function(add_werror target)
    if(NOT MSVC)
        if(CMAKE_CXX_COMPILER_ID MATCHES "Clang")
            target_compile_options(${target} PRIVATE -Werror -Winconsistent-missing-override -Wnon-virtual-dtor)
            # -Werror=inconsistent-missing-destructor-override ) we can't activate it
            # by default as KDAB/qitissue/qitissue_style/style.h doesn't have override
            target_compile_options(${target} PRIVATE -Wno-error=range-loop-analysis)
        else()
            target_compile_options(${target} PRIVATE -Werror -Wsuggest-override)
        endif()
        target_compile_options(${target} PRIVATE -Wno-error=deprecated-declarations -Wundef)
        # If we call directly -Wimplicit-fallthrough it's use it as a error ! not a warning
        # we need to tell gcc that we can't want to manage it as error
        # If code is ok we need to add [[fallthrough]]; before code
        if(CMAKE_CXX_COMPILER_ID MATCHES "GNU")
            target_compile_options(${target} PRIVATE -Wno-error=implicit-fallthrough -Wimplicit-fallthrough)
        endif()
    endif()
endfunction()

if(WIN32)
    set(CMAKE_WINDOWS_EXPORT_ALL_SYMBOLS ON)
endif()

## Adding subdirectories
add_subdirectory(src)

if(DEVELOPER_MODE)
    add_definitions(-DDEVELOPER_MODE)
    set(BUILD_TESTING ON)
    add_subdirectory(examples)
endif()

if(ENABLE_BUILD_3RDPARTY)
    add_subdirectory(3rdparty)
endif()

if(BUILD_TESTING)
    enable_testing()
    add_subdirectory(tests)
endif()

if(GENERATE_DOCS)
    add_subdirectory(docs) # needs to go last, in case there are build source files
endif()
