cmake_minimum_required(VERSION 3.15)
project(ppcg_polly C CXX)

set(CMAKE_C_STANDARD 99)
set(CMAKE_C_STANDARD_REQUIRED ON)

find_package(Git)
if(GIT_FOUND)
    execute_process(
        COMMAND ${GIT_EXECUTABLE} rev-parse HEAD
        WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
        OUTPUT_VARIABLE GIT_HEAD_VERSION
        OUTPUT_STRIP_TRAILING_WHITESPACE
    )
else()
    set(GIT_HEAD_VERSION "polly_integration_version")
endif()

file(WRITE ${CMAKE_CURRENT_BINARY_DIR}/gitversion.h 
    "#define GIT_HEAD_ID \"${GIT_HEAD_VERSION}\"\n"
)

set(PPCG_CORE_SOURCES
    cpu.c cpu.h
    cuda.c cuda.h
    opencl.c opencl.h
    cuda_common.c cuda_common.h
    gpu.c gpu.h
    gpu_array_tile.c gpu_array_tile.h
    gpu_group.c gpu_group.h
    gpu_hybrid.c gpu_hybrid.h
    gpu_print.c gpu_print.h
    gpu_tree.c gpu_tree.h
    grouping.c grouping.h
    hybrid.c hybrid.h
    schedule.c schedule.h
    ppcg_options.c ppcg_options.h
    print.c print.h
    util.c util.h
    version.c

    pet.c pet.h

    ppcg.c ppcg.h
)

add_library(ppcg STATIC ${PPCG_CORE_SOURCES})
target_include_directories(ppcg PRIVATE
    ${CMAKE_CURRENT_BINARY_DIR}
    ${CMAKE_CURRENT_SOURCE_DIR}/pet_mock/include
)

if(TARGET PollyISL)
    target_link_libraries(ppcg PRIVATE PollyISL)
else()
    # find_package(PkgConfig REQUIRED)
    # pkg_check_modules(ISL REQUIRED isl)
    #
    # target_include_directories(ppcg PRIVATE ${ISL_INCLUDE_DIRS})
    # target_link_libraries(ppcg PRIVATE ${ISL_LIBRARIES})
    #
    # target_include_directories(ppcgtest PRIVATE ${ISL_INCLUDE_DIRS})
    # target_link_libraries(ppcgtest PRIVATE ${ISL_LIBRARIES})

    set(ISL_INCLUDE_DIRS "${CMAKE_CURRENT_SOURCE_DIR}/isl/include")
    set(ISL_LIBRARIES "${CMAKE_CURRENT_SOURCE_DIR}/isl/.libs/libisl.a")
    
    target_include_directories(ppcg PRIVATE ${ISL_INCLUDE_DIRS})
    target_link_libraries(ppcg PRIVATE ${ISL_LIBRARIES} gmp)
endif()
