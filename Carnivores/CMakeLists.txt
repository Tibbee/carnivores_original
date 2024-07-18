cmake_minimum_required(VERSION 3.16.0 FATAL_ERROR)

set(CMAKE_SYSTEM_VERSION 10.0 CACHE STRING "" FORCE)

project(Carnivores1 CXX)

################################################################################
# Set target arch type if empty. Visual studio solution generator provides it.
################################################################################
if(NOT CMAKE_VS_PLATFORM_NAME)
    set(CMAKE_VS_PLATFORM_NAME "x86")
endif()
message("${CMAKE_VS_PLATFORM_NAME} architecture in use")

if(NOT ("${CMAKE_VS_PLATFORM_NAME}" STREQUAL "x64"
     OR "${CMAKE_VS_PLATFORM_NAME}" STREQUAL "x86"))
    message(FATAL_ERROR "${CMAKE_VS_PLATFORM_NAME} arch is not supported!")
endif()

################################################################################
# Global configuration types
################################################################################
set(CMAKE_CONFIGURATION_TYPES
    "Debug"
    "ReleaseD3D"
    "ReleaseSoft"
    "Release3DFX"
    CACHE STRING "" FORCE
)


################################################################################
# Global compiler options
################################################################################
if(MSVC)
    # Remove default flags provided with CMake for MSVC
    set(CMAKE_CXX_FLAGS "")
    set(CMAKE_CXX_FLAGS_DEBUG "")
    set(CMAKE_CXX_FLAGS_RELEASED3D "")
    set(CMAKE_CXX_FLAGS_RELEASESOFT "")
	set(CMAKE_CXX_FLAGS_RELEASE3DFX "")
endif()

################################################################################
# Global linker options
################################################################################
if(MSVC)
    # Remove default flags provided with CMake for MSVC
    set(CMAKE_EXE_LINKER_FLAGS "")
    set(CMAKE_MODULE_LINKER_FLAGS "")
    set(CMAKE_SHARED_LINKER_FLAGS "")
    set(CMAKE_STATIC_LINKER_FLAGS "")
    set(CMAKE_EXE_LINKER_FLAGS_DEBUG "${CMAKE_EXE_LINKER_FLAGS}")
    set(CMAKE_MODULE_LINKER_FLAGS_DEBUG "${CMAKE_MODULE_LINKER_FLAGS}")
    set(CMAKE_SHARED_LINKER_FLAGS_DEBUG "${CMAKE_SHARED_LINKER_FLAGS}")
    set(CMAKE_STATIC_LINKER_FLAGS_DEBUG "${CMAKE_STATIC_LINKER_FLAGS}")
	set(CMAKE_EXE_LINKER_FLAGS_RELEASED3D "${CMAKE_EXE_LINKER_FLAGS}")
    set(CMAKE_MODULE_LINKER_FLAGS_RELEASED3D "${CMAKE_MODULE_LINKER_FLAGS}")
    set(CMAKE_SHARED_LINKER_FLAGS_RELEASED3D "${CMAKE_SHARED_LINKER_FLAGS}")
    set(CMAKE_STATIC_LINKER_FLAGS_RELEASED3D "${CMAKE_STATIC_LINKER_FLAGS}")
	set(CMAKE_EXE_LINKER_FLAGS_RELEASESOFT "${CMAKE_EXE_LINKER_FLAGS}")
    set(CMAKE_MODULE_LINKER_FLAGS_RELEASESOFT "${CMAKE_MODULE_LINKER_FLAGS}")
    set(CMAKE_SHARED_LINKER_FLAGS_RELEASESOFT "${CMAKE_SHARED_LINKER_FLAGS}")
    set(CMAKE_STATIC_LINKER_FLAGS_RELEASESOFT "${CMAKE_STATIC_LINKER_FLAGS}")
	set(CMAKE_EXE_LINKER_FLAGS_RELEASE3DFX "${CMAKE_EXE_LINKER_FLAGS}")
    set(CMAKE_MODULE_LINKER_FLAGS_RELEASE3DFX "${CMAKE_MODULE_LINKER_FLAGS}")
    set(CMAKE_SHARED_LINKER_FLAGS_RELEASE3DFX "${CMAKE_SHARED_LINKER_FLAGS}")
    set(CMAKE_STATIC_LINKER_FLAGS_RELEASE3DFX "${CMAKE_STATIC_LINKER_FLAGS}")
