cmake_minimum_required(VERSION 3.15)
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

project(myproject VERSION 1.0 DESCRIPTION "secure-license-client" LANGUAGES CXX)

# Add Packages
# find_package(foobar CONFIG REQUIRED)
# find_package(package123 CONFIG REQUIRED)
# ...

# Find CPP files in src/
file(GLOB_RECURSE SRC_FILES "src/*.cpp")

#Create library
add_library(myproject SHARED ${SRC_FILES})

# Link libraries
# target_link_libraries(myproject PRIVATE foobar)
# target_link_libraries(myproject PRIVATE package123)
# ...

#===========================================
# Post-Build Task to convert to Clarion LIB
#===========================================
if(CONVERT_CLARION_LIB)
message(STATUS "Clarion library conversion enabled")

# Get the DLL target name
get_target_property(DLL_OUTPUT_NAME myproject OUTPUT_NAME)
if(NOT DLL_OUTPUT_NAME)
    set(DLL_OUTPUT_NAME myproject)
endif()

# Get the runtime output directory
get_target_property(DLL_OUTPUT_DIR myproject RUNTIME_OUTPUT_DIRECTORY)
if(NOT DLL_OUTPUT_DIR)
    set(DLL_OUTPUT_DIR ${CMAKE_RUNTIME_OUTPUT_DIRECTORY})
endif()

# If no directory, default to binary directory
if(NOT DLL_OUTPUT_DIR)
    set(DLL_OUTPUT_DIR ${CMAKE_BINARY_DIR})
endif()

set(PATH_TO_DLL "${DLL_OUTPUT_DIR}/$<CONFIG>/${DLL_OUTPUT_NAME}${CMAKE_SHARED_LIBRARY_SUFFIX}")
set(PATH_TO_LIB "${DLL_OUTPUT_DIR}/$<CONFIG>/Clarion/${DLL_OUTPUT_NAME}${CMAKE_STATIC_LIBRARY_SUFFIX}")


#Create the Clarion Lib output directory
add_custom_command(TARGET myproject POST_BUILD
    COMMAND ${CMAKE_COMMAND} -E make_directory "${DLL_OUTPUT_DIR}/$<CONFIG>/Clarion"
)

# Post-build task to convert DLL to Clarion .LIB using external tool
add_custom_command(TARGET myproject POST_BUILD
    COMMAND "${CMAKE_SOURCE_DIR}/tools/libmaker/MGLibMkrMinus.exe"
    ARGS "READ=${PATH_TO_DLL}" "WRITE=${PATH_TO_LIB}" "/CLOSE"
    COMMENT "Converting DLL to Clarion .LIB"
)

endif()
