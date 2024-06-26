cmake_minimum_required( VERSION 3.11 FATAL_ERROR )
set(CMAKE_CXX_STANDARD 17)
project(CMSCombineHarvester VERSION 0.0.1)

list(APPEND CMAKE_PREFIX_PATH $ENV{ROOTSYS})
find_package(ROOT REQUIRED COMPONENTS Core MathMore RooFitCore RooFit RooStats Minuit HistFactory RooFitHS3)
find_package(Eigen3 REQUIRED)
find_package(Vdt)
find_package(LCG QUIET) # only used for FindBoost in StatAnalysis
find_package(Boost REQUIRED COMPONENTS program_options filesystem)
find_package(ZLIB REQUIRED)
find_package(CMSCombine REQUIRED)

message(STATUS "Using ROOT From: ${ROOT_INCLUDE_DIRS}")
include(${ROOT_USE_FILE})

add_subdirectory(CombineTools)