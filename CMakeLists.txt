cmake_minimum_required(VERSION 3.11) # picked version 3.11 because that's when
                                     # FetchContent was added, make sure
                                     # that all the functions you rely on
                                     # exist in the version you pick
set(CMAKE_CXX_STANDARD 17)
# above is equal to:
# g++ ... -std=c++17
# Pick an older standard unless you have a specific reason,
# are you really using concepts and [[assume]]?

# Include directories
include_directories(/path/to/external/includes)

# Compile library
add_library(ABC A.cpp B.cpp C.cpp)

# Change name based on what the name of your main file is
add_executable(main main.cpp)

# Link libraries ("ABC" and external)
target_link_libraries(main ABC)
target_link_libraries(main /path/to/external/library.a)

# Both must have this option if you want to be able to place
# breakpoints in vscode and use debugging
target_compile_options(main PRIVATE -g)
target_compile_options(ABC PRIVATE -g)

### CxxTest

# search for cxxtest on the system,
# user may have already installed 
# it with a package manager
find_package(CxxTest)
# result gets stored in variable CXXTEST_FOUND

# if not found, download from git
if (NOT CXXTEST_FOUND)
    include(FetchContent)
    FetchContent_Declare(
    cxxtest
        GIT_REPOSITORY https://github.com/CxxTest/cxxtest.git
    )
    FetchContent_MakeAvailable(cxxtest)
    set(CXXTEST_INCLUDE_DIR ${CMAKE_SOURCE_DIR}/build/_deps/cxxtest-src/bin)
    find_package(CxxTest REQUIRED) #run it again, 
    # it screws up build cache variables 
    # if it is never properly found
endif()

include_directories(${CXXTEST_INCLUDE_DIR}/../)
enable_testing()

# below assumes your CxxTest is written in a file named mytest.h
CXXTEST_ADD_TEST(mytest mytest.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/mytest.h)
target_link_libraries(mytest ABC)

# If you want to use breakpoints/debugging your unit test.
# When debugging unit test, select the "mytest.h" file
# and press F5 on it.
target_compile_options(mytest PRIVATE -g)