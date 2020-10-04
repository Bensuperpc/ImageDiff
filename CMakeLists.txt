cmake_minimum_required (VERSION 3.13)
project(RPG VERSION 0.0.1 LANGUAGES CXX C)

#   ==================================================
#   ================    CMAKE CONF    ================
#   ==================================================


set(CMAKE_INCLUDE_CURRENT_DIR 1)
set(CMAKE_VERBOSE_MAKEFILE 1)

if(${CMAKE_VERSION} VERSION_GREATER_EQUAL "3.16.0")
    set(CMAKE_UNITY_BUILD 1)
    message(STATUS "CMAKE_UNITY_BUILD : 1")
else()
    set(CMAKE_UNITY_BUILD 0)
    message(STATUS "CMAKE_UNITY_BUILD : 0")
endif()

#set(Var1 "This is variable 1" CACHE INTERNAL "")
#set(Var1 "This is variable 1" PARENT_SCOPE)

#   ================================================
#   ================    COMPILER    ================
#   ================================================

if(NOT CMAKE_C_COMPILER)
#    set(CMAKE_C_COMPILER /usr/bin/clang)
    set(CMAKE_C_COMPILER /usr/bin/gcc)
endif()
if(NOT CMAKE_CXX_COMPILER)
#    set(CMAKE_CXX_COMPILER /usr/bin/clang++)
    set(CMAKE_CXX_COMPILER /usr/bin/g++)
endif()

#=== C++ VERSION CHECK ===
if(NOT CMAKE_CXX_STANDARD)
    include(CheckCXXCompilerFlag)

    CHECK_CXX_COMPILER_FLAG("-std=c++20" COMPILER_SUPPORTS_CXX20)
    CHECK_CXX_COMPILER_FLAG("-std=c++17" COMPILER_SUPPORTS_CXX17)
    CHECK_CXX_COMPILER_FLAG("-std=c++14" COMPILER_SUPPORTS_CXX14)
    CHECK_CXX_COMPILER_FLAG("-std=c++11" COMPILER_SUPPORTS_CXX11)

    if(COMPILER_SUPPORTS_CXX20)
        set(CMAKE_CXX_STANDARD 20)
        message(STATUS "C++20: OK")
    elseif(COMPILER_SUPPORTS_CXX17)
        set(CMAKE_CXX_STANDARD 17)
        message(STATUS "C++17: OK")
    elseif(COMPILER_SUPPORTS_CXX14)
        set(CMAKE_CXX_STANDARD 14)
        message(STATUS "C++14: OK")
    elseif(COMPILER_SUPPORTS_CXX11)
        set(CMAKE_CXX_STANDARD 11)
        message(STATUS "C++11: OK")
        message(WARNING "C++11 is old, please use newer compiler.")
    else()
        message(FATAL_ERROR "The compiler ${CMAKE_CXX_COMPILER} has no C++11 or above support. Please use a different C++ compiler.")
    endif()

    # Compiler-specific C++11 activation.
    if("${CMAKE_CXX_COMPILER_ID}" STREQUAL "GNU")
        # require at least gcc 4.8
        if (CMAKE_CXX_COMPILER_VERSION VERSION_LESS 8.0)
            message(FATAL_ERROR "GCC version must be at least 4.8!")
            set(CMAKE_CXX_STANDARD 14)
        endif()
    elseif ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "Clang")
        # require at least clang 3.2
        if (CMAKE_CXX_COMPILER_VERSION VERSION_LESS 8.0)
            message(FATAL_ERROR "Clang version must be at least 3.2!")
        endif()
    elseif(CMAKE_CXX_COMPILER_ID STREQUAL "MSVC")
        if(CMAKE_CXX_COMPILER_VERSION VERSION_LESS "14.0")
        message(FATAL_ERROR "Insufficient msvc version")
        endif()
    else()
        message(WARNING "You are using an unsupported compiler! Compilation has only been tested with Clang and GCC.")
    endif()
endif()


set(CMAKE_CXX_STANDARD_REQUIRED 1)
if(NOT CMAKE_CXX_EXTENSIONS)
    set(CMAKE_CXX_EXTENSIONS 1)
endif()

#=== OPTIMIZATION FLAGS ===
#-mavx2 -flto
set(OPTI_COMPILER_REL "-march=native -s -ffunction-sections -fdata-sections -Wl,--gc-sections -Wl,--strip-all")

