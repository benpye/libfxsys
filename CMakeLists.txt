cmake_minimum_required (VERSION 2.8)

list (APPEND CMAKE_MODULE_PATH "${PROJECT_SOURCE_DIR}/cmake/Modules")

if (NOT CMAKE_TOOLCHAIN_FILE)
  include (CMakeForceCompiler)
  set (CMAKE_C_FLAGS "-Os -nostdlib -Wall -fomit-frame-pointer -fno-ident")
  if (NOT SH3_TOOLCHAIN_ROOT)
    message (WARNING "No toolchain file was specified, nor was a toolchain installation root. Many assumptions are going to be made about your environment, which may be incorrect.\nIf any errors are encountered, please write a toolchain file and use that (see http://www.cmake.org/Wiki/CMake_Cross_Compiling for instructions)\nAlternatively, specify where your toolchain was installed and we will attempt to find the necessary stuff automatically.\nOne of the assumptions is that you have already set up a gcc toolchain for building for the (ex-Hitachi) Renesas SuperH 3 architecture.\nAnother is that it is located in ~/x-tools/sh3eb-casio-elf-gcc.")
    set (OLD_PATH "$ENV{PATH}")
    set (ENV{PATH} "$ENV{PATH}:$ENV{HOME}/x-tools/sh3eb-casio-elf-gcc/bin")

    set (CMAKE_CROSSCOMPILING TRUE)
    set (CMAKE_SYSTEM_NAME Generic)
    CMAKE_FORCE_C_COMPILER("sh3eb-casio-elf-gcc" "GNU")
  else (NOT SH3_TOOLCHAIN_ROOT)
    if (NOT SH3_TOOLCHAIN_TUPLE)
      message (WARNING "Toolchain tuple not specifying. Guessing sh3eb-casio-elf.")
      set (SH3_TOOLCHAIN_TUPLE "sh3-casio-elf")
    endif (NOT SH3_TOOLCHAIN_TUPLE)
    if (IS_DIRECTORY "${SH3_TOOLCHAIN_ROOT}/bin")
      if (EXISTS "${SH3_TOOLCHAIN_ROOT}/bin/${SH3_TOOLCHAIN_TUPLE}-gcc")
        CMAKE_FORCE_C_COMPILER ("${SH3_TOOLCHAIN_ROOT}/bin/${SH3_TOOLCHAIN_TUPLE}-gcc" "GNU")
      else (EXISTS "${SH3_TOOLCHAIN_ROOT}/bin/${SH3_TOOLCHAIN_TUPLE}-gcc")
        message (FATAL_ERROR "Could not find ${SH3_TOOLCHAIN_ROOT}/bin/${SH3_TOOLCHAIN_TUPLE}-gcc")
      endif (EXISTS "${SH3_TOOLCHAIN_ROOT}/bin/${SH3_TOOLCHAIN_TUPLE}-gcc")
    else (IS_DIRECTORY "${SH3_TOOLCHAIN_ROOT}/bin") 
      message (FATAL_ERROR "${SH3_TOOLCHAIN_ROOT}/bin appears not to be a directory. Please make sure your toolchain is installed correctly and that you have specified the correct path to it.")
    endif (IS_DIRECTORY "${SH3_TOOLCHAIN_ROOT}/bin")
  endif (NOT SH3_TOOLCHAIN_ROOT)
endif (NOT CMAKE_TOOLCHAIN_FILE)


project (libfxsys-ng C)


include_directories (${PROJECT_SOURCE_DIR}/include)

file (GLOB_RECURSE fxsys_SOURCES src/*.c )
file (GLOB_RECURSE fxsys_SYSCALL_SOURCES src/*.S )
set_source_files_properties (${fxsys_SYSCALL_SOURCES} PROPERTIES LANGUAGE C)

add_library (fxsys-ng STATIC ${fxsys_SOURCES} ${fxsys_SYSCALL_SOURCES})
install (TARGETS fxsys-ng
         ARCHIVE DESTINATION lib)
