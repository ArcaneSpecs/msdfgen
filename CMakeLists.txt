##cmake_minimum_required(VERSION 3.15)
##include(cmake/version.cmake)
##
##option(MSDFGEN_CORE_ONLY "Only build the core library with no dependencies" OFF)
##option(MSDFGEN_BUILD_STANDALONE "Build the msdfgen standalone executable" ON)
##option(MSDFGEN_USE_VCPKG "Use vcpkg package manager to link project dependencies" ON)
##option(MSDFGEN_USE_OPENMP "Build with OpenMP support for multithreaded code" OFF)
##option(MSDFGEN_USE_CPP11 "Build with C++11 enabled" ON)
##option(MSDFGEN_USE_SKIA "Build with the Skia library" ON)
##option(MSDFGEN_INSTALL "Generate installation target" OFF)
##option(MSDFGEN_DYNAMIC_RUNTIME "Link dynamic runtime library instead of static" OFF)
##option(BUILD_SHARED_LIBS "Generate dynamic library files instead of static" OFF)
##
##if(MSDFGEN_CORE_ONLY AND MSDFGEN_BUILD_STANDALONE)
##    message(WARNING "Option MSDFGEN_CORE_ONLY ignored - extensions are required for standalone executable")
##    set(MSDFGEN_CORE_ONLY OFF)
##endif()
##if(MSDFGEN_CORE_ONLY AND MSDFGEN_USE_VCPKG)
##    message(STATUS "Option MSDFGEN_USE_VCPKG ignored due to MSDFGEN_CORE_ONLY - core has no dependencies")
##    set(MSDFGEN_USE_VCPKG OFF)
##endif()
##
##get_property(MULTI_CONFIG GLOBAL PROPERTY GENERATOR_IS_MULTI_CONFIG)
##if(NOT MULTI_CONFIG AND NOT CMAKE_BUILD_TYPE)
##    message(STATUS "CMAKE_BUILD_TYPE not set, defaulting to Release")
##    set(CMAKE_BUILD_TYPE Release)
##endif()
##
##if(MSDFGEN_DYNAMIC_RUNTIME)
##    set(MSDFGEN_MSVC_RUNTIME "MultiThreaded$<$<CONFIG:Debug>:Debug>DLL")
##else()
##    set(MSDFGEN_MSVC_RUNTIME "MultiThreaded$<$<CONFIG:Debug>:Debug>")
##endif()
##
##if(BUILD_SHARED_LIBS)
##    set(CMAKE_WINDOWS_EXPORT_ALL_SYMBOLS ON)
##endif()
##
##if(MSDFGEN_USE_VCPKG)
##    # Make sure that vcpkg toolchain file is set
##    if(NOT CMAKE_TOOLCHAIN_FILE)
##        if(DEFINED ENV{VCPKG_ROOT})
##            set(CMAKE_TOOLCHAIN_FILE "$ENV{VCPKG_ROOT}/scripts/buildsystems/vcpkg.cmake")
##        else()
##            message(SEND_ERROR "Vcpkg toolchain not configured. Either set VCPKG_ROOT environment variable or pass -DCMAKE_TOOLCHAIN_FILE=VCPKG_ROOT/scripts/buildsystems/vcpkg.cmake to cmake")
##        endif()
##    endif()
##    # Default to statically linked vcpkg triplet on Windows
##    if(WIN32 AND NOT VCPKG_TARGET_TRIPLET AND NOT MSDFGEN_DYNAMIC_RUNTIME)
##        if(CMAKE_GENERATOR_PLATFORM MATCHES "64$" AND NOT CMAKE_GENERATOR_PLATFORM STREQUAL "ARM64")
##            set(VCPKG_TARGET_TRIPLET "x64-windows-static")
##        elseif(CMAKE_GENERATOR_PLATFORM MATCHES "32$" OR CMAKE_GENERATOR_PLATFORM STREQUAL "x86")
##            set(VCPKG_TARGET_TRIPLET "x86-windows-static")
##        else()
##            if(CMAKE_GENERATOR_PLATFORM)
##                message(WARNING "Vcpkg triplet not explicitly specified and could not be deduced. Recommend using -DVCPKG_TARGET_TRIPLET=x64-windows-static or similar")
##            else()
##                message(WARNING "Vcpkg triplet not explicitly specified and could not be deduced. Recommend using -A to explicitly select platform (Win32 or x64)")
##            endif()
##        endif()
##    endif()
##    # Select project features
##    if(NOT MSDFGEN_VCPKG_FEATURES_SET)
##        set(VCPKG_MANIFEST_NO_DEFAULT_FEATURES ON)
##        if(NOT MSDFGEN_CORE_ONLY)
##            list(APPEND VCPKG_MANIFEST_FEATURES "extensions")
##        endif()
##        if(MSDFGEN_BUILD_STANDALONE)
##            list(APPEND VCPKG_MANIFEST_FEATURES "standalone")
##        endif()
##        if(MSDFGEN_USE_SKIA)
##            list(APPEND VCPKG_MANIFEST_FEATURES "geometry-preprocessing")
##        endif()
##        if(MSDFGEN_USE_OPENMP)
##            list(APPEND VCPKG_MANIFEST_FEATURES "openmp")
##        endif()
##    endif()
##endif()
##
### Version is specified in vcpkg.json
##project(msdfgen VERSION ${MSDFGEN_VERSION} LANGUAGES CXX)
##
##file(GLOB_RECURSE MSDFGEN_CORE_HEADERS RELATIVE ${CMAKE_CURRENT_SOURCE_DIR} "core/*.h" "core/*.hpp")
##file(GLOB_RECURSE MSDFGEN_CORE_SOURCES RELATIVE ${CMAKE_CURRENT_SOURCE_DIR} "core/*.cpp")
##file(GLOB_RECURSE MSDFGEN_EXT_HEADERS RELATIVE ${CMAKE_CURRENT_SOURCE_DIR} "ext/*.h" "ext/*.hpp")
##file(GLOB_RECURSE MSDFGEN_EXT_SOURCES RELATIVE ${CMAKE_CURRENT_SOURCE_DIR} "ext/*.cpp" "lib/*.cpp")
##
### Core library
##add_library(msdfgen-core "${CMAKE_CURRENT_SOURCE_DIR}/msdfgen.h" ${MSDFGEN_CORE_HEADERS} ${MSDFGEN_CORE_SOURCES})
##add_library(msdfgen::msdfgen-core ALIAS msdfgen-core)
##set_target_properties(msdfgen-core PROPERTIES PUBLIC_HEADER "${MSDFGEN_CORE_HEADERS}")
##set_property(TARGET msdfgen-core PROPERTY MSVC_RUNTIME_LIBRARY "${MSDFGEN_MSVC_RUNTIME}")
##target_compile_definitions(msdfgen-core PUBLIC
##    MSDFGEN_VERSION=${MSDFGEN_VERSION}
##    MSDFGEN_VERSION_MAJOR=${MSDFGEN_VERSION_MAJOR}
##    MSDFGEN_VERSION_MINOR=${MSDFGEN_VERSION_MINOR}
##    MSDFGEN_VERSION_REVISION=${MSDFGEN_VERSION_REVISION}
##    MSDFGEN_COPYRIGHT_YEAR=${MSDFGEN_COPYRIGHT_YEAR}
##)
##target_include_directories(msdfgen-core INTERFACE
##    $<INSTALL_INTERFACE:include/msdfgen>
##    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/>
##)
##set_property(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR} PROPERTY VS_STARTUP_PROJECT msdfgen-core)
##
##if(MSDFGEN_USE_CPP11)
##    target_compile_features(msdfgen-core PUBLIC cxx_std_11)
##    target_compile_definitions(msdfgen-core PUBLIC MSDFGEN_USE_CPP11)
##endif()
##
##if(MSDFGEN_USE_OPENMP)
##    # Note: Clang doesn't support OpenMP by default...
##    find_package(OpenMP REQUIRED COMPONENTS CXX)
##    target_compile_definitions(msdfgen-core PUBLIC MSDFGEN_USE_OPENMP)
##    target_link_libraries(msdfgen-core PUBLIC OpenMP::OpenMP_CXX)
##endif()
##
##if(BUILD_SHARED_LIBS AND WIN32)
##    target_compile_definitions(msdfgen-core PRIVATE "MSDFGEN_PUBLIC=__declspec(dllexport)")
##    target_compile_definitions(msdfgen-core INTERFACE "MSDFGEN_PUBLIC=__declspec(dllimport)")
##else()
##    target_compile_definitions(msdfgen-core PUBLIC MSDFGEN_PUBLIC=)
##endif()
##
### Extensions library
##if(NOT MSDFGEN_CORE_ONLY)
##    if(NOT TARGET Freetype::Freetype)
##        find_package(Freetype REQUIRED)
##    endif()
##    if(NOT TARGET tinyxml2::tinyxml2)
##        find_package(tinyxml2 REQUIRED)
##    endif()
##    if(NOT TARGET PNG::PNG)
##        find_package(PNG REQUIRED)
##    endif()
##
##    add_library(msdfgen-ext "${CMAKE_CURRENT_SOURCE_DIR}/msdfgen-ext.h" ${MSDFGEN_EXT_HEADERS} ${MSDFGEN_EXT_SOURCES})
##    add_library(msdfgen::msdfgen-ext ALIAS msdfgen-ext)
##    set_target_properties(msdfgen-ext PROPERTIES PUBLIC_HEADER "${MSDFGEN_EXT_HEADERS}")
##    set_property(TARGET msdfgen-ext PROPERTY MSVC_RUNTIME_LIBRARY "${MSDFGEN_MSVC_RUNTIME}")
##    target_compile_definitions(msdfgen-ext PUBLIC MSDFGEN_USE_LIBPNG)
##    target_link_libraries(msdfgen-ext PRIVATE msdfgen::msdfgen-core Freetype::Freetype tinyxml2::tinyxml2 PNG::PNG)
##    target_include_directories(msdfgen-ext
##        PUBLIC
##            $<INSTALL_INTERFACE:include/msdfgen>
##            $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
##        PRIVATE
##            ${CMAKE_CURRENT_SOURCE_DIR}/include
##    )
##    set_property(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR} PROPERTY VS_STARTUP_PROJECT msdfgen-ext)
##
##    if(MSDFGEN_USE_SKIA)
##        set(MSDFGEN_SKIA_LIB skia)
##        set(CMAKE_THREAD_PREFER_PTHREAD TRUE)
##        set(THREADS_PREFER_PTHREAD_FLAG TRUE)
##        find_package(Threads REQUIRED)
##        if(NOT TARGET skia)
##            if(MSDFGEN_USE_VCPKG)
##                find_package(unofficial-skia REQUIRED)
##                set(MSDFGEN_SKIA_LIB unofficial::skia::skia)
##            else()
##                find_package(skia REQUIRED)
##            endif()
##        endif()
##        target_compile_features(msdfgen-ext PUBLIC cxx_std_17)
##        target_compile_definitions(msdfgen-ext PUBLIC MSDFGEN_USE_SKIA)
##        target_link_libraries(msdfgen-ext PRIVATE Threads::Threads ${MSDFGEN_SKIA_LIB})
##    endif()
##
##    add_library(msdfgen-full INTERFACE)
##    add_library(msdfgen::msdfgen ALIAS msdfgen-full)
##    target_link_libraries(msdfgen-full INTERFACE msdfgen::msdfgen-core msdfgen::msdfgen-ext)
##endif()
##
### Standalone executable
##if(MSDFGEN_BUILD_STANDALONE)
##    set(MSDFGEN_STANDALONE_SOURCES"${CMAKE_CURRENT_SOURCE_DIR}/main.cpp")
##    if(MSVC)
##        set(MSDFGEN_STANDALONE_SOURCES ${MSDFGEN_STANDALONE_SOURCES} "${CMAKE_CURRENT_SOURCE_DIR}/msdfgen.rc")
##    endif()
##    add_executable(msdfgen ${MSDFGEN_STANDALONE_SOURCES})
##    target_compile_definitions(msdfgen PUBLIC MSDFGEN_STANDALONE)
##    target_compile_definitions(msdfgen PRIVATE MSDFGEN_VERSION_UNDERLINE=${MSDFGEN_VERSION_UNDERLINE})
##    set_property(TARGET msdfgen PROPERTY MSVC_RUNTIME_LIBRARY "${MSDFGEN_MSVC_RUNTIME}")
##    target_link_libraries(msdfgen PRIVATE msdfgen::msdfgen)
##    set_property(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR} PROPERTY VS_STARTUP_PROJECT msdfgen)
##endif()
##
### Installation
##if(MSDFGEN_INSTALL)
##    include(GNUInstallDirs)
##    include(CMakePackageConfigHelpers)
##    set(MSDFGEN_CONFIG_PATH "lib/cmake/msdfgen")
##
##    # install tree package config
##    write_basic_package_version_file(
##        "${CMAKE_CURRENT_BINARY_DIR}/msdfgenConfigVersion.cmake"
##        VERSION ${PROJECT_VERSION}
##        COMPATIBILITY SameMajorVersion
##    )
##
##    configure_package_config_file(
##        cmake/msdfgenConfig.cmake.in
##        ${MSDFGEN_CONFIG_PATH}/msdfgenConfig.cmake
##        INSTALL_DESTINATION ${MSDFGEN_CONFIG_PATH}
##        NO_CHECK_REQUIRED_COMPONENTS_MACRO
##    )
##
##    # build tree package config
##    configure_file(
##        cmake/msdfgenConfig.cmake.in
##        msdfgenConfig.cmake
##        @ONLY
##    )
##
##    install(TARGETS msdfgen-core EXPORT msdfgenTargets
##        RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
##        LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
##        ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
##        FRAMEWORK DESTINATION ${CMAKE_INSTALL_LIBDIR}
##        PUBLIC_HEADER DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/msdfgen/core
##    )
##    install(FILES "${CMAKE_CURRENT_SOURCE_DIR}/msdfgen.h" DESTINATION include/msdfgen)
##    if(MSVC AND BUILD_SHARED_LIBS)
##        install(FILES $<TARGET_PDB_FILE:msdfgen-core> DESTINATION ${CMAKE_INSTALL_BINDIR} OPTIONAL)
##    endif()
##
##    if(NOT MSDFGEN_CORE_ONLY)
##        install(TARGETS msdfgen-ext EXPORT msdfgenTargets
##            RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
##            LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
##            ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
##            FRAMEWORK DESTINATION ${CMAKE_INSTALL_LIBDIR}
##            PUBLIC_HEADER DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/msdfgen/ext
##        )
##        install(FILES "${CMAKE_CURRENT_SOURCE_DIR}/msdfgen-ext.h" DESTINATION include/msdfgen)
##        if(MSVC AND BUILD_SHARED_LIBS)
##            install(FILES $<TARGET_PDB_FILE:msdfgen-ext> DESTINATION ${CMAKE_INSTALL_BINDIR} OPTIONAL)
##        endif()
##        install(TARGETS msdfgen-full EXPORT msdfgenTargets)
##    endif()
##
##    export(EXPORT msdfgenTargets NAMESPACE msdfgen:: FILE "${CMAKE_CURRENT_BINARY_DIR}/msdfgenTargets.cmake")
##    install(EXPORT msdfgenTargets FILE msdfgenTargets.cmake NAMESPACE msdfgen:: DESTINATION ${MSDFGEN_CONFIG_PATH})
##
##    if(MSDFGEN_BUILD_STANDALONE)
##        install(TARGETS msdfgen EXPORT msdfgenBinaryTargets DESTINATION ${CMAKE_INSTALL_BINDIR})
##        if(MSVC)
##            install(FILES $<TARGET_PDB_FILE:msdfgen> DESTINATION ${CMAKE_INSTALL_BINDIR} OPTIONAL)
##        endif()
##        export(EXPORT msdfgenBinaryTargets NAMESPACE msdfgen-standalone:: FILE "${CMAKE_CURRENT_BINARY_DIR}/msdfgenBinaryTargets.cmake")
##        install(EXPORT msdfgenBinaryTargets FILE msdfgenBinaryTargets.cmake NAMESPACE msdfgen-standalone:: DESTINATION ${MSDFGEN_CONFIG_PATH})
##    endif()
##
##    install(
##        FILES
##            "${CMAKE_CURRENT_BINARY_DIR}/${MSDFGEN_CONFIG_PATH}/msdfgenConfig.cmake"
##            "${CMAKE_CURRENT_BINARY_DIR}/msdfgenConfigVersion.cmake"
##        DESTINATION ${MSDFGEN_CONFIG_PATH}
##    )
##endif()

