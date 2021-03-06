include_directories("../common")
include_directories("../caliper")
include_directories(${CMAKE_CURRENT_BINARY_DIR})

# A macro to include service module source files in the caliper runtime lib

macro(add_service_sources)
  file(RELATIVE_PATH _relpath "${CMAKE_SOURCE_DIR}/src/services" "${CMAKE_CURRENT_SOURCE_DIR}")
  foreach(_src ${ARGN})
    if (_relpath)
      list(APPEND CALIPER_SERVICES_SOURCES "${_relpath}/${_src}")
    else()
      list(APPEND CALIPER_SERVICES_SOURCES "${_src}")
    endif()
  endforeach()
  if (_relpath)
    set(CALIPER_SERVICES_SOURCES ${CALIPER_SERVICES_SOURCES} PARENT_SCOPE)
  endif()
endmacro()

macro(add_caliper_service)
  string(REPLACE " " ";" NEW_SERVICE ${ARGV0})
  set(CALIPER_SERVICE_NAMES "${CALIPER_SERVICE_NAMES} ${NEW_SERVICE}" PARENT_SCOPE)
endmacro()
# A macro to include service modules as object libs in the caliper runtime lib.
# Used when service subdirectories needs additional includes etc. 

macro(add_service_objlib)
  if(${BUILD_SHARED_LIBS})
    set_property(TARGET "${ARGN}" PROPERTY POSITION_INDEPENDENT_CODE TRUE)
  endif()
  list(APPEND CALIPER_SERVICES_LIBS "$<TARGET_OBJECTS:${ARGN}>")
  set(CALIPER_SERVICES_LIBS ${CALIPER_SERVICES_LIBS} PARENT_SCOPE)
endmacro()

# Service subdirectories

add_subdirectory(aggregate)
if (LIBUNWIND_FOUND)
  add_subdirectory(callpath)
endif()
if (PAPI_FOUND)
  add_subdirectory(papi)
endif()
if (LIBCURL_FOUND)
  add_subdirectory(netout)
endif(LIBCURL_FOUND)
if (Mitos_FOUND)
  add_subdirectory(mitos)
endif()
if (CALIPER_HAVE_NVVP)
  add_subdirectory(nvvp)
endif()
add_subdirectory(debug)
add_subdirectory(env)
add_subdirectory(git)
add_subdirectory(service_cmake)
add_subdirectory(event)
add_subdirectory(textlog)
add_subdirectory(pthread)
add_subdirectory(recorder)
if(CALIPER_HAVE_SAMPLER)
  add_subdirectory(sampler)
endif()
add_subdirectory(timestamp)
add_subdirectory(statistics)
if(CALIPER_HAVE_TAU)
find_library(tau_lib  libTAU.so)
add_subdirectory(tau)
endif()
add_subdirectory(trace)
if (MPI_C_FOUND)
  add_subdirectory(mpiwrap)
endif()
if (OMPT_FOUND)
  add_subdirectory(ompt)
endif()
add_subdirectory(validator)

add_service_sources(Services.cpp)

configure_file(
   ${CMAKE_CURRENT_SOURCE_DIR}/gen_services_inc.py.in
   ${CMAKE_CURRENT_BINARY_DIR}/gen_services_inc.py
)

find_package(PythonInterp REQUIRED)

add_custom_target(services.inc.cpp
  COMMAND ${PYTHON_EXECUTABLE} gen_services_inc.py
  WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}
  COMMENT "Generating services.inc.cpp"
)

# Build service management classes 

add_library(caliper-services OBJECT ${CALIPER_SERVICES_SOURCES})

add_dependencies(caliper-services services.inc.cpp)

if (${BUILD_SHARED_LIBS})
  set_property(TARGET caliper-services PROPERTY POSITION_INDEPENDENT_CODE TRUE)
endif()

set(CALIPER_SERVICES_LIBS ${CALIPER_SERVICES_LIBS} PARENT_SCOPE)