#=== SECURITY RELEASE FLAGS ===

set(SEC_COMPILER_REL "-fstack-protector-all -Werror=format-security -fstack-protector-strong -fexceptions -D_FORTIFY_SOURCE=2 -fPIE")
#For Ubuntu 18.04 and 16.04
CHECK_CXX_COMPILER_FLAG("-fstack-clash-protection" COMPILER_SUPPORTS_SEC_FLAG)
if(NOT COMPILER_SUPPORTS_SEC_FLAG)
    list(APPEND CHECK_CXX_COMPILER_FLAG "/usr/share/SFML/cmake/Modules")
endif()

set(SEC_COMPILER_LINK_REL "-fPIC -Wl,-z,now -Wl,-z,relro")


set(WARNINGS_COMPILER_C "-Wall -Wpedantic -Wextra -Wstrict-prototypes -Wmissing-prototypes -Wfloat-equal -Wundef -Wshadow -Wpointer-arith -Wstrict-overflow=5 -Wswitch-default -Wunreachable-code -Wcast-align")
set(CMAKE_C_FLAGS                "${WARNINGS_COMPILER_C} -std=c99 -pipe")
set(CMAKE_C_FLAGS_RELEASE        "-O3 ${OPTI_COMPILER_REL} ${SEC_COMPILER_REL}")
set(CMAKE_C_FLAGS_MINSIZEREL     "-Os ${OPTI_COMPILER_REL} ${SEC_COMPILER_REL}")
set(CMAKE_C_FLAGS_DEBUG          "-g3 -Og -ggdb3 -v")
set(CMAKE_C_FLAGS_COVERAGE "${CMAKE_C_FLAGS_DEBUG} --coverage")

#-Wold-style-cast
set(WARNINGS_COMPILER_CXX "-Wpedantic -Wall -Wextra -Wshadow -Wstrict-overflow=5 -Wmissing-declarations -Wundef -Wstack-protector -Wno-unused-parameter -Woverloaded-virtual  -Wdisabled-optimization -Winline -Wredundant-decls -Wsign-conversion -Wformat-nonliteral -Wformat-security -Wmissing-include-dirs -Wmissing-noreturn -Wimport -Winit-self -Winvalid-pch -Wstrict-aliasing=2 -Wswitch-default -Wunreachable-code -Wunused -Wwrite-strings -Wcast-align -Wcast-qual -Wfloat-equal -Wvariadic-macros -Wpacked  -Wpadded -Wpointer-arith -Weffc++ -Wformat=2 -Wfloat-equal -Wabi=11")
#-Wdouble-promotion -fstack-usage
set(WARNINGS_COMPILER_CXX "${WARNINGS_COMPILER_CXX} -Wduplicated-branches -Wduplicated-cond -Wduplicated-branches -Wlogical-op -Wnull-dereference -Wuseless-cast")

# Removed : -Wconversion
#-ftime-report -static -lrt -pthread -Wl,--whole-archive -lpthread -Wl,--no-whole-archive -fprofile-arcs -ftest-coverage

#Add -lstdc++fs for old compiler
set(CMAKE_CXX_FLAGS                "${WARNINGS_COMPILER_CXX} -pipe -pthread -lstdc++fs -Wl,-rpath,../lib -Wl,-rpath,../external/lib -pthread")
set(CMAKE_CXX_FLAGS_MINSIZEREL     "-Os ${OPTI_COMPILER_REL} ${SEC_COMPILER_REL} -DNDEBUG")
set(CMAKE_CXX_FLAGS_RELEASE        "-O3 ${OPTI_COMPILER_REL} ${SEC_COMPILER_REL} -DNDEBUG")
set(CMAKE_CXX_FLAGS_DEBUG          "-g3 -Og -ggdb3 -v")
set(CMAKE_CXX_FLAGS_COVERAGE "${CMAKE_CXX_FLAGS_DEBUG} --coverage")

set(CMAKE_SHARED_LINKER_FLAGS_RELEASE "${SEC_COMPILER_LINK_REL}")
set(CMAKE_SHARED_LINKER_FLAGS_MINSIZEREL "${SEC_COMPILER_LINK_REL}")
set(CMAKE_EXE_LINKER_FLAGS_COVERAGE "${CMAKE_EXE_LINKER_FLAGS_DEBUG} --coverage")

