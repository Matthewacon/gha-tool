cmake_minimum_required(VERSION 3.25)

project(
 gha-tool
 VERSION 0.0.0
 LANGUAGES C
 DESCRIPTION "Small freestanding binary to interact with the GitHub and GitHub Actions APIs"
)

#NOTE: only meaningful when building with GNU Make
set(CMAKE_VERBOSE_MAKEFILE TRUE)

#[[setup project base]]
include("${CMAKE_CURRENT_LIST_DIR}/dependencies/cakemake/CMakeLists.txt")

#[[build flags]]
set(GHA_TOOL_DEBUG FALSE CACHE BOOL "Enable debug flags for gha-tool")
if(GHA_TOOL_DEBUG)
 set(GHA_TOOL_NOT_DEBUG FALSE)
else()
 set(GHA_TOOL_NOT_DEBUG TRUE)
endif()
set(GHA_TOOL_STATIC ${GHA_TOOL_NOT_DEBUG} CACHE BOOL "Enable flags for building a static executable")
set(GHA_TOOL_MUSL_LIBC ${GHA_TOOL_NOT_DEBUG} CACHE BOOL "Enable flags for linking against musl libc")
unset(GHA_TOOL_NOT_DEBUG)
set(GHA_TOOL_SANITIZERS ${GHA_TOOL_DEBUG} CACHE BOOL "Enable flags for building with sanitizers")

#[[add all compiler and linker flags to global lists]]
gha_tool_add_default_flags()
gha_tool_add_debug_flags(GHA_TOOL_DEBUG)
gha_tool_add_sanitizer_flags(GHA_TOOL_SANITIZERS)
gha_tool_add_static_flags(GHA_TOOL_STATIC)
gha_tool_add_musl_libc_flags(GHA_TOOL_MUSL_LIBC)
#TODO: gha_tool_add_test_flags(GHA_TOOL_TESTS)

#TODO: DEBUG, REMOVE
message(STATUS "COMPILE: ${GHA_TOOL_COMPILER_FLAGS}")
message(STATUS "COMPILE PUBLIC: ${GHA_TOOL_PUBLIC_COMPILER_FLAGS}")
message(STATUS "LINK: ${GHA_TOOL_LINKER_FLAGS}")
message(STATUS "LINK PUBLIC: ${GHA_TOOL_PUBLIC_LINKER_FLAGS}")

##[[add dependencies]]
add_subdirectory("${CMAKE_CURRENT_LIST_DIR}/dependencies/custom/cjson")

#[[configure executable]]
set(GHA_TOOL_SOURCE_DIRECTORY "${PROJECT_SOURCE_DIR}/src")
set(GHA_TOOL_INCLUDE_DIRECTORY "${PROJECT_SOURCE_DIR}/include")
file(
 GLOB_RECURSE
 GHA_TOOL_SOURCES
 LIST_DIRECTORIES FALSE
 "${GHA_TOOL_SOURCE_DIRECTORY}/*.c"
)

function(define_gha_tool_executable name_prefix dst_prefix)
 set(name "${name_prefix}gha-tool")

 add_executable("${name}")
 target_sources(
  "${name}"
  PUBLIC
   ${GHA_TOOL_SOURCES}
 )
 target_include_directories(
  "${name}"
  PUBLIC
   "${PROJECT_SOURCE_DIR}/include"
 )
 target_compile_options("${name}" PRIVATE ${GHA_TOOL_COMPILER_FLAGS})
 target_compile_options("${name}" PUBLIC ${GHA_TOOL_PUBLIC_COMPILER_FLAGS})
 target_link_options("${name}" PRIVATE ${GHA_TOOL_LINKER_FLAGS})
 target_link_options("${name}" PUBLIC ${GHA_TOOL_PUBLIC_LINKER_FLAGS})
 target_link_libraries("${name}" PRIVATE cjson cjson-utils)

 set(
  "${dst_prefix}_NAME"
  "${name}"
  PARENT_SCOPE
 )
endfunction()
gha_tool_add_source_project(define_gha_tool_executable)

#[[enforce sanity on all targets]]
set_target_properties(
 ${GHA_TOOL_TARGETS}
 PROPERTIES
  #enforce C99 w/o GNU extensions
  C_STANDARD 99
  C_EXTENSIONS FALSE
  C_STANDARD_REQUIRED TRUE
  #enable position independent code
  POSITION_INDEPENDENT_CODE TRUE
  #generate compile commands json
  EXPORT_COMPILE_COMMANDS TRUE
)
