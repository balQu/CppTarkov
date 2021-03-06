cmake_minimum_required(VERSION 3.2 FATAL_ERROR)
project(CppTarkov CXX)

message(STATUS "CMake version: ${CMAKE_VERSION}")

if(NOT CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE "Debug" CACHE STRING
      "Choose the type of build, options are: Debug Release RelWithDebInfo MinSizeRel." FORCE)
endif(NOT CMAKE_BUILD_TYPE)

message(STATUS "Build type: ${CMAKE_BUILD_TYPE}")

if (MSVC)
	message(STATUS "MSVC: ${MSVC_VERSION}")
endif()

if (CMAKE_SIZEOF_VOID_P EQUAL 4)
	message(FATAL_error "X86 build not supported!")
endif()

if(MSVC)
    set(LIB_RT_SUFFIX "mt")
    set(LIB_RT_OPTION "/MT")

    foreach(flag_var  CMAKE_C_FLAGS  CMAKE_CXX_FLAGS)
         foreach(config_name  ""  DEBUG  RELEASE  MINSIZEREL  RELWITHDEBINFO)
            set(var_name "${flag_var}")

            if(NOT "${config_name}" STREQUAL "")
                set(var_name "${var_name}_${config_name}")
            endif()
            
            string(REPLACE "/MD" "${LIB_RT_OPTION}" ${var_name} "${${var_name}}")
            set(${var_name} "${${var_name}}" CACHE STRING "" FORCE)
        endforeach()
    endforeach()

	set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /MP")
	set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} /MP")

    set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} /SAFESEH:NO /NODEFAULTLIB:libci.lib")
    
    add_definitions(-DNOMINMAX)
    add_definitions(-D_CRT_SECURE_NO_WARNINGS)
endif()

if(${CMAKE_BUILD_TYPE} STREQUAL "Debug")
	set(EXTRA_LIBS cpr-d libcurl-d zlib-d fmt-d spdlog-d)
else()
	set(EXTRA_LIBS cpr libcurl zlib fmt spdlog)
endif()

include_directories(${PROJECT_SOURCE_DIR}/extern/include)
link_directories(${PROJECT_SOURCE_DIR}/extern/lib_x64)

set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ../../Bin)

add_subdirectory(${PROJECT_SOURCE_DIR}/TarkovAPI)
add_subdirectory(${PROJECT_SOURCE_DIR}/Example)