# Minimum version to run the script
##cmake_minimum_required(VERSION 3.16.0)
##
### Setting build type
##set(DEFAULT_BUILD_TYPE "Debug")
##if(NOT CMAKE_BUILD_TYPE AND NOT CMAKE_CONFIGURATION_TYPES)
##  message(STATUS "Setting build type to '${DEFAULT_BUILD_TYPE}' as none was specified.")
##  set(CMAKE_BUILD_TYPE "${DEFAULT_BUILD_TYPE}" CACHE
##      STRING "Choose the type of build." FORCE)
##  # Set the possible values of build type for cmake-gui
##  set_property(CACHE CMAKE_BUILD_TYPE PROPERTY STRINGS
##    "Debug" "Release" "MinSizeRel" "RelWithDebInfo")
##endif()
##
### Setting up the project name and version
##project(msdfgen
##        VERSION 2.26.0
##        DESCRIPTION ""
##        HOMEPAGE_URL ""
##        LANGUAGES CXX)
##
### Setting the install prefix
##set(CMAKE_INSTALL_PREFIX ${CMAKE_SOURCE_DIR})
##
### First for the generic no-config case (e.g. with mingw)
##set(EXECUTABLE_OUTPUT_PATH ${CMAKE_BINARY_DIR}/bin)
##set(LIBRARY_OUTPUT_PATH ${CMAKE_BINARY_DIR}/lib)
##set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)
##set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
##set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
### Second, for multi-config builds (e.g. msvc)
##foreach(OUTPUT_CONFIG ${CMAKE_CONFIGURATION_TYPES})
##    string( TOUPPER ${OUTPUT_CONFIG} OUTPUT_CONFIG )
##    set(CMAKE_RUNTIME_OUTPUT_DIRECTORY_${OUTPUT_CONFIG} ${CMAKE_BINARY_DIR}/bin)
##    set(CMAKE_LIBRARY_OUTPUT_DIRECTORY_${OUTPUT_CONFIG} ${CMAKE_BINARY_DIR}/lib)
##    set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY_${OUTPUT_CONFIG} ${CMAKE_BINARY_DIR}/lib)
##endforeach(OUTPUT_CONFIG CMAKE_CONFIGURATION_TYPES)
##
### Setting project options
##option(MSDF_ATLAS_GEN_MSVC_DYNAMIC_LINKED_RUNTIME "Uses MSVC dynamic linked runtime" OFF)
##
##add_subdirectory(freetype EXCLUDE_FROM_ALL)
##
### Include dirs
##set(MSDF_ATLAS_GEN_INCLUDE_DIRS
##    ${CMAKE_CURRENT_SOURCE_DIR}
##    ${CMAKE_CURRENT_SOURCE_DIR}/include
##    ${CMAKE_CURRENT_SOURCE_DIR}/core
##    ${CMAKE_CURRENT_SOURCE_DIR}/ext
##    ${CMAKE_CURRENT_SOURCE_DIR}/freetype/include
##    
##    
##)
##
### Header files
##set(MSDF_ATLAS_GEN_HEADERS
##
##)
##
### Source files
##set(MSDF_ATLAS_GEN_SOURCES
##    "main.cpp"
##    "core/Contour.cpp"
##    "core/EdgeHolder.cpp"
##    "core/MSDFErrorCorrection.cpp"
##    "core/Projection.cpp"
##    "core/Scanline.cpp"
##    "core/Shape.cpp"
##    "core/SignedDistance.cpp"
##    "core/Vector2.cpp"
##    "core/contour-combiners.cpp"
##    "core/edge-coloring.cpp"
##    "core/edge-segments.cpp"
##    "core/edge-selectors.cpp"
##    "core/equation-solver.cpp"
##    "core/msdf-error-correction.cpp"
##    "core/msdfgen.cpp"
##    "core/rasterization.cpp"
##    "core/render-sdf.cpp"
##    "core/save-bmp.cpp"
##    "core/save-tiff.cpp"
##    "core/sdf-error-estimation.cpp"
##    "core/shape-description.cpp"
##    
##    "ext/import-font.cpp"
##    "ext/import-svg.cpp"
##    "ext/resolve-shape-geometry.cpp"
##    "ext/save-png.cpp"
##)
##
### MSDF_ATLAS_GEN library
##add_library(msdfgen ${MSDF_ATLAS_GEN_SOURCES})
##
### Check for MSVC
##if(MSVC)
##    # Check for dynamic runtime
##    if(MSDF_ATLAS_GEN_MSVC_DYNAMIC_LINKED_RUNTIME)
##        set_property(TARGET msdfgen PROPERTY
##            MSVC_RUNTIME_LIBRARY "MultiThreaded$<$<CONFIG:Debug>:Debug>DLL"
##        )
##    else()
##        set_property(TARGET msdfgen PROPERTY
##            MSVC_RUNTIME_LIBRARY "MultiThreaded$<$<CONFIG:Debug>:Debug>"
##        )
##    endif()
##endif()
##
### MSDF_ATLAS_GEN include dirs
##target_include_directories(msdfgen
##PUBLIC
##    ${MSDF_ATLAS_GEN_INCLUDE_DIRS}
##
##)
##
##target_compile_definitions(msdfgen PUBLIC "MSDFGEN_PUBLIC=")
### target_compile_definitions(msdfgen PUBLIC "MSDFGEN_PUBLIC=__declspec(dllimport)")
##
##target_link_libraries(msdfgen 
##PUBLIC
##    freetype
##
##)
##
### Setting the install command to the app
##install(TARGETS msdf_atlas_gen RUNTIME DESTINATION bin)
##install(TARGETS msdf_atlas_gen LIBRARY DESTINATION lib)

