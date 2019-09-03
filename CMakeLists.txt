project(VolumeLibrary Fortran)
cmake_minimum_required(VERSION 3.11)
enable_language (Fortran)

# Read the source file names as a list
#file(STRINGS "${CMAKE_CURRENT_SOURCE_DIR}/_sources.txt" vollib_sources)
file(GLOB vollib_sources .. *.f)

if (TARGET_32BIT)
    set(arch_flags "-m32")
    if (NOT DEFINED vollib)
        set (vollib vollib)
    endif ()
else ()
    set(arch_flags "-m64")
    if (NOT DEFINED vollib)
        set (vollib vollib64)
    endif ()
endif ()

if (NATIVE_ARCH)
    message(STATUS "Targeting host native architecture.")
    set(arch_flags "-march=native")
endif ()

set(CMAKE_C_FLAGS "${arch_flags} ${CMAKE_C_FLAGS}")
set(CMAKE_CXX_FLAGS "${arch_flags} ${CMAKE_CXX_FLAGS}")
set(CMAKE_Fortran_FLAGS "${arch_flags} ${CMAKE_Fortran_FLAGS}")
set(CMAKE_SHARED_LINKER_FLAGS "${arch_flags} ${CMAKE_SHARED_LINKER_FLAGS}")

if (NOT CMAKE_SYSTEM_NAME STREQUAL "Windows")
    set(CMAKE_C_FLAGS "-fPIC ${CMAKE_C_FLAGS}")
    set(CMAKE_CXX_FLAGS "-fPIC ${CMAKE_CXX_FLAGS}")
    set(CMAKE_Fortran_FLAGS "-fPIC ${CMAKE_Fortran_FLAGS}")
    set(CMAKE_SHARED_LINKER_FLAGS "-fPIC ${CMAKE_SHARED_LINKER_FLAGS}")
endif ()

if (CMAKE_SYSTEM_NAME STREQUAL "Windows")
    # TODO: Add STATIC_LINK option
    set(CMAKE_SHARED_LINKER_FLAGS "-static ${CMAKE_SHARED_LINKER_FLAGS} -s -Wl,--subsystem,windows")
endif ()

# -DCMAKE_BUILD_TYPE=Debug
set(debug_flags "-g3 -mfpmath=sse -msse")
set(CMAKE_Fortran_FLAGS_DEBUG "${debug_flags} -fbacktrace -ffpe-trap=invalid,zero,overflow,underflow")
set(CMAKE_CXX_FLAGS_DEBUG "${debug_flags}")
set(CMAKE_C_FLAGS_DEBUG "${debug_flags}")

# Compile sources into objects
add_library(${vollib}_objs OBJECT ${vollib_sources})

# Create a static library
add_library(${vollib}_static STATIC $<TARGET_OBJECTS:${vollib}_objs>)
target_sources(${vollib}_static PRIVATE) # FIXME: Workaround for bug with object library builds
set_target_properties(${vollib}_static PROPERTIES
    LINKER_LANGUAGE Fortran
    )
set(DEPS ${vollib}_static)

# Link a shared library
add_library(${vollib} SHARED $<TARGET_OBJECTS:${vollib}_objs>)
target_sources(${vollib} PRIVATE) # FIXME: Workaround for bug with object library builds
set_target_properties(${vollib} PROPERTIES
    LINKER_LANGUAGE Fortran
    )
set(DEPS ${DEPS} ${vollib})

set(OUTPUT "${CMAKE_CURRENT_SOURCE_DIR}/timestamp")
add_custom_command(
        OUTPUT ${OUTPUT}
        COMMAND ${CMAKE_COMMAND} -E touch ${OUTPUT}
        DEPENDS ${DEPS}
        WORKING_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}"
        )
add_custom_target(vollib_timestamp ALL DEPENDS ${OUTPUT})
