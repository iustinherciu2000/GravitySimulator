cmake_minimum_required(VERSION 3.22)
project(gravity_sim_all LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
add_compile_definitions(GL_SILENCE_DEPRECATION)

# Build Apple-Silicon binaries
if(APPLE)
    set(CMAKE_OSX_ARCHITECTURES "arm64")
endif()

# Homebrew prefix (M1 = /opt/homebrew; Intel = /usr/local)
set(HOMEBREW_PREFIX "/opt/homebrew")
if(NOT EXISTS "${HOMEBREW_PREFIX}/include")
    set(HOMEBREW_PREFIX "/usr/local")
endif()
list(APPEND CMAKE_PREFIX_PATH "${HOMEBREW_PREFIX}")

find_package(OpenGL REQUIRED)
find_package(glfw3 CONFIG REQUIRED)   # uses /opt/homebrew/lib/cmake/glfw3
find_package(GLEW REQUIRED)

# GLM is header-only
find_path(GLM_INCLUDE_DIR glm/glm.hpp
        PATHS "${HOMEBREW_PREFIX}/include" "/usr/local/include" REQUIRED)

function(add_app name src)
    add_executable(${name} ${src})
    target_include_directories(${name} PRIVATE ${GLM_INCLUDE_DIR})
    target_link_libraries(${name} PRIVATE OpenGL::GL glfw GLEW::GLEW)
    if(APPLE)
        target_link_libraries(${name} PRIVATE "-framework Cocoa" "-framework IOKit" "-framework CoreVideo")
    endif()
endfunction()

add_app(3D_test            3D_test.cpp)
add_app(gravity_sim gravity_simulator.cpp)
add_app(gravity_sim_3Dgrid gravity_simulator_3DGrid.cpp)
