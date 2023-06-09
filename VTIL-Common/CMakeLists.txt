# Extract project name from folder
get_filename_component(PROJECT_NAME ${CMAKE_CURRENT_LIST_DIR} NAME)
string(REPLACE " " "_" PROJECT_NAME "${PROJECT_NAME}")

file(GLOB_RECURSE SOURCES CONFIGURE_DEPENDS *.cpp *.hpp)
file(GLOB_RECURSE INCLUDES CONFIGURE_DEPENDS includes/*)

add_library(${PROJECT_NAME} STATIC
    ${SOURCES}
    ${INCLUDES}
)

source_group(TREE ${PROJECT_SOURCE_DIR} FILES ${SOURCES} ${INCLUDES})

target_include_directories(${PROJECT_NAME} PUBLIC includes)

# Common needs Capstone & Keystone
target_link_libraries(${PROJECT_NAME} PUBLIC capstone-static keystone)

# Put all VTIL-specific compiler flags here
# Linking (indirectly) to VTIL-Common will automatically propagate these flags
# https://www.youtube.com/watch?v=bsXLMQ6WgIk

# https://cmake.org/cmake/help/v3.14/manual/cmake-compile-features.7.html#requiring-language-standards
target_compile_features(${PROJECT_NAME} PUBLIC cxx_std_20)

# Make sure ::min/max does not resolve to a Windows.h macro
if(WIN32)
    target_compile_definitions(${PROJECT_NAME} PUBLIC WIN32_LEAN_AND_MEAN NOMINMAX)
endif()

# For portability on non-MSVC compilers, add some compile options
if(CMAKE_CXX_COMPILER_ID MATCHES "Clang|GNU")
    target_compile_options(${PROJECT_NAME} PUBLIC -Wno-narrowing)
    target_compile_options(${PROJECT_NAME} PUBLIC -Wno-format-security)
    target_compile_options(${PROJECT_NAME} PUBLIC -Wno-unused-value)
    target_compile_options(${PROJECT_NAME} PUBLIC -Wno-uninitialized)
    target_compile_options(${PROJECT_NAME} PUBLIC -Wno-parentheses)
    
    # Clang-only flags
    if(CMAKE_CXX_COMPILER_ID MATCHES "Clang")
        target_compile_options(${PROJECT_NAME} PUBLIC -Wno-unknown-attributes)
    endif()

    # GCC-only flags
    if(CMAKE_CXX_COMPILER_ID MATCHES "GNU")
        target_compile_options(${PROJECT_NAME} PUBLIC -Wvolatile)
        target_compile_options(${PROJECT_NAME} PUBLIC -fpermissive)
        target_compile_options(${PROJECT_NAME} PUBLIC -Wno-attributes)
        target_compile_options(${PROJECT_NAME} PUBLIC -Wno-multichar)
    endif()
endif()

if(MSVC)
    # -fpermissive MSVC equivalent
    target_compile_options(${PROJECT_NAME} PUBLIC /permissive-)
endif()

# Include Threads
find_package(Threads REQUIRED)
target_link_libraries(${PROJECT_NAME} PUBLIC ${CMAKE_THREAD_LIBS_INIT})