cmake_minimum_required(VERSION 3.7)
cmake_policy(VERSION 3.7)

option(SSDEEP_DISABLE_POSITION_ARRAY
    "Disable bit-parallel string operations." OFF)
option(SSDEEP_INSTALL_HEADERS "Install public header files" ON)

project(ssdeep VERSION 2.14.1)

include(CheckIncludeFile)
include(CheckFunctionExists)

check_include_file(dirent.h HAVE_DIRENT_H)
check_include_file(dlfcn.h HAVE_DLFCN_H)
check_include_file(fcntl.h HAVE_FCNTL_H)
check_include_file(inttypes.h HAVE_INTTYPES_H)
check_include_file(libgen.h HAVE_LIBGEN_H)
check_include_file(memory.h HAVE_MEMORY_H)
check_include_file(stdint.h HAVE_STDINT_H)
check_include_file(stdlib.h HAVE_STDLIB_H)
check_include_file(strings.h HAVE_STRINGS_H)
check_include_file(string.h HAVE_STRING_H)
check_include_file(sys/disk.h HAVE_SYS_DISK_H)
check_include_file(sys/ioctl.h HAVE_SYS_IOCTL_H)
check_include_file(sys/mount.h HAVE_SYS_MOUNT_H)
check_include_file(sys/param.h HAVE_SYS_PARAM_H)
check_include_file(sys/stat.h HAVE_SYS_STAT_H)
check_include_file(sys/types.h HAVE_SYS_TYPES_H)
check_include_file(unistd.h HAVE_UNISTD_H)
check_include_file(wchar.h HAVE_WCHAR_H)

check_function_exists(ftello.h HAVE_FSEEKO)

configure_file(config.h.in.cmake ${CMAKE_CURRENT_BINARY_DIR}/config.h)


add_library(fuzzy
    edit_dist.h
    edit_dist.c
    find-file-size.c
    fuzzy.h
    fuzzy.c
)

target_compile_definitions(fuzzy PRIVATE -DHAVE_CONFIG_H)

target_include_directories(fuzzy
    PRIVATE
        ${CMAKE_CURRENT_SOURCE_DIR}
        ${CMAKE_CURRENT_BINARY_DIR}
)

if(MSVC)
    target_compile_definitions(fuzzy PRIVATE -D_CRT_SECURE_NO_WARNINGS)
    target_compile_options(fuzzy PRIVATE $<$<CONFIG:RELEASE>:/guard:cf>)
endif()

if(INSTALL_HEADERS)
    set_target_properties(fuzzy PROPERTIES PUBLIC_HEADER "ssdeep.h")
endif()

install(
    TARGETS fuzzy
    EXPORT fuzzy
    ARCHIVE DESTINATION lib
    LIBRARY DESTINATION lib
    PUBLIC_HEADER DESTINATION include
)

install(
    EXPORT fuzzy
    DESTINATION share/ssdeep
    FILE fuzzyConfig.cmake
    NAMESPACE ssdeep::
    CONFIGURATIONS Release
)


add_executable(ssdeep
    cycles.cpp
    dig.cpp
    edit_dist.h
    engine.cpp
    filedata.h
    filedata.cpp
    find-file-size.c
    fuzzy.h
    helpers.cpp
    main.h
    main.cpp
    match.h
    match.cpp
    ui.cpp
    ssdeep.h
    tchar-local.h
)

target_compile_definitions(ssdeep
    PRIVATE
        -DHAVE_CONFIG_H
        -DVERSION="${PROJECT_VERSION}"
)

target_include_directories(ssdeep
    PRIVATE
        ${CMAKE_CURRENT_SOURCE_DIR}
        ${CMAKE_CURRENT_BINARY_DIR}
)

target_link_libraries(ssdeep PRIVATE fuzzy)

if(MSVC)
    target_compile_definitions(ssdeep PRIVATE -D_CRT_SECURE_NO_WARNINGS)
    target_compile_options(ssdeep PRIVATE $<$<CONFIG:RELEASE>:/guard:cf>)
endif()
