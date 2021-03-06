include(ExternalProject)

# Mandatory dependencis not found in the system will be built
# from submodules

# --------------------------------------------------------------------------------
# GLM
# --------------------------------------------------------------------------------
if(NOT glm_FOUND AND NOT TARGET glm)
    message(STATUS "Gathering GLM from submodules")

    add_library(glm INTERFACE)
    target_include_directories(glm INTERFACE
        $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/glm>)

    # Export target
    install(TARGETS glm
        EXPORT Brion-targets
        LIBRARY DESTINATION lib
        ARCHIVE DESTINATION lib
        INCLUDES DESTINATION include
    )
    install(TARGETS glm
        EXPORT Brain-targets
        LIBRARY DESTINATION lib
        ARCHIVE DESTINATION lib
        INCLUDES DESTINATION include
    )

    install(DIRECTORY glm/glm DESTINATION include)
endif()

# WAR for https://github.com/g-truc/glm/issues/854
add_compile_definitions(GLM_FORCE_CXX14)
if(CMAKE_COMPILER_IS_GNUCXX AND NOT CMAKE_CXX_COMPILER_VERSION VERSION_LESS 8.0)
  target_compile_options(glm INTERFACE -Wno-error=class-memaccess)
endif()
# Disable warnings
target_compile_options(glm INTERFACE -w)

# --------------------------------------------------------------------------------
# sonata (which also brings HighFive)
# --------------------------------------------------------------------------------
if(NOT sonata_FOUND AND NOT TARGET sonata::sonata_shared)
    message(STATUS "Gathering sonata from submodules")
    # Sonata links against gcov, but on Apple this library is not included
    # on the library path
    if(APPLE)
        add_library(gcov INTERFACE)
        # Export target
        install(TARGETS gcov EXPORT gcov-targets)
        install(EXPORT gcov-targets DESTINATION share/gcov/CMake)
        set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} -fprofile-instr-generate -fcoverage-mapping")
        set(CMAKE_C_FLAGS_DEBUG "${CMAKE_C_FLAGS_DEBUG} -fprofile-instr-generate -fcoverage-mapping")
    endif()
    set(SONATA_VERSION "0.1.8")
    add_subdirectory(libsonata)
    add_library(sonatainter INTERFACE)
    target_include_directories(sonatainter INTERFACE ${CMAKE_CURRENT_LIST_DIR}/libsonata/include)
    target_link_libraries(sonatainter INTERFACE sonata_shared)
    add_library(sonata::sonata_shared ALIAS sonatainter)
endif()

# --------------------------------------------------------------------------------
# HighFive (If sonata came from a package but HighFive was not found)
# --------------------------------------------------------------------------------
if(NOT HighFive_FOUND AND NOT TARGET HighFive)
    message(STATUS "Gathering HighFive from submodules")
    add_subdirectory(HighFive)
endif()

# --------------------------------------------------------------------------------
# MVDTool
# --------------------------------------------------------------------------------
if(NOT MVDTool_FOUND AND NOT TARGET MVDTool)
    message(STATUS "Gathering MVDTool from submodules")
    find_package(Boost 1.41 QUIET REQUIRED COMPONENTS system)
    find_package(HDF5 QUIET REQUIRED)

    add_library(MVDTool INTERFACE)

    target_include_directories(MVDTool SYSTEM INTERFACE
          $<BUILD_INTERFACE:${CMAKE_CURRENT_LIST_DIR}/MVDTool/include>
          $<INSTALL_INTERFACE:include>)
    target_include_directories(MVDTool SYSTEM INTERFACE ${Boost_INCLUDE_DIR})
    target_link_libraries(MVDTool INTERFACE HighFive)
    target_compile_definitions(MVDTool INTERFACE -DH5_USE_BOOST)
endif()

# --------------------------------------------------------------------------------
# MorphIO
# --------------------------------------------------------------------------------
if(NOT MorphIO_FOUND AND NOT TARGET MorphIO::morphio_shared)
    set(MORPHIO_INCLUDE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/MorphIO/include/)

    # Deps
    add_subdirectory(MorphIO/3rdparty/GSL_LITE)
    target_include_directories(gsl-lite SYSTEM INTERFACE)
    add_library(lexertl INTERFACE)
    target_include_directories(lexertl SYSTEM INTERFACE MorphIO/3rdparty/lexertl14/include)

    # Retrieve version
    set(MORPHIO_VERSION_STRING "2.3.10")
    set(MorphIO_VERSION_MAJOR 2)
    set(MorphIO_VERSION_MINOR 3)
    set(MorphIO_VERSION_PATCH 10)
    # Morphio
    add_subdirectory(MorphIO/src)
    add_library(morphiointer INTERFACE)
    target_include_directories(morphiointer INTERFACE ${CMAKE_CURRENT_LIST_DIR}/MorphIO/include)
    target_link_libraries(morphiointer INTERFACE morphio_shared)
    add_library(MorphIO::morphio_shared ALIAS morphiointer)
endif()
