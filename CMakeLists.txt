cmake_minimum_required(VERSION 3.19)

project(remc2 LANGUAGES C CXX)

include (GNUInstallDirs)
include (FindPkgConfig)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

################################################################################
# Conan setup
################################################################################

#include(${CMAKE_BINARY_DIR}/conanbuildinfo.cmake)
#conan_basic_setup()



################################################################################
# Dependencies
################################################################################

list(APPEND CMAKE_MODULE_PATH ${CMAKE_CURRENT_SOURCE_DIR}/cmake/sdl2)

if (GITHUB_CI_COMPATIBILITY_PATH)
    set(SDL2_DIR "${GITHUB_CI_COMPATIBILITY_PATH}/cmake/SDL2")
    set(CMAKE_LIBRARY_PATH "${GITHUB_CI_COMPATIBILITY_PATH}")
endif ()
pkg_search_module(ZLIB REQUIRED zlib)
pkg_search_module(PNG REQUIRED libpng)
pkg_search_module(PNG2 REQUIRED libpng2)
pkg_search_module(PNG3 REQUIRED libpng3)
pkg_search_module(OGG REQUIRED libogg)
pkg_search_module(VORBIS REQUIRED libvorbis)
pkg_search_module(CDIO REQUIRED libcdio)
pkg_search_module(SDL2 REQUIRED libSDL2)
pkg_search_module(SDL2_mixer REQUIRED libSDL2_mixer)
pkg_search_module(SDL2_ttf REQUIRED libSDL2_ttf)
pkg_search_module(SDL2_image REQUIRED libSDL2_image)
pkg_search_module(SDL_MIXER_X OPTIONAL SDL-Mixer-X)
pkg_search_module(KISS_SDL2 OPTIONAL kiss_sdl2)
pkg_search_module(ADLMIDI REQUIRED libADLMIDI)
pkg_search_module(OPNMIDI OPTIONAL libOPNMIDI)
pkg_search_module(FLUIDSYNTH OPTIONAL fluidsynth)
################################################################################
# Global configuration types
################################################################################
set(CMAKE_CONFIGURATION_TYPES
    "Debug"
    "Release"
    CACHE STRING "" FORCE
)

################################################################################
# Global compiler options
################################################################################
if(MSVC)
    # remove default flags provided with CMake for MSVC
    set(CMAKE_C_FLAGS "")
    set(CMAKE_C_FLAGS_DEBUG "")
    set(CMAKE_C_FLAGS_RELEASE "")
    set(CMAKE_C_FLAGS_XBOX_DEBUG "")
    set(CMAKE_C_FLAGS_XBOX_RELEASE "")
    set(CMAKE_CXX_FLAGS "")
    set(CMAKE_CXX_FLAGS_DEBUG "")
    set(CMAKE_CXX_FLAGS_RELEASE "")
    set(CMAKE_CXX_FLAGS_XBOX_DEBUG "")
    set(CMAKE_CXX_FLAGS_XBOX_RELEASE "")
else()
    #add_compile_options(-fpermissive)
    add_compile_options(-Wno-narrowing)
    if (ALL_WARNINGS)
        add_compile_options(-Wall)
    else ()
        add_compile_options(-Wunused)
        add_compile_options(-Wunused-function)
    endif ()
    if (USE_SANITIZERS)
        add_compile_options(-fsanitize=address)
        add_compile_options(-fsanitize=pointer-compare)
        add_compile_options(-fsanitize=pointer-subtract)
        add_compile_options(-fsanitize=leak)
        add_compile_options(-fsanitize=undefined)
    endif ()
endif()

################################################################################
# Use solution folders feature
################################################################################
set_property(GLOBAL PROPERTY USE_FOLDERS ON)

################################################################################
# Sub-projects
################################################################################
        
# remc2editor

set(SOURCE_FILES_EDITOR
    "editor.cpp"
    "editor_main.cpp"
    "editor.h"
)
foreach(FILE_ENTRY ${SOURCE_FILES_ENGINE} ${SOURCE_FILES__portability} ${SOURCE_FILES__libs__inih})
  list(APPEND SOURCE_FILES_EDITOR "../${FILE_ENTRY}")
endforeach()
source_group("Source Files Editor" FILES ${SOURCE_FILES_EDITOR} )

add_executable(remc2editor FILES ${SOURCE_FILES_EDITOR})

target_compile_options(remc2editor PUBLIC ${SDL2_CFLAGS} ${SDL2_TTF_CFLAGS}
	${SDL2_IMAGE_CFLAGS} ${SDL2_MIXER_LIBRARIES} ${PNG_LIBRARY}
	${FINDFIRST_CFLAGS}
	${ADLMIDI_CFLAGS} ${KISS_SDL2_CFLAGS})

target_link_libraries(remc2editor PUBLIC 
	${SDL2_LIBRARIES}
	${SDL2_TTF_LIBRARIES}
	${SDL2_IMAGE_LIBRARIES}
	${SDL2_MIXER_LIBRARIES}
	${PNG_LIBRARIES}
	${FINDFIRST_LIBRARIES}
    	${ADLMIDI_LIBRARIES}
    	${KISS_SDL2_LIBRARIES}
)

target_include_directories(remc2editor PUBLIC
   "${CMAKE_SOURCE_DIR}/include/inih"
   "${CMAKE_SOURCE_DIR}/include/findfirst"
)

endif ()
