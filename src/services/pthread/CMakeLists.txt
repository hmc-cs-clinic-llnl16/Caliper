set(CALIPER_PTHREAD_SOURCES
    PthreadService.cpp)

add_service_sources(${CALIPER_PTHREAD_SOURCES})
add_caliper_service("pthread")
