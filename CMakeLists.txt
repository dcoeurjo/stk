cmake_minimum_required(VERSION 2.8)
cmake_policy(SET CMP0015 OLD)
list(APPEND CMAKE_MODULE_PATH "modules")

########################################################################

set(CAIRO_ENABLED 1)
set(CMAKE_SHARED_LIBRARY_CXX_FLAGS "${CMAKE_SHARED_LIBRARY_CXX_FLAGS} -framework Accelerate" )

project(stk)

set(LIBRARY_OUTPUT_PATH ./lib)
set(EXECUTABLE_OUTPUT_PATH .)
set(CMAKE_SHARED_MODULE_PREFIX)
set(STK_BIN_PATH ./bin)
set(STK_TEST_BIN_PATH ./unit-test/bin)
set(STK_MODULE_PATH /usr/local/share/stk)

file(MAKE_DIRECTORY ${STK_BIN_PATH})
file(MAKE_DIRECTORY ${STK_TEST_BIN_PATH})
file(MAKE_DIRECTORY lib/sampler)

set(CMAKE_BUILD_TYPE Release)  # None | Debug  | Release | RelWithDebInfo | MinSizeRel

find_package(PNG REQUIRED)
find_package(Boost 1.42.0 COMPONENTS thread program_options unit_test_framework timer REQUIRED)
find_package(FFTW REQUIRED)
find_package(CGAL REQUIRED COMPONENTS Core)
#find_package(OpenMP REQUIRED)

if(CAIRO_ENABLED)
	find_package(Cairo REQUIRED)
	add_definitions(-DCAIRO_ENABLED)
endif(CAIRO_ENABLED)

add_definitions(-DDIR_MODULE="${STK_MODULE_PATH}/")
set(CMAKE_CXX_FLAGS ${CMAKE_CXX_FLAGS} -frounding-math)


include_directories(
	./src
	${PNG_INCLUDE_DIR}
	${Boost_INCLUDE_DIR}
	${FFTW_INCLUDE_DIR}
	${CGAL_INCLUDE_DIRS}
	${CGAL_3RD_PARTY_INCLUDE_DIRS}
    /opt/local/include/cairo/
    /usr/include/suitesparse
)

link_directories(
	./lib
	${PNG_LIBRARY}
	${Boost_LIBRARY_DIRS}
	${FFTW_INCLUDE_DIR}
	${CGAL_LIBRARIES_DIR}
	${CGAL_3RD_PARTY_LIBRARIES_DIRS}
	${CAIRO_LIBRARY_DIRS}
)

########################################################################
# SOURCES
########################################################################