cmake_minimum_required(VERSION 3.15)

option(MSDFGEN_CORE_ONLY "Only build the core library with no dependencies" OFF)
option(MSDFGEN_BUILD_STANDALONE "Build the msdfgen standalone executable" ON)
option(MSDFGEN_USE_VCPKG "Use vcpkg package manager to link project dependencies" OFF)
option(MSDFGEN_USE_OPENMP "Build with OpenMP support for multithreaded code" OFF)
option(MSDFGEN_USE_CPP11 "Build with C++11 enabled" ON)
option(MSDFGEN_USE_SKIA "Build with the Skia library" ON)
option(MSDFGEN_INSTALL "Generate installation target" ON)
option(MSDFGEN_DYNAMIC_RUNTIME "Link dynamic runtime library instead of static" OFF)
option(BUILD_SHARED_LIBS "Generate dynamic library files instead of static" OFF)

if(MSDFGEN_CORE_ONLY AND MSDFGEN_BUILD_STANDALONE)
    message(WARNING "Option MSDFGEN_CORE_ONLY ignored - extensions are required for standalone executable")
    set(MSDFGEN_CORE_ONLY OFF)
endif()
if(MSDFGEN_CORE_ONLY AND MSDFGEN_USE_VCPKG)
    message(STATUS "Option MSDFGEN_USE_VCPKG ignored due to MSDFGEN_CORE_ONLY - core has no dependencies")
    set(MSDFGEN_USE_VCPKG OFF)
