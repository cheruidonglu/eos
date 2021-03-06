project(matlab-bindings)
cmake_minimum_required(VERSION 3.7.0)

# If Matlab_ROOT_DIR is set, the Matlab at that location is used.
find_package(Matlab COMPONENTS MX_LIBRARY REQUIRED)

# Dependencies of the modules:
find_package(OpenCV 2.4.3 REQUIRED core)
set_target_properties(${OpenCV_LIBS} PROPERTIES MAP_IMPORTED_CONFIG_RELWITHDEBINFO RELEASE)
if(MSVC)
	# The standard find_package for boost on Win finds the dynamic libs, so for dynamic linking to boost we need to #define:
	add_definitions(-DBOOST_ALL_NO_LIB) # Don't use the automatic library linking by boost with VS (#pragma ...). Instead, we specify everything here in cmake.
	add_definitions(-DBOOST_ALL_DYN_LINK) # Link against the dynamic boost lib - needs to match with the version that find_package finds.
endif()
find_package(Boost 1.50.0 COMPONENTS system filesystem REQUIRED) # Why do we need boost for MorphableModel.hpp?

# See: https://cmake.org/cmake/help/v3.7/module/FindMatlab.html?highlight=findmatlab#command:matlab_add_mex
matlab_add_mex(
    NAME eos_fitting
    #[EXECUTABLE | MODULE | SHARED] # SHARED is the default.
    SRC +eos/+fitting/private/fitting.cpp
    OUTPUT_NAME fitting #[OUTPUT_NAME output_name]
    # DOCUMENTATION +eos/+fitting/fit_shape_and_pose.m # doesn't work - wrong path probably. But it renames the file to fitting.m, so not what we want anyway.
    LINK_TO ${OpenCV_LIBS} ${Boost_LIBRARIES} #[LINK_TO target1 target2 ...]
    #[...]
)

target_include_directories(eos_fitting PRIVATE ${CMAKE_SOURCE_DIR}/3rdparty/mexplus/include ${CMAKE_SOURCE_DIR}/matlab/include)

install(FILES ${CMAKE_SOURCE_DIR}/matlab/demo.m DESTINATION matlab)
install(DIRECTORY ${CMAKE_SOURCE_DIR}/matlab/include DESTINATION matlab)
install(DIRECTORY ${CMAKE_SOURCE_DIR}/matlab/+eos DESTINATION matlab PATTERN "*.cpp" EXCLUDE)
install(TARGETS eos_fitting DESTINATION matlab/+eos/+fitting/private)