file(GLOB stk_sources
	src/stk/*.cpp
	src/stk/conversion/*.cpp
	src/stk/diff-space/*.cpp
	src/stk/fourier/*.cpp
	src/stk/function/*.cpp
	src/stk/io/*.cpp
	src/stk/optimization/*.cpp
	src/stk/sampler/*.cpp
	src/stk/unstable/*.cpp
)

file(GLOB stk_headers src/stk/*.hpp)
file(GLOB stk_headers_conversion src/stk/conversion/*.hpp)
file(GLOB stk_headers_diff-space src/stk/diff-space/*.hpp)
file(GLOB stk_headers_fourier src/stk/fourier/*.hpp)
file(GLOB stk_headers_function src/stk/function/*.hpp)
file(GLOB stk_headers_io src/stk/io/*.hpp)
file(GLOB stk_headers_optimization src/stk/optimization/*.hpp)
file(GLOB stk_headers_sampler src/stk/sampler/*.hpp)
file(GLOB stk_headers_unstable src/stk/unstable/*.hpp)

########################################################################
# LIBRARIES
########################################################################

add_library(stk SHARED ${stk_sources})
add_library(stk_static STATIC ${stk_sources})

target_link_libraries(
	stk
	${CGAL_LIBRARIES} -lgmp
	${CGAL_3RD_PARTY_LIBRARIES}
	${PNG_LIBRARIES}
	${FFTW_LIBRARIES}
	${CAIRO_LIBRARIES}
)
target_link_libraries(
	stk_static
	${CGAL_LIBRARIES} -lgmp
	${CGAL_3RD_PARTY_LIBRARIES}
	${PNG_LIBRARIES}
	${FFTW_LIBRARIES}
	${CAIRO_LIBRARIES}
)

set_target_properties(stk PROPERTIES VERSION 0.1)
set_target_properties(stk PROPERTIES LINKER_LANGUAGE CXX)
set_target_properties(stk_static PROPERTIES VERSION 0.1)
set_target_properties(stk_static PROPERTIES LINKER_LANGUAGE CXX)

install(
	TARGETS stk stk_static
	DESTINATION lib
)

install(FILES ${stk_headers}
	DESTINATION /usr/local/include/stk)
install(FILES ${stk_headers_conversion}
	DESTINATION /usr/local/include/stk/conversion)
install(FILES ${stk_headers_fourier}
	DESTINATION /usr/local/include/stk/fourier)
install(FILES ${stk_headers_function}
	DESTINATION /usr/local/include/stk/function)
install(FILES ${stk_headers_io}
	DESTINATION /usr/local/include/stk/io)
install(FILES ${stk_headers_optimization}
	DESTINATION /usr/local/include/stk/optimization)
install(FILES ${stk_headers_sampler}
	DESTINATION /usr/local/include/stk/sampler)
install(FILES ${stk_headers_unstable}
	DESTINATION /usr/local/include/stk/unstable)

install(FILES data/cmake/FindSTK.cmake
	DESTINATION /usr/share/cmake-2.8/Modules)

if(CUDA_ENABLED)
	install(FILES data/cmake/enabled/STKCuda.cmake
		DESTINATION /usr/local/share/stk/cmake)
else(CUDA_ENABLED)
	install(FILES data/cmake/disabled/STKCuda.cmake
		DESTINATION /usr/local/share/stk/cmake)
endif(CUDA_ENABLED)

if(CAIRO_ENABLED)
	install(FILES data/cmake/enabled/STKCairo.cmake
		DESTINATION /usr/local/share/stk/cmake)
else(CAIRO_ENABLED)
	install(FILES data/cmake/disabled/STKCairo.cmake
		DESTINATION /usr/local/share/stk/cmake)
endif(CAIRO_ENABLED)

########################################################################
# EXECUTABLES
########################################################################

function(stk_make_program prog_name prog_libs)
	add_executable(${STK_BIN_PATH}/stk-${prog_name} src/${prog_name}.cpp)
	target_link_libraries(
		${STK_BIN_PATH}/stk-${prog_name}
		stk
		${Boost_LIBRARIES}
		${prog_libs}
	)
	install(PROGRAMS ${STK_BIN_PATH}/stk-${prog_name} DESTINATION bin)
endfunction()

stk_make_program(fourier "")
stk_make_program(integration-test "")
stk_make_program(pts-info "")
#~ stk_make_program(checkerboard "")
#~ stk_make_program(zoneplate "")
#~ stk_make_program(zoneplate2 "")
#~ stk_make_program(zoneplate-variance -ldl)
#~ stk_make_program(lloyd "")
#~ stk_make_program(convert "")
#~ stk_make_program(bpts "")
#~ stk_make_program(ddf "")
#~ stk_make_program(integration -ldl)
#~ stk_make_program(discrepancy "")
#~ stk_make_program(discrepancy3d "")
#~ stk_make_program(voronoi "")
#~ stk_make_program(capacity "")
#~ stk_make_program(cdf "")
#~ stk_make_program(discrepancy4d "")

#~ stk_make_program(pts-merge "")
#~ stk_make_program(pts-take "")

if(CAIRO_ENABLED)
	stk_make_program(draw-pts "")
	#~ stk_make_program(draw-histo "")
endif(CAIRO_ENABLED)

########################################################################
# SAMPLER
########################################################################

stk_make_program(sampler-whitenoiseT2 "")
stk_make_program(sampler-stratifiedT2 "")
stk_make_program(sampler-sobolT2 "")
stk_make_program(sampler-haltonT2 "")

add_executable(${STK_BIN_PATH}/stk-sampler-fpoT2 src/sampler-fpo/sampler-fpoT2.cpp src/sampler-fpo/DT.cpp)
target_link_libraries(
	${STK_BIN_PATH}/stk-sampler-fpoT2
	stk
	${Boost_LIBRARIES}
	${prog_libs}
)
install(PROGRAMS ${STK_BIN_PATH}/stk-sampler-fpoT2 DESTINATION bin)

#~ add_library(sampler/bnot MODULE
#~ add_executable(${STK_BIN_PATH}/bnot
	#~ src/sampler/bnot/bnot.cpp
	#~ src/sampler/bnot/matrix/sparse_array.cpp
	#~ src/sampler/bnot/matrix/sparse_matrix.cpp
	#~ src/sampler/bnot/energy.cpp
	#~ src/sampler/bnot/init.cpp
	#~ src/sampler/bnot/optimizer.cpp
	#~ src/sampler/bnot/regularity.cpp
	#~ src/sampler/bnot/sites.cpp)
	
#~ target_link_libraries(
	#~ sampler/bnot
	#~ ${CGAL_LIBRARIES} -lgmp
	#~ -lCGAL -lCGAL_Core 
	#~ stk
	#~ -lm -lamd -lcamd -lcolamd -lccolamd -lcholmod -lspqr -ltbb -lmetis -lblas -llapack
#~ )
#~ install(TARGETS sampler/bnot DESTINATION ${STK_MODULE_PATH}/samplers)


########################################################################
# UNIT TEST
########################################################################

enable_testing()

function(stk_make_test test_name)
	add_executable(${STK_TEST_BIN_PATH}/test-${test_name} unit-test/src/${test_name}.cpp)
	target_link_libraries(
		${STK_TEST_BIN_PATH}/test-${test_name}
		stk
		${Boost_LIBRARIES}
		${Boost_UNIT_TEST_FRAMEWORK_LIBRARY_DEBUG}
	)
	add_test(test-${test_name} ${STK_TEST_BIN_PATH}/test-${test_name})
endfunction()

#~ stk_make_test(vector)
#~ stk_make_test(domain)
#~ stk_make_test(pointset)
#~ stk_make_test(pointset-stream)
#~ stk_make_test(array)