endif()

get_property(MULTI_CONFIG GLOBAL PROPERTY GENERATOR_IS_MULTI_CONFIG)
if(NOT MULTI_CONFIG AND NOT CMAKE_BUILD_TYPE)
    message(STATUS "CMAKE_BUILD_TYPE not set, defaulting to Release")
    set(CMAKE_BUILD_TYPE Release)
endif()

if(MSDFGEN_DYNAMIC_RUNTIME)
    set(MSDFGEN_MSVC_RUNTIME "MultiThreaded$<$<CONFIG:Debug>:Debug>DLL")
else()
    set(MSDFGEN_MSVC_RUNTIME "MultiThreaded$<$<CONFIG:Debug>:Debug>")
endif()

if(BUILD_SHARED_LIBS)
    set(CMAKE_WINDOWS_EXPORT_ALL_SYMBOLS ON)
endif()

# Version is specified in vcpkg.json
# project(msdfgen 
        # VERSION 1.0.0 
        # LANGUAGES CXX)

file(GLOB_RECURSE MSDFGEN_CORE_HEADERS RELATIVE ${CMAKE_CURRENT_SOURCE_DIR} "core/*.h" "core/*.hpp")
file(GLOB_RECURSE MSDFGEN_CORE_SOURCES RELATIVE ${CMAKE_CURRENT_SOURCE_DIR} "core/*.cpp")
file(GLOB_RECURSE MSDFGEN_EXT_HEADERS RELATIVE ${CMAKE_CURRENT_SOURCE_DIR} "ext/*.h" "ext/*.hpp")
file(GLOB_RECURSE MSDFGEN_EXT_SOURCES RELATIVE ${CMAKE_CURRENT_SOURCE_DIR} "ext/*.cpp" "lib/*.cpp")

