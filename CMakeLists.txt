cmake_minimum_required(VERSION 3.19)

project(remc2 LANGUAGES C CXX)

include (GNUInstallDirs)
include (FindPkgConfig)
include(GoogleTest)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

################################################################################
# Dependencies
################################################################################

#include(${CMAKE_BINARY_DIR}/conanbuildinfo.cmake)
#conan_basic_setup()

pkg_search_module(ZLIB REQUIRED zlib)
pkg_search_module(PNG REQUIRED libpng)
#pkg_search_module(OGG REQUIRED libogg)
#pkg_search_module(VORBIS REQUIRED libvorbis)
#pkg_search_module(CDIO REQUIRED libcdio)
#pkg_search_module(SDL2 REQUIRED libSDL2)
#pkg_search_module(SDL2_mixer REQUIRED libSDL2_mixer)
#pkg_search_module(SDL2_ttf REQUIRED libSDL2_ttf)
#pkg_search_module(SDL2_image REQUIRED libSDL2_image)
#pkg_search_module(SDL_MIXER_X OPTIONAL SDL-Mixer-X)
#pkg_search_module(KISS_SDL2 OPTIONAL kiss_sdl2)
pkg_search_module(ADLMIDI REQUIRED libADLMIDI)
pkg_search_module(OPNMIDI REQUIRED libOPNMIDI)
#pkg_search_module(FLUIDSYNTH REQUIRED fluidsynth)
#pkg_search_module(GTEST REQUIRED gtest)

################################################################################
# Global compiler options
################################################################################
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

################################################################################
# Use solution folders feature
################################################################################
set_property(GLOBAL PROPERTY USE_FOLDERS ON)

################################################################################
# Sub-projects
################################################################################
# remc2


################################################################################
# Source groups
################################################################################
#set(HEADER_FILES "")
#source_group("Header Files" FILES ${HEADER_FILES})
set(SOURCE_FILES "src/remc2.cpp")
source_group("Source Files" FILES ${SOURCE_FILES})

set(SOURCE_FILES_ENGINE
    "src/engine/ail_sound.h"
    "src/engine/defs.h"
    "src/engine/engine_support.cpp"
    "src/engine/engine_support.h"
    "src/engine/engine_support_converts.cpp"
    "src/engine/engine_support_converts.h"
    "src/utilities/BitmapIO.cpp"
    "src/utilities/BitmapIO.h"
    "src/utilities/DataFileIO.cpp"
    "src/utilities/DataFileIO.h"
    "src/engine/read_config.cpp"
    "src/engine/read_config.h"
    "src/sub_main.cpp"
    "src/sub_main.h"
    "src/engine/Sound.cpp"
    "src/engine/Sound.h"
    "src/engine/Basic.cpp"
    "src/engine/Basic.h"
    "src/engine/Animation.cpp"
    "src/engine/Animation.h"
    "src/engine/Terrain.cpp"
    "src/engine/Terrain.h"
    "src/engine/sub_main_mouse.h"
)
source_group("Source Files\\engine" FILES ${SOURCE_FILES_ENGINE})

set(SOURCE_FILES__libs__inih
    "libs/inih/ini.c"
    "libs/inih/ini.h"
    "libs/inih/INIReader.cpp"
    "libs/inih/INIReader.h"
)
source_group("Source Files\\libs\\inih" FILES ${SOURCE_FILES__libs__inih})

set(SOURCE_FILES__libs__findfirst
    "libs/findfirst/findfirst.c"
    "libs/findfirst/findfirst.h"
    "libs/findfirst/spec.c"
    "libs/findfirst/spec.h"
)
source_group("Source Files\\libs\\findfirst" FILES ${SOURCE_FILES__libs__findfirst})


set(SOURCE_FILES__portability
    "src/portability/mctypes.h"
    "src/portability/port_filesystem.cpp"
    "src/portability/port_filesystem.h"
    "src/portability/port_inputs.cpp"
    "src/portability/port_inputs.h"
    "src/portability/port_outputs.cpp"
    "src/portability/port_outputs.h"
    "src/portability/port_sdl_sound.cpp"
    "src/portability/port_sdl_sound.h"
    "src/portability/port_sdl_vga_mouse.cpp"
    "src/portability/port_sdl_vga_mouse.h"
    "src/portability/port_time.cpp"
    "src/portability/port_time.h"
    "src/portability/xmi2mid.cpp"
    "src/portability/xmi2mid.h"
)
source_group("Source Files\\portability" FILES ${SOURCE_FILES__portability})

