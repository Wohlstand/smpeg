cmake_minimum_required(VERSION 3.2)
project(libsmpeg VERSION 2.0.0 LANGUAGES CXX ASM)

include(${CMAKE_CURRENT_SOURCE_DIR}/audio_codec_common.cmake)

option(BUILD_SMPEG_STATIC "Build static library of SMPEG" ON)
option(BUILD_SMPEG_SHARED "Build shared library of SMPEG" ON)

if(CMAKE_COMPILER_IS_GNUCC OR CMAKE_COMPILER_IS_GNUCXX)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -std=gnu++98 -Wno-pedantic -Wno-sequence-point")
    if(NOT "${CMAKE_CXX_COMPILER_ID}" MATCHES "^(Apple)?Clang$")
        set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wno-unused-but-set-variable")
    endif()
endif()

if(MSVC)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /wd4244 /wd4706 /wd4100 /wd4701 /wd4458 /wd4996 /wd4267 /wd4838 /wd4456 /wd4245")
endif()

add_definitions(
    -DNOCONTROLS
    -DSTDC_HEADERS=1
    -DHAVE_SYS_TYPES_H=1
    -DHAVE_SYS_STAT_H=1
    -DHAVE_STDLIB_H=1
    -DHAVE_STRING_H=1
    -DHAVE_MEMORY_H=1
    -DHAVE_INTTYPES_H=1
    -DHAVE_STDINT_H=1
    -D_THREAD_SAFE
    -DTHREADED_AUDIO
)

IF(NOT WIN32)
    add_definitions(
        -DHAVE_UNISTD_H=1
        -DHAVE_DLFCN_H=1
        -DHAVE_STRINGS_H=1
    )
ELSEIF(NOT MSVC)
    add_definitions(
        -DWIN32
    )
ENDIF()

set(SMPEG_SRC
    src/MPEG.cpp
    src/MPEGlist.cpp
    src/MPEGring.cpp
    src/MPEGstream.cpp
    src/MPEGsystem.cpp
    src/smpeg.cpp
    src/audio/bitwindow.cpp
    src/audio/filter.cpp
    src/audio/filter_2.cpp
    src/audio/hufftable.cpp
    src/audio/MPEGaudio.cpp
    src/audio/mpeglayer1.cpp
    src/audio/mpeglayer2.cpp
    src/audio/mpeglayer3.cpp
    src/audio/mpegtable.cpp
    src/audio/mpegtoraw.cpp
    src/video/decoders.cpp
    src/video/floatdct.cpp
    src/video/gdith.cpp
    src/video/jrevdct.cpp
    src/video/motionvec.cpp
    src/video/MPEGvideo.cpp
    src/video/parseblock.cpp
    src/video/readfile.cpp
    src/video/util.cpp
    src/video/video.cpp
    src/video/mmxflags_asm.S
    src/video/mmxidct_asm.S
)

if(HAIKU)
    find_library(SDL2_LIBRARY SDL2)
    find_path(SDL2_INCLUDE_DIR "SDL.h" PATH_SUFFIXES SDL2)
    if(NOT SDL2_LIBRARY AND NOT SDL2_INCLUDE_DIR)
        message(FATAL_ERROR "The SDL2 Library was not found!")
    endif()
    set(SDL2_INCLUDE_DIRS ${SDL2_INCLUDE_DIR})
    set(SDL2_LIBRARIES ${SDL2_LIBRARY})
else()
    find_package(SDL2 REQUIRED)
    if(TARGET SDL2::SDL2)
        set(SDL2_LIBRARIES SDL2::SDL2main SDL2::SDL2)
    endif()
endif()

message("Found ${SDL2_LIBRARIES}")

if(BUILD_SMPEG_STATIC)
    add_library(smpeg_static STATIC ${SMPEG_SRC})
    set_target_properties(smpeg_static PROPERTIES OUTPUT_NAME smpeg)

    target_include_directories(smpeg_static PRIVATE ${SDL2_INCLUDE_DIRS})
    target_link_libraries(smpeg_static PUBLIC ${SDL2_LIBRARIES})

    include_directories(${CMAKE_CURRENT_SOURCE_DIR}/include/smpeg)
    include_directories(${CMAKE_CURRENT_SOURCE_DIR}/src)

    install(TARGETS smpeg_static
            LIBRARY DESTINATION "lib"
            ARCHIVE DESTINATION "lib"
            INCLUDES DESTINATION "include")
endif()

if(BUILD_SMPEG_SHARED)
    add_library(smpeg_shared SHARED ${SMPEG_SRC})
    set_target_properties(smpeg_shared PROPERTIES OUTPUT_NAME smpeg)

    set_target_properties(smpeg_shared PROPERTIES
        VERSION ${libsmpeg_VERSION}
        SOVERSION ${libsmpeg_VERSION_MAJOR}
    )

    target_include_directories(smpeg_shared PRIVATE ${SDL2_INCLUDE_DIRS})
    target_link_libraries(smpeg_shared PUBLIC ${SDL2_LIBRARIES})

    include_directories(${CMAKE_CURRENT_SOURCE_DIR}/include/smpeg)
    include_directories(${CMAKE_CURRENT_SOURCE_DIR}/src)

    install(TARGETS smpeg_shared
            LIBRARY DESTINATION "lib"
            ARCHIVE DESTINATION "lib"
            INCLUDES DESTINATION "include")
endif()

install(FILES
        include/smpeg/smpeg.h
        include/smpeg/MPEGframe.h
        DESTINATION include/smpeg/)