endif()

################################################################################
# Nuget packages function stub.
################################################################################
function(use_package TARGET PACKAGE VERSION)
    message(WARNING "No implementation of use_package. Create yours. "
                    "Package /"${PACKAGE}/" with version /"${VERSION}/" "
                    "for target /"${TARGET}/" is ignored!")
endfunction()

################################################################################
# Common utils
################################################################################
include(CMake/Utils.cmake)

################################################################################
# Additional Global Settings(add specific info there)
################################################################################
include(CMake/GlobalSettingsInclude.cmake OPTIONAL)

################################################################################
# Use solution folders feature
################################################################################
set_property(GLOBAL PROPERTY USE_FOLDERS ON)

################################################################################
# Sub-projects
################################################################################


set(PROJECT_NAME Carnivores1)

################################################################################
# Source groups
################################################################################

source_group("" FILES ${no_group_source_files})

set(Header_Files
    "Hunt/Audio.h"
    "Hunt/Hunt.h"
    "Hunt/resource.h"
)
source_group("Header Files" FILES ${Header_Files})

set(Source_Files
    "Hunt/Audio.cpp"
    "Hunt/Characters.cpp"
    "Hunt/Game.cpp"
    "Hunt/Hunt.cpp"
    "Hunt/Interface.cpp"
    "Hunt/mathematics.cpp"
    "Hunt/Render3DFX.cpp"
    #"Hunt/renderasm.cpp"
    "Hunt/renderd3d.cpp"
    "Hunt/RenderSoft.cpp"
    "Hunt/Resources.cpp"
	"Hunt/Resource.rc"
)
source_group("Source Files" FILES ${Source_Files})

set(ALL_FILES
    ${Header_Files}
    ${Source_Files}
)

################################################################################
# Target
################################################################################
add_executable(${PROJECT_NAME} ${ALL_FILES})

use_props(${PROJECT_NAME} "${CMAKE_CONFIGURATION_TYPES}" "${DEFAULT_CXX_PROPS}")
set_target_properties(${PROJECT_NAME} PROPERTIES
    VS_GLOBAL_KEYWORD "Win32Proj"
)

# Include directories for 3dfx
################################################################################
if("${CMAKE_VS_PLATFORM_NAME}" STREQUAL "x86")
    target_include_directories(${PROJECT_NAME} PRIVATE
        $<$<CONFIG:Release3DFX>:${CMAKE_SOURCE_DIR}/Glide2/include>
    )
endif()

################################################################################
# Link directories and libraries for 3dfx
################################################################################
if("${CMAKE_VS_PLATFORM_NAME}" STREQUAL "x86")
    target_link_directories(${PROJECT_NAME} PRIVATE
        $<$<CONFIG:Release3DFX>:${CMAKE_SOURCE_DIR}/Glide2/lib>
    )
    target_link_libraries(${PROJECT_NAME} PRIVATE
        $<$<CONFIG:Release3DFX>:glide2x;sst1init>
    )
endif()

################################################################################
# Target name
################################################################################
if("${CMAKE_VS_PLATFORM_NAME}" STREQUAL "x86")
    set_target_properties(${PROJECT_NAME} PROPERTIES
        TARGET_NAME_DEBUG       "HUNT_DEBUG"
        TARGET_NAME_RELEASED3D  "HUNT_D3D"
        TARGET_NAME_RELEASESOFT "HUNT_SOFT"
        TARGET_NAME_RELEASE3DFX "HUNT_3DFX"
    )
endif()
################################################################################
# MSVC runtime library
################################################################################
get_property(MSVC_RUNTIME_LIBRARY_DEFAULT TARGET ${PROJECT_NAME} PROPERTY MSVC_RUNTIME_LIBRARY)
if("${CMAKE_VS_PLATFORM_NAME}" STREQUAL "x86")
    string(CONCAT "MSVC_RUNTIME_LIBRARY_STR"
        $<$<CONFIG:Debug>:
            MultiThreadedDebugDLL
        >
        $<$<CONFIG:ReleaseD3D>:
            MultiThreaded
        >
        $<$<CONFIG:Release3DFX>:
            MultiThreaded
        >
        $<$<CONFIG:ReleaseSoft>:
            MultiThreaded
        >
        $<$<NOT:$<OR:$<CONFIG:Debug>,$<CONFIG:Release3DFX>,$<CONFIG:ReleaseD3D>,$<CONFIG:ReleaseSoft>>>:${MSVC_RUNTIME_LIBRARY_DEFAULT}>
    )
