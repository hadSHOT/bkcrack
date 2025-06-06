cmake_minimum_required(VERSION 3.19)

# project definition
project(bkcrack
    VERSION 1.7.1 # remember to update bkcrack_VERSION_DATE below when releasing a new version
    DESCRIPTION "Crack legacy zip encryption with Biham and Kocher's known plaintext attack."
    HOMEPAGE_URL "https://github.com/kimci86/bkcrack"
    LANGUAGES CXX)
set(bkcrack_VERSION_DATE "2024-12-21")

# default build type
set(default_build_type "Release")
if(NOT CMAKE_BUILD_TYPE AND NOT CMAKE_CONFIGURATION_TYPES)
    message(STATUS "Setting build type to '${default_build_type}' as none was specified.")
    set(CMAKE_BUILD_TYPE "${default_build_type}" CACHE STRING "Choose the type of build." FORCE)
    set_property(CACHE CMAKE_BUILD_TYPE PROPERTY STRINGS "Debug" "Release" "MinSizeRel" "RelWithDebInfo")
endif()

# main program
add_subdirectory(src)

# documentation generation
option(BKCRACK_BUILD_DOC "Enable documentation generation with doxygen." OFF)
if(BKCRACK_BUILD_DOC)
    add_subdirectory(doc)
endif()

# automated tests
option(BKCRACK_BUILD_TESTING "Enable automated testing with ctest." OFF)
if(BKCRACK_BUILD_TESTING)
    enable_testing()
    add_subdirectory(tests)
endif()

# code coverage report
option(BKCRACK_BUILD_COVERAGE "Enable code coverage report generation." OFF)
if(BKCRACK_BUILD_COVERAGE)
    target_compile_options(bkcrack PRIVATE --coverage)
    target_link_options(bkcrack PRIVATE --coverage)

    find_program(BKCRACK_GCOVR_EXECUTABLE gcovr DOC "Path to gcovr program used to generate code coverage report." REQUIRED)

    add_custom_target(coverage
        COMMAND ${CMAKE_COMMAND} -E make_directory coverage
        COMMAND ${BKCRACK_GCOVR_EXECUTABLE} --root ${CMAKE_SOURCE_DIR} --html-nested coverage/index.html
        COMMENT "Generating code coverage HTML report"
        VERBATIM)

    add_custom_target(coveralls
        COMMAND ${BKCRACK_GCOVR_EXECUTABLE} --root ${CMAKE_SOURCE_DIR} --coveralls coverage.json
        COMMENT "Generating coverage.json report for coveralls.io"
        VERBATIM)
endif()

# code formatting
find_program(BKCRACK_CLANG_FORMAT_EXECUTABLE clang-format DOC "Path to clang-format program used to format C++ code.")
if(BKCRACK_CLANG_FORMAT_EXECUTABLE)
    get_target_property(files_to_format bkcrack SOURCES)
    add_custom_target(format
        COMMAND ${BKCRACK_CLANG_FORMAT_EXECUTABLE} -i ${files_to_format}
        COMMENT "Formatting C++ code with ${BKCRACK_CLANG_FORMAT_EXECUTABLE}"
        VERBATIM)
endif()

# install rules
install(DIRECTORY example DESTINATION .)
install(DIRECTORY tools DESTINATION .)
install(FILES readme.md DESTINATION .)
install(FILES license.txt DESTINATION .)

# package generation
if(WIN32)
    set(CPACK_GENERATOR "ZIP")
else()
    set(CPACK_GENERATOR "TGZ")
endif()
if(APPLE)
    set(CPACK_SYSTEM_NAME "macOS-${CMAKE_HOST_SYSTEM_PROCESSOR}")
elseif(NOT WIN32)
    set(CPACK_SYSTEM_NAME "${CMAKE_SYSTEM_NAME}-${CMAKE_HOST_SYSTEM_PROCESSOR}")
endif()
include(CPack)