set(CMAKE_SHARED_LINKER_FLAGS_COVERAGE "${CMAKE_SHARED_LINKER_FLAGS_DEBUG} --coverage")

#set(BUILD_SHARED_LIBRARIES 0)
#set(CMAKE_EXE_LINKER_FLAGS "-static")

set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)
#set(PROJECT_BINARY_DIR ${CMAKE_RUNTIME_OUTPUT_DIRECTORY})
#set(CTEST_BINARY_DIRECTORY ${CMAKE_BINARY_DIR}/bin)

message(STATUS "Build type: ${CMAKE_BUILD_TYPE}")
#   ================    COMPILER DEF    ================


add_definitions(-DPROJECT_VERSION="${PROJECT_VERSION}")
add_definitions(-DCMAKE_CXX_EXTENSIONS=${CMAKE_CXX_EXTENSIONS})
add_definitions(-DCMAKE_CXX_COMPILER_ID="${CMAKE_CXX_COMPILER_ID}")
add_definitions(-DCMAKE_CXX_COMPILER_VERSION="${CMAKE_CXX_COMPILER_VERSION}")

if(CMAKE_SIZEOF_VOID_P EQUAL 4)
    message(STATUS "32 bit mode")
elseif(CMAKE_SIZEOF_VOID_P EQUAL 8)
    message(STATUS "64 bit mode")
elseif(CMAKE_SIZEOF_VOID_P EQUAL 16)
    message(STATUS "128 bit mode")
else()
    message(STATUS "unknow size")
endif()

#   =================================================
#   ==============    EXTERNAL LIBS    ==============
#   =================================================

include(ExternalProject)
set(EXTERNAL_INSTALL_LOCATION ${CMAKE_BINARY_DIR}/external)


#   ================    OPENCV    ================
find_package(OpenCV REQUIRED QUIET)

if (OPENCV_FOUND)
    INCLUDE_DIRECTORIES( ${OpenCV_INCLUDE_DIRS})
    message(STATUS "OPENCV: FOUND")
else()
    message(STATUS "OPENCV: NOT FOUND")
    if(NOT EXTERNAL_LIB)
        set(EXTERNAL_LIB "")
    endif()
    string(APPEND EXTERNAL_LIB "opencv_lib")
    ExternalProject_Add(opencv_lib
        GIT_REPOSITORY https://github.com/opencv/opencv.git
        GIT_TAG 4.4.0
        CMAKE_ARGS -DCMAKE_INSTALL_PREFIX=${EXTERNAL_INSTALL_LOCATION}
        #CMAKE_ARGS -DCMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE}
        CMAKE_ARGS -DCMAKE_BUILD_TYPE=Release
    )
endif()


#   =================================================
#   =============    INTERNAL LIBS    ===============
#   =================================================

add_subdirectory(lib/utils/opencv "lib/utils/opencv")

if(EXTERNAL_LIB)
    add_dependencies(utils_opencv ${EXTERNAL_LIB})
endif()

#   =================================================
#   =============    SOURCES FILES    ===============
#   =================================================

#   ================    FILES    ================
set(MAIN main.cpp)

set(SRCS
    )

set(HEADERS
    )


#   ====================================================
#   ====================    LIBS    ====================
#   ====================================================

#set(MAIN_LLIBS Qt${QT_VERSION_MAJOR}::Core Qt${QT_VERSION_MAJOR}::Quick Qt${QT_VERSION_MAJOR}::Multimedia sfml-graphics sfml-window sfml-audio sfml-network sfml-system ${Boost_LIBRARIES} ${OPENGL_LIBRARIES} ${GLUT_LIBRARY} ${OpenCV_LIBS})
set(MAIN_LLIBS  ${OpenCV_LIBS})

set(CUSTOM_LIBS utils_opencv)

#   ==================================================
#   ===============    EXEC FILES    =================
#   ==================================================

add_executable(diff ${MAIN} ${SRCS} ${HEADERS})
if(EXTERNAL_LIB)
    add_dependencies(diff ${EXTERNAL_LIB})
endif()

# stdc++fs is needed for Debian
target_link_libraries(diff PRIVATE ${MAIN_LLIBS} ${CUSTOM_LIBS})