endif()
set_target_properties(${PROJECT_NAME} PROPERTIES MSVC_RUNTIME_LIBRARY ${MSVC_RUNTIME_LIBRARY_STR})

################################################################################
# Compile definitions
################################################################################
if("${CMAKE_VS_PLATFORM_NAME}" STREQUAL "x86")
    target_compile_definitions(${PROJECT_NAME} PRIVATE
        "$<$<CONFIG:Debug>:"
            "_soft;"
            "_DEBUG"
        ">"
        "$<$<CONFIG:ReleaseD3D>:"
            "_d3d;"
            "NDEBUG;"
            "_MBCS"
        ">"
        "$<$<CONFIG:Release3DFX>:"
            "_3dfx;"
            "NDEBUG;"
            "_MBCS"
        ">"
        "$<$<CONFIG:ReleaseSoft>:"
            "_soft;"
            "NDEBUG;"
            "_MBCS"
        ">"
        "WIN32;"
        "_WINDOWS"
    )
endif()

################################################################################
# Compile and link options
################################################################################
if(MSVC)
    if("${CMAKE_VS_PLATFORM_NAME}" STREQUAL "x64")
        target_compile_options(${PROJECT_NAME} PRIVATE
            ${DEFAULT_CXX_DEBUG_INFORMATION_FORMAT};
            ${DEFAULT_CXX_EXCEPTION_HANDLING}
        )
    elseif("${CMAKE_VS_PLATFORM_NAME}" STREQUAL "x86")
        target_compile_options(${PROJECT_NAME} PRIVATE
            $<$<CONFIG:Debug>:
                /Od;
                /arch:SSE;
                /Zi
            >
            $<$<CONFIG:ReleaseD3D>:
                /GF;
				/O2;
				/GL;
				/Ob2;
				/Oy;
                ${DEFAULT_CXX_DEBUG_INFORMATION_FORMAT}
            >
            $<$<CONFIG:Release3DFX>:
                /GF;
				/O2;
				/GL;
				/Ob2;
				/Oy;
                ${DEFAULT_CXX_DEBUG_INFORMATION_FORMAT}
            >
            $<$<CONFIG:ReleaseSoft>:
                /GF;
                ${DEFAULT_CXX_DEBUG_INFORMATION_FORMAT}
            >
            /W3;
            ${DEFAULT_CXX_EXCEPTION_HANDLING}
        )
    endif()
    if("${CMAKE_VS_PLATFORM_NAME}" STREQUAL "x64")
        target_link_options(${PROJECT_NAME} PRIVATE
            /DEBUG:FULL
        )
    elseif("${CMAKE_VS_PLATFORM_NAME}" STREQUAL "x86")
        target_link_options(${PROJECT_NAME} PRIVATE
            $<$<CONFIG:Debug>:
                /DEBUG;
                /INCREMENTAL
            >
            $<$<CONFIG:ReleaseD3D>:
                /OPT:REF;
                /OPT:ICF;
				/LTCG;
                /INCREMENTAL:NO
            >
            $<$<CONFIG:Release3DFX>:
                /OPT:REF;
                /OPT:ICF;
				/LTCG;
                /INCREMENTAL:NO
            >
            $<$<CONFIG:ReleaseSoft>:
                /OPT:REF;
                /OPT:ICF;
                /INCREMENTAL:NO
            >
            /MACHINE:X86;
            /SUBSYSTEM:WINDOWS
        )
    endif()
endif()

################################################################################
# Dependencies
################################################################################
if("${CMAKE_VS_PLATFORM_NAME}" STREQUAL "x86")
    set(ADDITIONAL_LIBRARY_DEPENDENCIES
        "ddraw;"
        "dsound;"
        "winmm"
    )
endif()
target_link_libraries(${PROJECT_NAME} PRIVATE "${ADDITIONAL_LIBRARY_DEPENDENCIES}")

if(CMAKE_BUILD_TYPE)
    message(STATUS "Current build type: ${CMAKE_BUILD_TYPE}")
else()
    message(STATUS "No build type specified. Defaulting to Debug.")
    set(CMAKE_BUILD_TYPE Debug)
endif()
