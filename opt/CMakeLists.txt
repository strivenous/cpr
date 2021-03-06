macro(set_cache_variable VAR_NAME VAR_DESCRIPTION)
    set(${VAR_NAME} ${${VAR_NAME}} CACHE INTERNAL ${VAR_DESCRIPTION})
    message(STATUS "Set ${VAR_NAME} to ${${VAR_NAME}}.")
endmacro()


# Code coverage

if(BUILD_CPR_TESTS AND GENERATE_COVERAGE)
    set(CMAKE_BUILD_TYPE COVERAGE CACHE INTERNAL "Coverage enabled build")
    message(STATUS "Enabling gcov support")
    if(NOT "${CMAKE_CXX_COMPILER_ID}" STREQUAL "Clang")
        set(COVERAGE_FLAG "--coverage")
    endif()
    set(CMAKE_CXX_FLAGS_COVERAGE
        "-g -O0 ${COVERAGE_FLAG} -fprofile-arcs -ftest-coverage"
        CACHE STRING "Flags used by the C++ compiler during coverage builds."
        FORCE)
    set(CMAKE_C_FLAGS_COVERAGE
        "-g -O0 ${COVERAGE_FLAG} -fprofile-arcs -ftest-coverage"
        CACHE STRING "Flags used by the C compiler during coverage builds."
        FORCE)
    set(CMAKE_EXE_LINKER_FLAGS_COVERAGE
        ""
        CACHE STRING "Flags used for linking binaries during coverage builds."
        FORCE)
    set(CMAKE_SHARED_LINKER_FLAGS_COVERAGE
        ""
        CACHE STRING "Flags used by the shared libraries linker during coverage builds."
        FORCE)
    mark_as_advanced(
        CMAKE_CXX_FLAGS_COVERAGE
        CMAKE_C_FLAGS_COVERAGE
        CMAKE_EXE_LINKER_FLAGS_COVERAGE
        CMAKE_SHARED_LINKER_FLAGS_COVERAGE)
endif()


# Curl configuration

if(USE_SYSTEM_CURL)
    find_package(CURL)
endif()
if(NOT USE_SYSTEM_CURL OR NOT CURL_FOUND)
    message(STATUS "Not using system Curl, using built-in curl project instead.")
    option(BUILD_TESTING "Set to ON to build cURL tests." OFF)
    option(BUILD_CURL_EXE "Set to ON to build cURL executable." OFF)
    add_subdirectory(curl EXCLUDE_FROM_ALL)
    set(CURL_FOUND TRUE)
    set(CURL_LIBRARIES libcurl)
    set(CURL_INCLUDE_DIRS
        ${CURL_SOURCE_DIR}/include
        ${CURL_BINARY_DIR}/include/curl)

    # Group under the "external" project folder in IDEs such as Visual Studio.
    if(BUILD_CURL_EXE)
        set_property(TARGET curl PROPERTY FOLDER "external")
    endif()

    set_property(TARGET libcurl PROPERTY FOLDER "external")
endif()

set_cache_variable(CURL_FOUND "Set if libcurl is found or built")
set_cache_variable(CURL_LIBRARIES "Location of libcurl")
set_cache_variable(CURL_INCLUDE_DIRS "Location of curl include files")


# SSL configuration
if(CMAKE_USE_OPENSSL)
    find_package(OpenSSL REQUIRED)

    message(STATUS "using system OpenSSL library.")

    include_directories(${OPENSSL_INCLUDE_DIR})
    set(SSL_LIBRARIES ${OPENSSL_LIBRARIES})
endif()

# GTest configuration

if(BUILD_CPR_TESTS)
    if(USE_SYSTEM_GTEST)
        find_package(GTest)
    endif()
    if(NOT USE_SYSTEM_GTEST OR NOT GTEST_FOUND)
        message(STATUS "Not using system gtest, using built-in googletest project instead.")
        if(MSVC)
            # By default, GTest compiles on Windows in CRT static linkage mode. We use this
            # variable to force it into using the CRT in dynamic linkage (DLL), just as CPR
            # does.
            set(gtest_force_shared_crt ON CACHE BOOL "Force gtest to use the shared c runtime")
        endif()
        add_subdirectory(googletest)
        set(GTEST_FOUND TRUE)
        set(GTEST_LIBRARIES gtest)
        set(GTEST_MAIN_LIBRARIES gtest_main)
        set(GTEST_BOTH_LIBRARIES gtest gtest_main)
        set(GTEST_INCLUDE_DIRS ${gtest_SOURCE_DIR}/include)

        # Group under the "tests/gtest" project folder in IDEs such as Visual Studio.
    set_property(TARGET gtest PROPERTY FOLDER "tests/gtest")
    set_property(TARGET gtest_main PROPERTY FOLDER "tests/gtest")
    endif()

    set_cache_variable(GTEST_FOUND "Set if libgtest was found or built")
    set_cache_variable(GTEST_LIBRARIES "Location of libgtest")
    set_cache_variable(GTEST_MAIN_LIBRARIES "Location of libgtest-main")
    set_cache_variable(GTEST_BOTH_LIBRARIES "Location of both gtest libraries")
    set_cache_variable(GTEST_INCLUDE_DIRS "Location of gtest include files")
endif()


# Mongoose configuration

if(BUILD_CPR_TESTS)
    message(STATUS "Building mongoose project for test support.")

    project(mongoose C)
    add_library(mongoose STATIC mongoose/mongoose.c)
    if(CMAKE_USE_OPENSSL)
        target_compile_definitions(mongoose PUBLIC NS_ENABLE_SSL)
        target_link_libraries(mongoose ${OPENSSL_LIBRARIES})
    endif()

    set(MONGOOSE_FOUND TRUE)
    set(MONGOOSE_LIBRARIES mongoose)
    set(MONGOOSE_INCLUDE_DIRS ${CMAKE_CURRENT_SOURCE_DIR}/mongoose)

    include_directories(${MONGOOSE_INCLUDE_DIRS})

    set_cache_variable(MONGOOSE_FOUND "Set if libmongoose was found or built")
    set_cache_variable(MONGOOSE_LIBRARIES "Location of libmongoose")
    set_cache_variable(MONGOOSE_INCLUDE_DIRS "Location of mongoose include files")

    # Group under the "external" project folder in IDEs such as Visual Studio.
    set_property(TARGET mongoose PROPERTY FOLDER "external")
endif()
