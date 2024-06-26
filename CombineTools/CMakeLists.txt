include_directories(${ROOT_INCLUDE_DIRS})
include_directories(BEFORE ${CMAKE_CURRENT_SOURCE_DIR})
include_directories(${Boost_INCLUDE_DIR})
include_directories(${CMSCombine_INCLUDE_DIR})
add_definitions(${ROOT_CXX_FLAGS})

set(LIBNAME CombineHarvesterCombineTools)

file(GLOB HEADERS RELATIVE ${CMAKE_CURRENT_SOURCE_DIR} interface/*.h*)
file(GLOB SOURCES RELATIVE ${CMAKE_CURRENT_SOURCE_DIR} src/*.c*)

# includes require "CombineHarvester/CombineTools" prefix in many places
# so create a symlink in the build dir
file(MAKE_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/CombineHarvester)
execute_process(COMMAND ${CMAKE_COMMAND} -E create_symlink "${CMAKE_CURRENT_SOURCE_DIR}" "${CMAKE_CURRENT_BINARY_DIR}/CombineHarvester/CombineTools" )
include_directories(${CMAKE_CURRENT_BINARY_DIR})

ROOT_GENERATE_DICTIONARY(G__${LIBNAME} CombineHarvester/CombineTools/src/classes.h LINKDEF src/classes_def.xml
        MODULE ${LIBNAME}
        OPTIONS --deep)
add_library(${LIBNAME} SHARED ${SOURCES} G__${LIBNAME}.cxx)
set_target_properties(${LIBNAME} PROPERTIES PUBLIC_HEADER "${HEADERS}")
target_link_libraries (${LIBNAME} Eigen3::Eigen ${ROOT_LIBRARIES} ${Boost_LIBRARIES} VDT::VDT CMSCombine::CMSCombine ZLIB::ZLIB)

set(EXE_SOURCES
        bin/Example1.cpp
        bin/Example2.cpp
        bin/Example3.cpp
        bin/GamGamExample.cpp
        bin/ImpactsTable.cpp
        bin/LimitCompare.cpp
        bin/MSSMExample.cpp
        bin/MSSMUpdate.cpp
        bin/MSSMYieldTable.cpp
        bin/MSSMtauptYieldTable.cpp
        bin/NuisanceSummary.cpp
        bin/Plot1DScan.cpp
        bin/PlotMassScan.cpp
        bin/PlotTest.cpp
        bin/PostFitPlot2.cpp
        bin/PostFitShapesFromWorkspace.cpp
        bin/PrePost.cpp
        bin/PrintPulls.cpp
        bin/RoundTrip.cpp
        bin/SBWeighted.cpp
        bin/SMLegacyExample.cpp
        bin/SOBPlot.cpp
        bin/YieldTable.cpp
        bin/hzz4l.cpp
)
foreach(exe_source ${EXE_SOURCES})
  get_filename_component(exe_name ${exe_source} NAME_WE)
  add_executable(${exe_name} ${exe_source})
  target_link_libraries(${exe_name} PUBLIC ${LIBNAME})
endforeach()

# Create empty __init__.py file in the build directory that will be installed
# in the Python library directories.
set(empty_init_py "${CMAKE_CURRENT_BINARY_DIR}/__init__.py")

# #####################
# # Installation part #
# #####################


# Install the dictionaries.
install(FILES ${CMAKE_CURRENT_BINARY_DIR}/lib${LIBNAME}_rdict.pcm
        ${CMAKE_CURRENT_BINARY_DIR}/lib${LIBNAME}.rootmap
        DESTINATION lib)

# Install the libraries and header files.
install(TARGETS ${LIBNAME}
        LIBRARY DESTINATION lib
        PUBLIC_HEADER DESTINATION include/CombineHarvester/CombineTools/interface
        )
# dictionary requires classes.h to be in include path exactly as it was specified
install(FILES ${CMAKE_CURRENT_BINARY_DIR}/CombineHarvester/CombineTools/src/classes.h
        DESTINATION include/CombineHarvester/CombineTools/src)

# Install the executables in the bin directory.
foreach(exe_source ${EXE_SOURCES})
  get_filename_component(exe_name ${exe_source} NAME_WE)
  install(TARGETS ${exe_name} DESTINATION bin)
endforeach()

# Install the scripts to the bin directory.
install(DIRECTORY scripts/ DESTINATION bin)

# This block is commented out for now, while using the less sophisticated location below
# Check if the Python library installation directory is outside the install
# prefix. If it is, we error out because CMake should not install files outside
# the prefix. In the future, one can imagine to let the user choose where the
# Python libraries get installed in the prefix with a CMake configuration flag.
#find_package(Python COMPONENTS Interpreter Development) # To get the Python library install directory into Python_SITELIB
#cmake_path(IS_PREFIX CMAKE_INSTALL_PREFIX "${Python_SITELIB}" sitelib_in_prefix)
#if(NOT ${sitelib_in_prefix})
#    message( FATAL_ERROR "Your Python library installation directory ${Python_SITELIB} "
#            "is outside the install prefix ${CMAKE_INSTALL_PREFIX}! "
#            "This is not supported for now. Consider changing the install prefix "
#            "with the -DCMAKE_INSTALL_PREFIX:PATH=<path> cmake configuration option.")
#endif()
#
## The the Python library installation directory relative to the install prefix.
#file(RELATIVE_PATH Python_SITELIB_IN_PREFIX ${CMAKE_INSTALL_PREFIX} ${Python_SITELIB})

set(Python_SITELIB_IN_PREFIX "python")


message (STATUS "Using Python install location:" ${Python_SITELIB_IN_PREFIX})
# The python package will be installed in such a way that the original
# CMSSW-style directory structure is kept, for maximal compatibility.
install(DIRECTORY python/ DESTINATION ${Python_SITELIB_IN_PREFIX}/CombineHarvester/CombineTools)

# Create empty __init__.py files in the Python package subdirectories such that
# the Python imports work.
file(TOUCH ${empty_init_py})
INSTALL(FILES ${empty_init_py} DESTINATION ${Python_SITELIB_IN_PREFIX}/CombineHarvester/CombineTools)
