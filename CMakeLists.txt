cmake_minimum_required(VERSION 3.22)

project(fortinfer LANGUAGES Fortran)

option(FORTINFER_BUILD_TESTING "Build the fortinfer tests" ${PROJECT_IS_TOP_LEVEL})

include(FetchContent)
# A consumer that already has a fortnum target (TIAGO, libneo) keeps it.
set(FORTINFER_FORTNUM_TAG "9a62001569b30c055634391ffedfec0dd8350b22"
    CACHE STRING "fortnum commit used by fortinfer")
if(NOT TARGET fortnum)
    FetchContent_Declare(fortnum
        GIT_REPOSITORY https://github.com/lazy-fortran/fortnum.git
        GIT_TAG ${FORTINFER_FORTNUM_TAG})
    FetchContent_MakeAvailable(fortnum)
endif()

add_library(fortinfer STATIC
    src/generated/fortinfer_empirical_rank_leaf.f90
    src/generated/fortinfer_empirical_tail_leaf.f90
    src/generated/fortinfer_interval_leaf.f90
    src/generated/fortinfer_laplace_leaf.f90
    src/generated/fortinfer_logpdf_leaf.f90
    src/generated/fortinfer_mills_step_leaf.f90
    src/generated/fortinfer_truncated_leaf.f90
    src/fortinfer_affine_filter.f90
    src/fortinfer_empirical.f90
    src/fortinfer_gaussian.f90
    src/fortinfer_laplace.f90
    src/fortinfer_mixture.f90
    src/fortinfer_nuts.f90
    src/fortinfer_psis.f90
    src/fortinfer_regression.f90
    src/fortinfer_truncated.f90)
set_target_properties(fortinfer PROPERTIES
    Fortran_MODULE_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/mod)
target_include_directories(fortinfer PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_BINARY_DIR}/mod>)
target_link_libraries(fortinfer PUBLIC fortnum)

if(FORTINFER_BUILD_TESTING)
    include(CTest)
    FetchContent_Declare(test-drive
        GIT_REPOSITORY https://github.com/fortran-lang/test-drive.git
        GIT_TAG 791484a79a24cbc00be6f74e53c6e030cb75efe2)
    set(TESTDRIVE_BUILD_TESTING OFF)
    FetchContent_MakeAvailable(test-drive)
    foreach(name affine_filter empirical gaussian laplace mixture nuts psis truncated)
        add_executable(test_${name} test/test_${name}.f90)
        set_target_properties(test_${name} PROPERTIES
            Fortran_MODULE_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/mod/test_${name})
        target_link_libraries(test_${name} PRIVATE fortinfer test-drive)
        add_test(NAME ${name} COMMAND test_${name})
    endforeach()
endif()