# Core library
add_library(msdfgen-core "${CMAKE_CURRENT_SOURCE_DIR}/msdfgen.h" ${MSDFGEN_CORE_HEADERS} ${MSDFGEN_CORE_SOURCES})
add_library(msdfgen::msdfgen-core ALIAS msdfgen-core)
set_target_properties(msdfgen-core PROPERTIES PUBLIC_HEADER "${MSDFGEN_CORE_HEADERS}")
set_property(TARGET msdfgen-core PROPERTY MSVC_RUNTIME_LIBRARY "${MSDFGEN_MSVC_RUNTIME}")
target_compile_definitions(msdfgen-core PUBLIC
    MSDFGEN_VERSION=1.0.0
    MSDFGEN_VERSION_MAJOR=1
    MSDFGEN_VERSION_MINOR=0
    MSDFGEN_VERSION_REVISION=0
    MSDFGEN_COPYRIGHT_YEAR=2023
)
target_include_directories(msdfgen-core INTERFACE
    $<INSTALL_INTERFACE:include/msdfgen>
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/>
)
set_property(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR} PROPERTY VS_STARTUP_PROJECT msdfgen-core)

if(MSDFGEN_USE_CPP11)
    target_compile_features(msdfgen-core PUBLIC cxx_std_11)
    target_compile_definitions(msdfgen-core PUBLIC MSDFGEN_USE_CPP11)