set(ALL_FILES
	#   ${HEADER_FILES}
    ${SOURCE_FILES}
    ${SOURCE_FILES_ENGINE}
    ${SOURCE_FILES__libs__inih}
    ${SOURCE_FILES__portability}
)

################################################################################
# targets
################################################################################
add_library(findfirst ${SOURCE_FILES__libs__findfirst})
add_executable(${PROJECT_NAME} ${ALL_FILES})



target_include_directories(${PROJECT_NAME} PUBLIC
     "${CMAKE_SOURCE_DIR}/libs/inih"
     "${CMAKE_SOURCE_DIR}/libs/findfirst"
)

    #target_compile_definitions(${PROJECT_NAME} PRIVATE
    #

    target_compile_options(${PROJECT_NAME} PUBLIC
	    ${SDL2_CFLAGS}
	    ${SDL2_MIXER_CFLAGS}
	    ${PNG_CFLAGS}
	    ${FINDFIRST_CFLAGS}
	    ${ADLMIDI_CFLAGS}
	    ${OPNMIDI_CFLAGS}
    )

    target_link_libraries(${PROJECT_NAME} PUBLIC
	    ${SDL2_LIBRARIES}
	    ${SDL2_MIXER_LIBRARIES}
	    ${PNG_LIBRARIES}
	    ${FINDFIRST_LIBRARIES}
	    ${ADLMIDI_LIBRARIES}
	    ${OPNMIDI_LIBRARIES}
    )

    target_link_directories(${PROJECT_NAME} PUBLIC
    )


################################################################################
# Unit tests
################################################################################

add_executable(remc2unittests src/UnitTests.cpp)
target_include_directories(remc2unittests PUBLIC
    "${CMAKE_SOURCE_DIR}/libs/findfirst"
)
target_link_libraries(remc2unittests PUBLIC 
    findfirst
    ${GTEST_LIBRARIES}
    ${PTHREAD_LIBRARIES}
)

gtest_add_tests(TARGET      remc2unittests
                TEST_SUFFIX .noArgs
                TEST_LIST   noArgsTests
)
set_tests_properties(${noArgsTests}   PROPERTIES TIMEOUT 10)

# editor

if (USE_SANITIZERS)
	set(SANITIZER_OPTIONS
		-fsanitize=address
		-fsanitize=pointer-compare
		-fsanitize=pointer-subtract
		-fsanitize=leak
		-fsanitize=undefined
		-static-libasan
		-static-liblsan
		-static-libubsan
		)
	target_link_options(remc2 PRIVATE ${SANITIZER_OPTIONS})
	target_link_options(remc2unittests PRIVATE ${SANITIZER_OPTIONS})
	target_link_options(remc2editor PRIVATE ${SANITIZER_OPTIONS})
endif ()


# remc2editor

set(SOURCE_FILES_EDITOR
    "src/editor/editor.cpp"
    "src/editor/editor_main.cpp"
    "src/editor/editor.h"
    ${SOURCE_FILES_ENGINE}
    ${SOURCE_FILES__portability}
    ${SOURCE_FILES__libs__inih}
)

source_group("Source Files Editor" FILES ${SOURCE_FILES_EDITOR} )

add_executable(remc2editor ${SOURCE_FILES_EDITOR})

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
   "${CMAKE_SOURCE_DIR}/libs/inih"
   "${CMAKE_SOURCE_DIR}/libs/findfirst"
)

add_executable(compressTMAPS "tools/compressTMAPS/compressTMAPS.cpp")
add_executable(convertpal "tools/convertpal/convertpal.cpp")
add_executable(decompressTMAPS "tools/decompressTMAPS/decompressTMAPS.cpp")
add_executable(decompresTmaps "tools/decompresTmaps/decompresTmaps.cpp")
add_executable(makeczmc2 "tools/makeczmc2/makeczmc2.cpp")
add_executable(palletelight "tools/palletelight/palletelight.cpp")
add_executable(test_cfs "tools/test_cfs/test_cfs.cpp")
add_executable(xbrz "tools/xbrz/xbrz.cpp")

install(
    TARGETS ${PROJECT_NAME}
)
install(
    FILES 
    ${CMAKE_SOURCE_DIR}/data/font/16x16-font.bmp
    DESTINATION bin/font
)
install(
    FILES 
    ${CMAKE_SOURCE_DIR}/config.ini
    DESTINATION bin
)
install(
    DIRECTORY
    ${CMAKE_SOURCE_DIR}/data/music-ogg
    ${CMAKE_SOURCE_DIR}/data/biggraphics
    DESTINATION bin
)

set(ROOT_NAMESPACE remc2)
