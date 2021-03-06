# # # # sol2
# The MIT License (MIT)
# 
# Copyright (c) 2013-2017 Rapptz, ThePhD, and contributors
# 
# Permission is hereby granted, free of charge, to any person obtaining a copy of
# this software and associated documentation files (the "Software"), to deal in
# the Software without restriction, including without limitation the rights to
# use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of
# the Software, and to permit persons to whom the Software is furnished to do so,
# subject to the following conditions:
# 
# The above copyright notice and this permission notice shall be included in all
# copies or substantial portions of the Software.
# 
# THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
# IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS
# FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR
# COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER
# IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN
# CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.

# # # # sol2 tests

file(GLOB SOL2_TEST_SOURCES test*.cpp)
source_group(test_sources FILES ${SOL2_TEST_SOURCES})
file(DOWNLOAD https://github.com/catchorg/Catch2/releases/download/v2.0.1/catch.hpp ${CMAKE_BINARY_DIR}/vendor/Catch/include/catch.hpp)
#file(DOWNLOAD https://github.com/catchorg/Catch2/releases/download/v1.11.0/catch.hpp ${CMAKE_BINARY_DIR}/vendor/Catch/include/catch.hpp)

function(CREATE_TEST test_target_name test_name is_single)
	if (is_single)
		set(header_files ${SOL2_SINGLE_HEADER_SOURCES})
	else()
		set(header_files ${SOL2_HEADER_SOURCES})
	endif()

	add_executable(${test_target_name} ${SOL2_TEST_SOURCES} ${header_files})
	set_target_properties(${test_target_name}
		PROPERTIES
		OUTPUT_NAME ${test_name})
	if (is_single)
		target_link_libraries(${test_target_name} sol2_single)
	else()
		target_link_libraries(${test_target_name} sol2)
	endif()
	
	target_include_directories(${test_target_name}
		PRIVATE "${LUA_INCLUDE_DIR}" "${CMAKE_BINARY_DIR}/vendor/Catch/include/")
	if (MSVC)
		if (NOT CMAKE_COMPILER_ID MATCHES "Clang")
			target_compile_options(${test_target_name} 
				PRIVATE /bigobj)
		endif()
	else()
		target_compile_options(${test_target_name} 
			PRIVATE -Wno-noexcept-type -ftemplate-depth=1024 -pthread)
	endif()
	if (CI)
		target_compile_definitions(${test_target_name} 
			PRIVATE SOL2_CI)
	endif()
	target_compile_features(${test_target_name}
		PRIVATE ${CXX_FEATURES})
	# if CMAKE_DL_LIBS is empty, then that just resolves to nothingness
	target_link_libraries(${test_target_name}
		Threads::Threads ${LUA_LIBRARIES} ${CMAKE_DL_LIBS})

	add_test(NAME ${test_name} COMMAND ${test_target_name})
	install(TARGETS ${test_target_name} RUNTIME DESTINATION bin)
endfunction(CREATE_TEST)

CREATE_TEST(tests "tests" FALSE)
if (TESTS_SINGLE AND SOL2_SINGLE_FOUND)
	CREATE_TEST(tests_single "tests.single" TRUE)
endif()
