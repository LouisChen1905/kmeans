cmake_minimum_required(VERSION 2.8)
project(kmeans)

set(kmeans_MAJOR_VERSION 0)
set(kmeans_MINOR_VERSION 1)
set(kmeans_PATCH_VERSION 0)
set(kmeans_VERSION
  ${kmeans_MAJOR_VERSION}.${kmeans_MINOR_VERSION}.${kmeans_PATCH_VERSION})


set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")

# Offer the user the choice of overriding the installation directories
set(INSTALL_LIB_DIR lib CACHE PATH "Installation directory for libraries")
set(INSTALL_BIN_DIR bin CACHE PATH "Installation directory for executables")
set(INSTALL_INCLUDE_DIR include CACHE PATH
  "Installation directory for header files")
if(WIN32 AND NOT CYGWIN)
  set(DEF_INSTALL_CMAKE_DIR cmake)
else()
  set(DEF_INSTALL_CMAKE_DIR share/kmeans/cmake)
endif()
set(INSTALL_CMAKE_DIR ${DEF_INSTALL_CMAKE_DIR} CACHE PATH
  "Installation directory for CMake files")

# Make relative paths absolute (needed later on)
foreach(p LIB BIN INCLUDE CMAKE)
  set(var INSTALL_${p}_DIR)
  if(NOT IS_ABSOLUTE "${${var}}")
    set(${var} "${CMAKE_INSTALL_PREFIX}/${${var}}")
  endif()
endforeach()


include_directories(include)

set(SOURCE_FILES
        src/point.cpp
        src/kmeans.cpp
        )

add_library(kmeans_lib SHARED ${SOURCE_FILES})
set_target_properties(kmeans_lib PROPERTIES PUBLIC_HEADER
    "include/kmeans/kmeans.h;include/kmeans/point.h")

install(TARGETS kmeans_lib
    EXPORT kmeansTargets
    LIBRARY DESTINATION lib
    PUBLIC_HEADER DESTINATION "${INSTALL_INCLUDE_DIR}/kmeans")

export(TARGETS kmeans_lib
    FILE "${PROJECT_BINARY_DIR}/kmeansTargets.cmake")

export(PACKAGE kmeans)

# Create the FooBarConfig.cmake and FooBarConfigVersion files
file(RELATIVE_PATH REL_INCLUDE_DIR "${INSTALL_CMAKE_DIR}"
   "${INSTALL_INCLUDE_DIR}")
# ... for the build tree
set(CONF_INCLUDE_DIRS "${INSTALL_INCLUDE_DIR}")
configure_file(kmeansConfig.cmake.in
    "${PROJECT_BINARY_DIR}/kmeansConfig.cmake" @ONLY)
# ... for the install tree
set(CONF_INCLUDE_DIRS "\${kmeans_CMAKE_DIR}/${REL_INCLUDE_DIR}")
configure_file(kmeansConfig.cmake.in
    "${PROJECT_BINARY_DIR}${CMAKE_FILES_DIRECTORY}/kmeansConfig.cmake" @ONLY)
# ... for both
configure_file(kmeansConfigVersion.cmake.in
    "${PROJECT_BINARY_DIR}/kmeansConfigVersion.cmake" @ONLY)

install(FILES
    "${PROJECT_BINARY_DIR}${CMAKE_FILES_DIRECTORY}/kmeansConfig.cmake"
    "${PROJECT_BINARY_DIR}/kmeansConfigVersion.cmake"
    DESTINATION "${INSTALL_CMAKE_DIR}" COMPONENT dev)

install(EXPORT kmeansTargets
    DESTINATION "${INSTALL_CMAKE_DIR}" COMPONENT dev)


add_executable(kmeans src/main.cpp)
target_link_libraries(kmeans kmeans_lib)

configure_file(src/sample_data/data_200_2.txt data_200_2.txt COPYONLY)
configure_file(src/sample_data/data_400_50.txt data_400_50.txt COPYONLY)

install(TARGETS kmeans
    EXPORT kmeansTargets
    RUNTIME DESTINATION bin COMPONENT bin)

enable_testing()
add_subdirectory(googletest)
add_subdirectory(src/test)
