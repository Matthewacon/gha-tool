cmake_minimum_required(VERSION 3.25)

#TODO: use `git describe --tags` to get version
set(version 1.7.17)

project(
 cjson
 VERSION ${version}
 LANGUAGES C
)
unset(version)

#[[set up base project]]
include("${CMAKE_CURRENT_LIST_DIR}/../../cakemake/CMakeLists.txt")

#[[build flags]]
set(CJSON_DEBUG FALSE CACHE BOOL "Enable debug flags for cJSON")
if(CJSON_DEBUG)
 set(CJSON_NOT_DEBUG FALSE)
else()
 set(CJSON_NOT_DEBUG TRUE)
endif()
set(CJSON_STATIC ${CJSON_NOT_DEBUG} CACHE BOOL "Enable flags for building as a static archive")
set(CJSON_MUSL_LIBC ${CJSON_NOT_DEBUG} CACHE BOOL "Enable flags for linking against musl libc")
unset(CJSON_NOT_DEBUG)
set(CJSON_SANITIZERS ${CJSON_DEBUG} CACHE BOOL "Enable flags for building with sanitizers")

#[[add all compiler and linker flags to global lists]]
cjson_add_default_flags()
cjson_add_debug_flags(CJSON_DEBUG)
cjson_add_sanitizer_flags(CJSON_SANITIZERS)
cjson_add_static_flags(CJSON_STATIC)
cjson_add_musl_libc_flags(CJSON_MUSL_LIBC)
#TODO: cjson_add_test_flags(CJSON_TESTS)

#[[configure libraries]]
#public compiler flags for all cjson targets and dependents
list(
 APPEND CJSON_PUBLIC_COMPILER_FLAGS
  -DCJSON_EXPORT_SYMBOLS
  -DCJSON_API_VISIBILITY
)

#cjson library
function(define_cjson_library name_prefix dst_prefix)
 set(name "${name_prefix}cjson")

 add_library("${name}")
 target_sources(
   "${name}"
   PUBLIC
    ../../cJSON/cJSON.c
 )
 target_include_directories(
  "${name}"
  PUBLIC
   ../../cJSON
 )
 target_compile_options("${name}" PRIVATE ${CJSON_COMPILER_FLAGS})
 target_compile_options("${name}" PUBLIC ${CJSON_PUBLIC_COMPILER_FLAGS})
 target_link_options("${name}" PRIVATE ${CJSON_LINKER_FLAGS})
 target_link_options("${name}" PUBLIC ${CJSON_PUBLIC_LINKER_FLAGS})

 set(
  "${dst_prefix}_NAME"
  "${name}"
  PARENT_SCOPE
 )
endfunction()
cjson_add_source_project(define_cjson_library)

#cjson-utils library
function(define_cjson_utils_library name_prefix dst_prefix)
 set(name "${name_prefix}cjson-utils")

 add_library("${name}")
 target_sources(
   "${name}"
   PUBLIC
    ../../cJSON/cJSON_Utils.c
 )
 target_include_directories(
  "${name}"
  PUBLIC
   ../../cJSON
 )
 target_compile_options("${name}" PRIVATE ${CJSON_COMPILER_FLAGS})
 target_compile_options("${name}" PUBLIC ${CJSON_PUBLIC_COMPILER_FLAGS})
 target_link_options("${name}" PRIVATE ${CJSON_LINKER_FLAGS})
 target_link_options("${name}" PUBLIC ${CJSON_PUBLIC_LINKER_FLAGS})

 set(
  "${dst_prefix}_NAME"
  "${name}"
  PARENT_SCOPE
 )
endfunction()
cjson_add_source_project(define_cjson_utils_library)

#[[enforce sanity on all targets]]
set_target_properties(
 ${CJSON_TARGETS}
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