endif()

if(MSDFGEN_USE_OPENMP)
    # Note: Clang doesn't support OpenMP by default...
    find_package(OpenMP REQUIRED COMPONENTS CXX)
    target_compile_definitions(msdfgen-core PUBLIC MSDFGEN_USE_OPENMP)
    target_link_libraries(msdfgen-core PUBLIC OpenMP::OpenMP_CXX)
endif()

if(BUILD_SHARED_LIBS AND WIN32)
    target_compile_definitions(msdfgen-core PRIVATE "MSDFGEN_PUBLIC=__declspec(dllexport)")
    target_compile_definitions(msdfgen-core INTERFACE "MSDFGEN_PUBLIC=__declspec(dllimport)")
else()
    target_compile_definitions(msdfgen-core PUBLIC MSDFGEN_PUBLIC=)
endif()

add_subdirectory(freetype EXCLUDE_FROM_ALL)

# Extensions library
if(NOT MSDFGEN_CORE_ONLY)
    # if(NOT TARGET Freetype::Freetype)
        # find_package(Freetype REQUIRED)
    # endif()
    # if(NOT TARGET tinyxml2::tinyxml2)
        # find_package(tinyxml2 REQUIRED)
    # endif()
    # if(NOT TARGET PNG::PNG)
        # find_package(PNG REQUIRED)
    # endif()

    add_library(msdfgen-ext "${CMAKE_CURRENT_SOURCE_DIR}/msdfgen-ext.h" ${MSDFGEN_EXT_HEADERS} ${MSDFGEN_EXT_SOURCES})
    add_library(msdfgen::msdfgen-ext ALIAS msdfgen-ext)
    set_target_properties(msdfgen-ext PROPERTIES PUBLIC_HEADER "${MSDFGEN_EXT_HEADERS}")
    set_property(TARGET msdfgen-ext PROPERTY MSVC_RUNTIME_LIBRARY "${MSDFGEN_MSVC_RUNTIME}")
    
    target_compile_definitions(msdfgen-ext 
        PUBLIC 
            MSDFGEN_USE_LIBPNG
    )
    
    target_link_libraries(msdfgen-ext 
        PUBLIC
            freetype
            msdfgen::msdfgen-core 
            # tinyxml2::tinyxml2 
            # PNG::PNG
    )
    target_include_directories(msdfgen-ext
        PUBLIC
            $<INSTALL_INTERFACE:include/msdfgen>
            $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
        PRIVATE
            ${CMAKE_CURRENT_SOURCE_DIR}/include
            
    )
    set_property(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR} PROPERTY VS_STARTUP_PROJECT msdfgen-ext)

    if(MSDFGEN_USE_SKIA)
        set(MSDFGEN_SKIA_LIB skia)
        set(CMAKE_THREAD_PREFER_PTHREAD TRUE)
        set(THREADS_PREFER_PTHREAD_FLAG TRUE)
        find_package(Threads REQUIRED)
        if(NOT TARGET skia)
            if(MSDFGEN_USE_VCPKG)
                find_package(unofficial-skia REQUIRED)
                set(MSDFGEN_SKIA_LIB unofficial::skia::skia)
            else()
                find_package(skia REQUIRED)
            endif()
        endif()
        target_compile_features(msdfgen-ext PUBLIC cxx_std_17)
        target_compile_definitions(msdfgen-ext PUBLIC MSDFGEN_USE_SKIA)
        target_link_libraries(msdfgen-ext PUBLIC Threads::Threads ${MSDFGEN_SKIA_LIB})
    endif()

    add_library(msdfgen-full INTERFACE)
    add_library(msdfgen::msdfgen ALIAS msdfgen-full)
    target_link_libraries(msdfgen-full INTERFACE msdfgen::msdfgen-core msdfgen::msdfgen-ext)
