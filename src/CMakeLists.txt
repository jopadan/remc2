project(remc2 C CXX)

include(GNUInstallDirs)
include(FindPkgConfig)
include(GoogleTest)

pkg_search_module(ADLMIDI REQUIRED libadlmidi)
pkg_search_module(SDL2 REQUIRED SDL2::Main)
pkg_search_module(SDL2_MIXER REQUIRED SDL2::Mixer)
pkg_search_module(OPNMIDI OPTIONAL libopnmidi)
pkg_search_module(GTEST OPTIONAL gtest)

# libpodfmt library
add_library( ${PROJECT_NAME} SHARED ${SOURCE_FILES} ${INCLUDE_FILES}
	${CRC_INCLUDE} ${ZIP_INCLUDE})
target_compile_options( ${PROJECT_NAME} PUBLIC ${CRC_CFLAGS} ${ZIP_CFLAGS})
target_include_directories(${PROJECT_NAME} PUBLIC ${CRC_INCLUDEDIR} ${ZIP_INCLUDEDIR}
			$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/src>
			$<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}/lib${PROJECT_NAME}>
			)

################################################################################
# Source groups
################################################################################
set(HEADER_FILES source_group)
source_group("Header Files" FILES ${HEADER_FILES})
set(SOURCE_FILES "remc2.cpp")
source_group("Source Files" FILES ${SOURCE_FILES})

set(SOURCE_FILES_ENGINE
    "engine/ail_sound.h"
    "engine/defs.h"
    "engine/engine_support.cpp"
    "engine/engine_support.h"
    "engine/engine_support_converts.cpp"
    "engine/engine_support_converts.h"
    "utilities/BitmapIO.cpp"
    "utilities/BitmapIO.h"
    "utilities/DataFileIO.cpp"
    "utilities/DataFileIO.h"
    "engine/read_config.cpp"
    "engine/read_config.h"
    "sub_main.cpp"
    "sub_main.h"
    "engine/Sound.cpp"
    "engine/Sound.h"
    "engine/Basic.cpp"
    "engine/Basic.h"
    "engine/Animation.cpp"
    "engine/Animation.h"
    "engine/Terrain.cpp"
    "engine/Terrain.h"
    "engine/sub_main_mouse.h"
)
source_group("Source Files\\engine" FILES ${SOURCE_FILES_ENGINE})

set(SOURCE_FILES__libs__inih
    "../inih/ini.c"
    "../inih/ini.h"
    "../inih/INIReader.cpp"
    "../inih/INIReader.h"
)
source_group("Source Files\\libs\\inih" FILES ${SOURCE_FILES__libs__inih})

set(SOURCE_FILES__libs__findfirst
    "../findfirst/findfirst.c"
    "../findfirst/findfirst.h"
    "../findfirst/spec.c"
    "../findfirst/spec.h"
)
source_group("Source Files\\libs\\findfirst" FILES ${SOURCE_FILES__libs__findfirst})


set(SOURCE_FILES__portability
    "portability/mctypes.h"
    "portability/port_filesystem.cpp"
    "portability/port_filesystem.h"
    "portability/port_inputs.cpp"
    "portability/port_inputs.h"
    "portability/port_outputs.cpp"
    "portability/port_outputs.h"
    "portability/port_sdl_sound.cpp"
    "portability/port_sdl_sound.h"
    "portability/port_sdl_vga_mouse.cpp"
    "portability/port_sdl_vga_mouse.h"
    "portability/port_time.cpp"
    "portability/port_time.h"
    "portability/xmi2mid.cpp"
    "portability/xmi2mid.h"
)
source_group("Source Files\\portability" FILES ${SOURCE_FILES__portability})

set(ALL_FILES
    ${HEADER_FILES}
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

install(
    TARGETS ${PROJECT_NAME}
    CONFIGURATIONS Debug;Release;RelWithDebInfo
    RUNTIME DESTINATION bin
    LIBRARY DESTINATION lib
    ARCHIVE DESTINATION lib/static
)
install(
    FILES 
    ${CMAKE_SOURCE_DIR}/Debug/font/16x16-font.bmp
    DESTINATION bin/font
)
install(
    FILES 
    ${CMAKE_SOURCE_DIR}/config.ini
    DESTINATION bin
)
install(
    DIRECTORY
    ${CMAKE_SOURCE_DIR}/enhancedassets/music-ogg
    ${CMAKE_SOURCE_DIR}/enhancedassets/biggraphics
    DESTINATION bin
)


set(ROOT_NAMESPACE remc2)


################################################################################
# Linux settings
################################################################################
if (UNIX)
    target_include_directories(${PROJECT_NAME} PUBLIC
        "${CMAKE_SOURCE_DIR}/inih"
        "${CMAKE_SOURCE_DIR}/findfirst"
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
endif ()


################################################################################
# Unit tests
################################################################################

add_executable(remc2unittests UnitTests.cpp)
target_include_directories(remc2unittests PUBLIC
    "${CMAKE_SOURCE_DIR}/findfirst"
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
add_subdirectory(editor)


if (UNIX)
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
endif ()
