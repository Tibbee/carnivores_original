cmake_minimum_required(VERSION 3.10)
project(Carnivores1 LANGUAGES CXX)

# Set C++ standard
set(CMAKE_CXX_STANDARD 23)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# Common source files for all configurations
set(SOURCES
    Hunt/Audio.cpp
    Hunt/Characters.cpp
    Hunt/Game.cpp
    Hunt/Hunt.cpp
    Hunt/Interface.cpp
    Hunt/mathematics.cpp
    Hunt/Render3DFX.cpp
    #Hunt/renderasm.cpp //RenderSoft.cpp includes it into its own code
    Hunt/renderd3d.cpp
    Hunt/RenderSoft.cpp
    Hunt/Resources.cpp
    Hunt/resource.rc  # Windows resource file (icons, etc.)
)

set(HEADERS
    Hunt/Audio.h
    Hunt/Hunt.h
    Hunt/resource.h
    Hunt/afxres.h
)

# Set Glide2 specific details for Release3DFX
set(GLIDE2_INCLUDE_DIR "${CMAKE_SOURCE_DIR}/Glide2/include")
set(GLIDE2_LIB_DIR "${CMAKE_SOURCE_DIR}/Glide2/lib")

# Define renderer options
option(BUILD_SOFT "Build for software renderer" OFF)
option(BUILD_D3D "Build for Direct3D renderer" OFF)
option(BUILD_3DFX "Build for 3DFX renderer" OFF)

# Set executable name
set(EXECUTABLE_NAME "HUNT")
if(BUILD_SOFT)
    set(EXECUTABLE_NAME "HUNTSOFT")
elseif(BUILD_D3D)
    set(EXECUTABLE_NAME "HUNTD3D")
elseif(BUILD_3DFX)
    set(EXECUTABLE_NAME "HUNT3DFX")
endif()

# Add executable target
add_executable(${EXECUTABLE_NAME} WIN32)
target_sources(${EXECUTABLE_NAME} PRIVATE ${SOURCES} ${HEADERS})

# Define the post-build step: Copy the executable to your game folder
add_custom_command(TARGET ${EXECUTABLE_NAME} POST_BUILD
    COMMAND ${CMAKE_COMMAND} -E copy_if_different 
        $<TARGET_FILE:${EXECUTABLE_NAME}> 
        "E:/Games/Carnivores1/${EXECUTABLE_NAME}.exe"
        COMMAND ${CMAKE_COMMAND} -E copy_directory
        $<TARGET_FILE_DIR:${EXECUTABLE_NAME}>/
        "E:/Games/Carnivores1/"
    COMMENT "Copying executable and all other files to game folder..."
)

# Set compile definitions based on renderer options
if(BUILD_SOFT)
    target_compile_definitions(${EXECUTABLE_NAME} PRIVATE _soft)
elseif(BUILD_D3D)
    target_compile_definitions(${EXECUTABLE_NAME} PRIVATE _d3d)
elseif(BUILD_3DFX)
    target_compile_definitions(${EXECUTABLE_NAME} PRIVATE _3dfx)
endif()

# Set compiler flags
target_compile_options(${EXECUTABLE_NAME} PRIVATE
    $<$<CONFIG:MinSizeRel>:/GL>
    $<$<CONFIG:Debug>:/DDEBUG>
)

# Set output directories
set_target_properties(${EXECUTABLE_NAME} PROPERTIES
    RUNTIME_OUTPUT_DIRECTORY_DEBUG "${CMAKE_BINARY_DIR}/Debug"
    RUNTIME_OUTPUT_DIRECTORY_RELEASE "${CMAKE_BINARY_DIR}/Release"
    RUNTIME_OUTPUT_DIRECTORY_MINSIZEREL "${CMAKE_BINARY_DIR}/Release"
    ARCHIVE_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/lib"
    LIBRARY_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/lib"
)

# Add Glide2 includes and link libraries for 3DFX build
if(BUILD_3DFX)
    target_include_directories(${EXECUTABLE_NAME} PRIVATE ${GLIDE2_INCLUDE_DIR})
    target_link_directories(${EXECUTABLE_NAME} PRIVATE ${GLIDE2_LIB_DIR})
    target_link_libraries(${EXECUTABLE_NAME} PRIVATE glide2x sst1init)
endif()

# Link system libraries
target_link_libraries(${EXECUTABLE_NAME} PRIVATE dsound ddraw winmm)

# Enable parallel compilation in MSVC
if (MSVC)
    target_compile_options(${EXECUTABLE_NAME} PRIVATE /MP)
endif()

# Install targets
install(TARGETS ${EXECUTABLE_NAME}
    RUNTIME DESTINATION bin
    LIBRARY DESTINATION lib
    ARCHIVE DESTINATION lib
)