endif()

# Standalone executable
if(MSDFGEN_BUILD_STANDALONE)
    set(MSDFGEN_STANDALONE_SOURCES "${CMAKE_CURRENT_SOURCE_DIR}/main.cpp")
    if(MSVC)
        set(MSDFGEN_STANDALONE_SOURCES ${MSDFGEN_STANDALONE_SOURCES} "${CMAKE_CURRENT_SOURCE_DIR}/msdfgen.rc")
    endif()
    add_executable(msdfgen ${MSDFGEN_STANDALONE_SOURCES})
    target_compile_definitions(msdfgen PUBLIC MSDFGEN_STANDALONE)
    target_compile_definitions(msdfgen PRIVATE MSDFGEN_VERSION_UNDERLINE=${MSDFGEN_VERSION_UNDERLINE})
    set_property(TARGET msdfgen PROPERTY MSVC_RUNTIME_LIBRARY "${MSDFGEN_MSVC_RUNTIME}")
    target_link_libraries(msdfgen PRIVATE msdfgen::msdfgen)
    set_property(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR} PROPERTY VS_STARTUP_PROJECT msdfgen)
endif()

# Installation
if(MSDFGEN_INSTALL)
    install(TARGETS msdfgen-core EXPORT msdfgenTargets
        RUNTIME DESTINATION bin 
        LIBRARY DESTINATION lib 
        ARCHIVE DESTINATION lib 
        FRAMEWORK DESTINATION lib 
        PUBLIC_HEADER DESTINATION msdfgen/core
    )
    install(FILES "${CMAKE_CURRENT_SOURCE_DIR}/msdfgen.h" DESTINATION include/msdfgen)
    if(MSVC AND BUILD_SHARED_LIBS)
        install(FILES $<TARGET_PDB_FILE:msdfgen-core> DESTINATION ${CMAKE_INSTALL_BINDIR} OPTIONAL)
    endif()

    if(NOT MSDFGEN_CORE_ONLY)
        install(TARGETS msdfgen-ext EXPORT msdfgenTargets
            RUNTIME DESTINATION bin 
            LIBRARY DESTINATION lib 
            ARCHIVE DESTINATION lib 
            FRAMEWORK DESTINATION lib 
            PUBLIC_HEADER DESTINATION msdfgen/ext
        )
        install(FILES "${CMAKE_CURRENT_SOURCE_DIR}/msdfgen-ext.h" DESTINATION include/msdfgen)
        if(MSVC AND BUILD_SHARED_LIBS)
            install(FILES $<TARGET_PDB_FILE:msdfgen-ext> DESTINATION ${CMAKE_INSTALL_BINDIR} OPTIONAL)
        endif()
        install(TARGETS msdfgen-full EXPORT msdfgenTargets)
    endif()

    export(EXPORT msdfgenTargets NAMESPACE msdfgen:: FILE "${CMAKE_CURRENT_BINARY_DIR}/msdfgenTargets.cmake")
    install(EXPORT msdfgenTargets FILE msdfgenTargets.cmake NAMESPACE msdfgen:: DESTINATION ${MSDFGEN_CONFIG_PATH})

    if(MSDFGEN_BUILD_STANDALONE)
        install(TARGETS msdfgen EXPORT msdfgenBinaryTargets DESTINATION ${CMAKE_INSTALL_BINDIR})
        if(MSVC)
            install(FILES $<TARGET_PDB_FILE:msdfgen> DESTINATION ${CMAKE_INSTALL_BINDIR} OPTIONAL)
        endif()
        export(EXPORT msdfgenBinaryTargets NAMESPACE msdfgen-standalone:: FILE "${CMAKE_CURRENT_BINARY_DIR}/msdfgenBinaryTargets.cmake")
        install(EXPORT msdfgenBinaryTargets FILE msdfgenBinaryTargets.cmake NAMESPACE msdfgen-standalone:: DESTINATION ${MSDFGEN_CONFIG_PATH})
    endif()

    install(
        FILES
            "${CMAKE_CURRENT_BINARY_DIR}/${MSDFGEN_CONFIG_PATH}/msdfgenConfig.cmake"
            "${CMAKE_CURRENT_BINARY_DIR}/msdfgenConfigVersion.cmake"
        DESTINATION ${MSDFGEN_CONFIG_PATH}
    )
endif()