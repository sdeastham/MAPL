cmake_minimum_required (VERSION 3.13)
cmake_policy (SET CMP0053 NEW)
cmake_policy (SET CMP0054 NEW)

project (
  MAPL
  VERSION 2.2.7
  LANGUAGES Fortran CXX C)  # Note - CXX is required for ESMF

if (EXISTS ${CMAKE_CURRENT_LIST_DIR}/ESMA_cmake)
  list (APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_LIST_DIR}/ESMA_cmake")
  include (esma)
else ()
  if (NOT COMMAND esma) # build as standalone project

    if (NOT SKIP_MEPO)
       set (MEPO_INIT_COMMAND mepo init)
       execute_process (
          COMMAND ${MEPO_INIT_COMMAND}
          WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}
          )

       set (MEPO_CLONE_COMMAND mepo clone)
       execute_process (
          COMMAND ${MEPO_CLONE_COMMAND}
          WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}
          )
    endif()
    option (SKIP_MEPO "Set to skip mepo steps" ON)

    list (APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_LIST_DIR}/ESMA_cmake")
    include (esma)
  endif()
endif()

# Turn off for GCHP
#option(BUILD_WITH_FLAP "Use FLAP for command line processing" ON)

ecbuild_declare_project()

###find_package(PFLOGGER REQUIRED)
find_package(PFLOGGER ${PFLOGGER_IS_REQUIRED_ARG} CONFIG)

# Special case - MAPL_cfio is built twice with two different precisions.
add_subdirectory (MAPL_cfio MAPL_cfio_r4)
add_subdirectory (MAPL_cfio MAPL_cfio_r8)

add_subdirectory (pfio)
add_subdirectory (profiler)
add_subdirectory (generic)
add_subdirectory (shared)
add_subdirectory (include)
add_subdirectory (base)
add_subdirectory (MAPL)

if (PFUNIT_FOUND)
  add_subdirectory (MAPL_pFUnit EXCLUDE_FROM_ALL)
endif ()

# Support for automated code generation
list (APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_LIST_DIR}/cmake")
include(mapl_acg)
include(mapl_create_stub_component)
add_subdirectory (Apps)

add_subdirectory (Tests)

# @env will exist here if MAPL is built as itself but not as part of, say, GEOSgcm
esma_add_subdirectory (ESMA_env FOUND ESMA_env_FOUND)

# Install the Python directory
install (
   DIRECTORY Python
   DESTINATION lib
   USE_SOURCE_PERMISSIONS
   )

ecbuild_install_project (NAME MAPL)
