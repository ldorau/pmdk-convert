#
# Copyright 2018-2020, Intel Corporation
#
# Redistribution and use in source and binary forms, with or without
# modification, are permitted provided that the following conditions
# are met:
#
#     * Redistributions of source code must retain the above copyright
#       notice, this list of conditions and the following disclaimer.
#
#     * Redistributions in binary form must reproduce the above copyright
#       notice, this list of conditions and the following disclaimer in
#       the documentation and/or other materials provided with the
#       distribution.
#
#     * Neither the name of the copyright holder nor the names of its
#       contributors may be used to endorse or promote products derived
#       from this software without specific prior written permission.
#
# THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS
# "AS IS" AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT
# LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR
# A PARTICULAR PURPOSE ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT
# OWNER OR CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL,
# SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT
# LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE,
# DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY
# THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT
# (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
# OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.

include(${CMAKE_SOURCE_DIR}/utils/utils.cmake)

set(TEST_DIR ${CMAKE_CURRENT_BINARY_DIR}/test
	CACHE STRING "working directory for tests")

option(TESTS_USE_FORCED_PMEM "let tests force enable or force disable use of optimized flush in libpmemobj (to speed them up)" OFF)

set(DEVICE_DAX_PATHS "" CACHE STRING
	"for tests that require raw dax devices without a file system. Some tests might require two DAX devices.
	Example: '/dev/dax1.0 /dev/dax2.0'")

if(WIN32)
	set(CDB_PATH "" CACHE PATH "requied for windows tests with cdb debugger")
endif()

set(GLOBAL_TEST_ARGS
	-DPARENT_DIR=${TEST_DIR}/
	-DTESTS_USE_FORCED_PMEM=${TESTS_USE_FORCED_PMEM}
	-DDEVICE_DAX_PATHS=${DEVICE_DAX_PATHS}
	-DCDB_PATH=${CDB_PATH})

# convert the DEVICE_DAX_PATHS list to the array
if(DEVICE_DAX_PATHS)
	string(REPLACE " " ";" DEVICE_DAX_PATHS ${DEVICE_DAX_PATHS})
endif()

list(LENGTH DEVICE_DAX_PATHS devdax_num)

if(TRACE_TESTS)
	set(GLOBAL_TEST_ARGS ${GLOBAL_TEST_ARGS} --trace-expand)
endif()

# Configures test ${name}
function(test name file)
	add_test(NAME ${name}
		COMMAND ${CMAKE_COMMAND}
		${GLOBAL_TEST_ARGS}
		-DTEST_NAME=${name}
		-DSRC_DIR=${CMAKE_CURRENT_SOURCE_DIR}
		-DBIN_DIR=${CMAKE_CURRENT_BINARY_DIR}/${file}
		-DVERSIONS=${VERSIONS}
		-DCONFIG=$<CONFIG>
		-P ${CMAKE_CURRENT_SOURCE_DIR}/${file}.cmake)

	set_tests_properties(${name} PROPERTIES
		ENVIRONMENT "LC_ALL=C;PATH=$ENV{PATH}")
endfunction()

# Finding cdb executable debugger file and setting path to it
function(set_cdb_executable)
	if(EXISTS ${CDB_PATH})
		find_program(CDB_EXE cdb.exe ${CDB_PATH})
		if(NOT CDB_EXE)
			message(WARNING "CDB Debugger not found!")
		endif()
	else()
		unset(CDB_EXE)
	endif()
endfunction()
if(WIN32)
	set(OPEN_SOURCE openW.c)
	set(CREATE_SOURCE createW.c)
else()
	set(OPEN_SOURCE open.c)
	set(CREATE_SOURCE create.c)
endif()

if(BUILD_CONVERT_V1)
	set(VERSIONS "${VERSIONS} 1.0")
endif()
if(BUILD_CONVERT_V2)
	set(VERSIONS "${VERSIONS} 1.1")
endif()
if(BUILD_CONVERT_V3)
	set(VERSIONS "${VERSIONS} 1.2")
endif()
if(BUILD_13)
	set(VERSIONS "${VERSIONS} 1.3")
endif()
if(BUILD_CONVERT_V4)
	set(VERSIONS "${VERSIONS} 1.4")
endif()
if(BUILD_15)
	set(VERSIONS "${VERSIONS} 1.5")
endif()
if(BUILD_CONVERT_V5)
	set(VERSIONS "${VERSIONS} 1.6")
endif()
if(BUILD_17)
	set(VERSIONS "${VERSIONS} 1.7")
endif()
if(BUILD_CONVERT_V6)
	set(VERSIONS "${VERSIONS} 1.8")
endif()

if(BUILD_CONVERT_V1)
	add_executable(create_10 ${CREATE_SOURCE})
	target_include_directories(create_10 PRIVATE ${CMAKE_SOURCE_DIR}/${VER10}/src/include)
	target_link_libraries(create_10 PRIVATE libpmem-convert)
	target_link_libraries(create_10 PRIVATE pmemobj_10)

	add_executable(open_10 open.c)
	target_include_directories(open_10 PRIVATE ${CMAKE_SOURCE_DIR}/${VER10}/src/include)
	target_link_libraries(open_10 PRIVATE libpmem-convert)
	target_link_libraries(open_10 PRIVATE pmemobj_10)

	test("1.0_pool_conversion" basic10)
endif()

if(BUILD_CONVERT_V2)
	add_executable(create_11 ${CREATE_SOURCE})
	target_include_directories(create_11 PRIVATE ${CMAKE_SOURCE_DIR}/${VER11}/src/include)
	target_link_libraries(create_11 PRIVATE libpmem-convert)
	target_link_libraries(create_11 PRIVATE pmemobj_11)

	add_executable(open_11 ${OPEN_SOURCE})
	target_include_directories(open_11 PRIVATE ${CMAKE_SOURCE_DIR}/${VER11}/src/include)
	target_link_libraries(open_11 PRIVATE libpmem-convert)
	target_link_libraries(open_11 PRIVATE pmemobj_11)

	test("1.1_pool_conversion" basic11)
endif()

if(BUILD_CONVERT_V3)
	add_executable(create_12 ${CREATE_SOURCE})
	target_include_directories(create_12 PRIVATE ${CMAKE_SOURCE_DIR}/${VER12}/src/include)
	target_link_libraries(create_12 PRIVATE libpmem-convert)
	target_link_libraries(create_12 PRIVATE pmemobj_12)

	add_executable(open_12 ${OPEN_SOURCE})
	target_include_directories(open_12 PRIVATE ${CMAKE_SOURCE_DIR}/${VER12}/src/include)
	target_link_libraries(open_12 PRIVATE libpmem-convert)
	target_link_libraries(open_12 PRIVATE pmemobj_12)

	test("1.2_pool_conversion" basic12)
endif()

if(BUILD_13)
	add_executable(create_13 ${CREATE_SOURCE})
	target_include_directories(create_13 PRIVATE ${CMAKE_SOURCE_DIR}/${VER13}/src/include)
	target_link_libraries(create_13 PRIVATE libpmem-convert)
	target_link_libraries(create_13 PRIVATE pmemobj_13)

	add_executable(open_13 ${OPEN_SOURCE})
	target_include_directories(open_13 PRIVATE ${CMAKE_SOURCE_DIR}/${VER13}/src/include)
	target_link_libraries(open_13 PRIVATE libpmem-convert)
	target_link_libraries(open_13 PRIVATE pmemobj_13)

	test("1.3_pool_conversion" basic13)

	if(WIN32)
		# on windows dll and executable must be in the same dir
		add_custom_command(TARGET open_13 POST_BUILD
			COMMAND ${CMAKE_COMMAND} -E copy_if_different
			$<TARGET_FILE:pmemobj_13>
			$<TARGET_FILE_DIR:open_13>)
	endif()
endif()

if(BUILD_CONVERT_V4)
	add_executable(create_14 ${CREATE_SOURCE})
	target_include_directories(create_14 PRIVATE ${CMAKE_SOURCE_DIR}/${VER14}/src/include)
	target_link_libraries(create_14 PRIVATE libpmem-convert)
	target_link_libraries(create_14 PRIVATE pmemobj_14)

	add_executable(open_14 ${OPEN_SOURCE})
	target_include_directories(open_14 PRIVATE ${CMAKE_SOURCE_DIR}/${VER14}/src/include)
	target_link_libraries(open_14 PRIVATE libpmem-convert)
	target_link_libraries(open_14 PRIVATE pmemobj_14)

	test("1.4_pool_conversion" basic14)

	if(WIN32)
		# on windows dll and executable must be in the same dir
		add_custom_command(TARGET open_14 POST_BUILD
			COMMAND ${CMAKE_COMMAND} -E copy_if_different
			$<TARGET_FILE:pmemobj_14>
			$<TARGET_FILE_DIR:open_14>)
	endif()
endif()

if(BUILD_15)
	add_executable(create_15 ${CREATE_SOURCE})
	target_include_directories(create_15 PRIVATE ${CMAKE_SOURCE_DIR}/${VER15}/src/include)
	target_link_libraries(create_15 PRIVATE libpmem-convert)
	target_link_libraries(create_15 PRIVATE pmemobj_15)

	add_executable(open_15 ${OPEN_SOURCE})
	target_include_directories(open_15 PRIVATE ${CMAKE_SOURCE_DIR}/${VER15}/src/include)
	target_link_libraries(open_15 PRIVATE libpmem-convert)
	target_link_libraries(open_15 PRIVATE pmemobj_15)

	test("1.5_pool_conversion" basic15)

	if(WIN32)
		# on windows dll and executable must be in the same dir
		add_custom_command(TARGET open_15 POST_BUILD
			COMMAND ${CMAKE_COMMAND} -E copy_if_different
			$<TARGET_FILE:pmemobj_15>
			$<TARGET_FILE_DIR:open_15>)
	endif()
endif()

if(BUILD_CONVERT_V5)
	add_executable(create_16 ${CREATE_SOURCE})
	target_include_directories(create_16 PRIVATE ${CMAKE_SOURCE_DIR}/${VER16}/src/include)
	target_link_libraries(create_16 PRIVATE libpmem-convert)
	target_link_libraries(create_16 PRIVATE pmemobj_16)

	add_executable(open_16 ${OPEN_SOURCE})
	target_include_directories(open_16 PRIVATE ${CMAKE_SOURCE_DIR}/${VER16}/src/include)
	target_link_libraries(open_16 PRIVATE libpmem-convert)
	target_link_libraries(open_16 PRIVATE pmemobj_16)

	test("1.6_pool_conversion" basic16)

	if(WIN32)
		# on windows dll and executable must be in the same dir
		add_custom_command(TARGET open_16 POST_BUILD
			COMMAND ${CMAKE_COMMAND} -E copy_if_different
			$<TARGET_FILE:pmemobj_16>
			$<TARGET_FILE_DIR:open_16>)

		add_custom_command(TARGET open_16 POST_BUILD
			COMMAND ${CMAKE_COMMAND} -E copy_if_different
			$<TARGET_FILE_DIR:pmemobj_16>/libpmem-convert.dll
			$<TARGET_FILE_DIR:open_16>)
	endif()
endif()

if(BUILD_17)
	add_executable(create_17 ${CREATE_SOURCE})
	target_include_directories(create_17 PRIVATE ${CMAKE_SOURCE_DIR}/${VER17}/src/include)
	target_link_libraries(create_17 PRIVATE libpmem-convert)
	target_link_libraries(create_17 PRIVATE pmemobj_17)

	add_executable(open_17 ${OPEN_SOURCE})
	target_include_directories(open_17 PRIVATE ${CMAKE_SOURCE_DIR}/${VER17}/src/include)
	target_link_libraries(open_17 PRIVATE libpmem-convert)
	target_link_libraries(open_17 PRIVATE pmemobj_17)

	test("1.7_pool_conversion" basic17)

	if(WIN32)
		# on windows dll and executable must be in the same dir
		add_custom_command(TARGET open_17 POST_BUILD
			COMMAND ${CMAKE_COMMAND} -E copy_if_different
			$<TARGET_FILE:pmemobj_17>
			$<TARGET_FILE_DIR:open_17>)

		add_custom_command(TARGET open_17 POST_BUILD
			COMMAND ${CMAKE_COMMAND} -E copy_if_different
			$<TARGET_FILE_DIR:pmemobj_17>/libpmem-convert.dll
			$<TARGET_FILE_DIR:open_17>)
	endif()
endif()

if(BUILD_CONVERT_V6)
	add_executable(create_18 ${CREATE_SOURCE})
	target_include_directories(create_18 PRIVATE ${CMAKE_SOURCE_DIR}/${VER18}/src/include)
	target_link_libraries(create_18 PRIVATE libpmem-convert)
	target_link_libraries(create_18 PRIVATE pmemobj_18)

	add_executable(open_18 ${OPEN_SOURCE})
	target_include_directories(open_18 PRIVATE ${CMAKE_SOURCE_DIR}/${VER18}/src/include)
	target_link_libraries(open_18 PRIVATE libpmem-convert)
	target_link_libraries(open_18 PRIVATE pmemobj_18)

	if(WIN32)
		# on windows dll and executable must be in the same dir
		add_custom_command(TARGET open_18 POST_BUILD
			COMMAND ${CMAKE_COMMAND} -E copy_if_different
			$<TARGET_FILE:pmemobj_18>
			$<TARGET_FILE_DIR:open_18>)

		add_custom_command(TARGET open_18 POST_BUILD
			COMMAND ${CMAKE_COMMAND} -E copy_if_different
			$<TARGET_FILE_DIR:pmemobj_18>/libpmem-convert.dll
			$<TARGET_FILE_DIR:open_18>)
	endif()

	set(COMMON_SOURCES
		${CMAKE_SOURCE_DIR}/${MAX_STABLE}/src/common/alloc.c
		${CMAKE_SOURCE_DIR}/${MAX_STABLE}/src/common/badblock.c
		${CMAKE_SOURCE_DIR}/${MAX_STABLE}/src/common/file.c
		${CMAKE_SOURCE_DIR}/${MAX_STABLE}/src/common/mmap.c
		${CMAKE_SOURCE_DIR}/${MAX_STABLE}/src/common/pool_hdr.c
		${CMAKE_SOURCE_DIR}/${MAX_STABLE}/src/common/out.c
		${CMAKE_SOURCE_DIR}/${MAX_STABLE}/src/common/os_dimm_none.c
		${CMAKE_SOURCE_DIR}/${MAX_STABLE}/src/common/set.c
		${CMAKE_SOURCE_DIR}/${MAX_STABLE}/src/common/shutdown_state.c
		${CMAKE_SOURCE_DIR}/${MAX_STABLE}/src/common/util.c
		${CMAKE_SOURCE_DIR}/${MAX_STABLE}/src/common/uuid.c
		${CMAKE_SOURCE_DIR}/${MAX_STABLE}/src/common/ctl_cow.c
		${CMAKE_SOURCE_DIR}/${MAX_STABLE}/src/common/ctl_fallocate.c
		${CMAKE_SOURCE_DIR}/${MAX_STABLE}/src/common/ctl_sds.c
		${CMAKE_SOURCE_DIR}/${MAX_STABLE}/src/common/ctl.c
		${CMAKE_SOURCE_DIR}/${MAX_STABLE}/src/common/ctl_prefault.c
		)

	if(WIN32)
		set(COMMON_SOURCES ${COMMON_SOURCES}
			${CMAKE_SOURCE_DIR}/${MAX_STABLE}/src/common/badblock_none.c
			${CMAKE_SOURCE_DIR}/${MAX_STABLE}/src/common/file_windows.c
			${CMAKE_SOURCE_DIR}/${MAX_STABLE}/src/common/fs_windows.c
			${CMAKE_SOURCE_DIR}/${MAX_STABLE}/src/common/mmap_windows.c
			${CMAKE_SOURCE_DIR}/${MAX_STABLE}/src/common/os_deep_windows.c
			${CMAKE_SOURCE_DIR}/${MAX_STABLE}/src/common/os_thread_windows.c
			${CMAKE_SOURCE_DIR}/${MAX_STABLE}/src/common/os_windows.c
			${CMAKE_SOURCE_DIR}/${MAX_STABLE}/src/common/util_windows.c
			${CMAKE_SOURCE_DIR}/${MAX_STABLE}/src/common/uuid_windows.c
			)
	else()
		set(COMMON_SOURCES ${COMMON_SOURCES}
			${CMAKE_SOURCE_DIR}/${MAX_STABLE}/src/common/badblock_ndctl.c
			${CMAKE_SOURCE_DIR}/${MAX_STABLE}/src/common/extent_linux.c
			${CMAKE_SOURCE_DIR}/${MAX_STABLE}/src/common/file_posix.c
			${CMAKE_SOURCE_DIR}/${MAX_STABLE}/src/common/fs_posix.c
			${CMAKE_SOURCE_DIR}/${MAX_STABLE}/src/common/mmap_posix.c
			${CMAKE_SOURCE_DIR}/${MAX_STABLE}/src/common/os_deep_linux.c
			${CMAKE_SOURCE_DIR}/${MAX_STABLE}/src/common/os_posix.c
			${CMAKE_SOURCE_DIR}/${MAX_STABLE}/src/common/os_thread_posix.c
			${CMAKE_SOURCE_DIR}/${MAX_STABLE}/src/common/util_posix.c
			${CMAKE_SOURCE_DIR}/${MAX_STABLE}/src/common/uuid_linux.c
			)
	endif()

	function(build_libpmemblk_18)
		set(BLK_SOURCES
			${CMAKE_SOURCE_DIR}/${MAX_STABLE}/src/libpmemblk/blk.c
			${CMAKE_SOURCE_DIR}/${MAX_STABLE}/src/libpmemblk/btt.c
			${CMAKE_SOURCE_DIR}/${MAX_STABLE}/src/libpmemblk/libpmemblk.c
			${COMMON_SOURCES}
			)

		if(WIN32)
			set(BLK_SOURCES ${BLK_SOURCES}
				${CMAKE_SOURCE_DIR}/${MAX_STABLE}/src/libpmemblk/libpmemblk_main.c
				)
		endif()

		if(MSVC)
			FILE(READ ${CMAKE_SOURCE_DIR}/${MAX_STABLE}/src/libpmemblk/libpmemblk.def DEFFILE)
			string(REPLACE "libpmemblk" "pmemblk_18" DEFFILE "${DEFFILE}")
			FILE(WRITE ${CMAKE_CURRENT_BINARY_DIR}/pmemblk_18.def "${DEFFILE}")
			add_library(pmemblk_18 SHARED ${BLK_SOURCES} ${CMAKE_CURRENT_BINARY_DIR}/pmemblk_18.def)
			target_compile_options(pmemblk_18 PRIVATE /FI"${CMAKE_SOURCE_DIR}/${VER18}/src/windows/include/platform.h")
		else()
			add_library(pmemblk_18 SHARED ${BLK_SOURCES})
		endif()

		target_compile_definitions(pmemblk_18 PRIVATE SRCVERSION="${MAX_STABLE}")
		target_compile_definitions(pmemblk_18 PRIVATE PMDK_UTF8_API)
		target_compile_definitions(pmemblk_18 PRIVATE _PMEMBLK_INTRNL)

		target_include_directories(pmemblk_18 PRIVATE ${CMAKE_SOURCE_DIR}/${MAX_STABLE}/src/include)
		target_include_directories(pmemblk_18 PRIVATE ${CMAKE_SOURCE_DIR}/${MAX_STABLE}/src/libpmemblk)
		target_include_directories(pmemblk_18 PRIVATE ${CMAKE_SOURCE_DIR}/${MAX_STABLE}/src/common)
		target_include_directories(pmemblk_18 PRIVATE ${PMEM_INCLUDE_DIRS})

		if(WIN32)
			target_include_directories(pmemblk_18 PRIVATE ${CMAKE_SOURCE_DIR}/${MAX_STABLE}/src/windows/include/)
			target_include_directories(pmemblk_18 PRIVATE ${CMAKE_SOURCE_DIR}/include/)
		endif()

		target_link_libraries(pmemblk_18 PRIVATE libpmem-convert)
		target_link_libraries(pmemblk_18 PRIVATE ${CMAKE_DL_LIBS})
		target_link_libraries(pmemblk_18 PRIVATE ${CMAKE_THREAD_LIBS_INIT})

		if(NOT MSVC)
			FILE(READ ${CMAKE_SOURCE_DIR}/${MAX_STABLE}/src/libpmemblk/libpmemblk.link.in LINKFILE)
			if(DEFINED ENV{FAULT_INJECTION})
				string(REPLACE "fault_injection;" "libpmemblk_inject_fault_at;\n\t\tlibpmemblk_fault_injection_enabled;" LINKFILE "${LINKFILE}")
			else()
				string(REPLACE "fault_injection;" "" LINKFILE "${LINKFILE}")
			endif()
			FILE(WRITE ${CMAKE_SOURCE_DIR}/${MAX_STABLE}/src/libpmemblk/libpmemblk.link "${LINKFILE}")
			target_link_libraries(pmemblk_18 PRIVATE -Wl,--version-script=${CMAKE_SOURCE_DIR}/${MAX_STABLE}/src/libpmemblk/libpmemblk.link)
		endif()
	endfunction(build_libpmemblk_18)

	build_libpmemblk_18()

	function(build_libpmemlog_18)
		set(LOG_SOURCES
			${CMAKE_SOURCE_DIR}/${MAX_STABLE}/src/libpmemlog/libpmemlog.c
			${CMAKE_SOURCE_DIR}/${MAX_STABLE}/src/libpmemlog/log.c
			${COMMON_SOURCES}
			)

		if(WIN32)
			set(LOG_SOURCES ${LOG_SOURCES}
				${CMAKE_SOURCE_DIR}/${MAX_STABLE}/src/libpmemlog/libpmemlog_main.c
				)
		endif()

		if(MSVC)
			FILE(READ ${CMAKE_SOURCE_DIR}/${MAX_STABLE}/src/libpmemlog/libpmemlog.def DEFFILE)
			string(REPLACE "libpmemlog" "pmemlog_18" DEFFILE "${DEFFILE}")
			FILE(WRITE ${CMAKE_CURRENT_BINARY_DIR}/pmemlog_18.def "${DEFFILE}")
			add_library(pmemlog_18 SHARED ${LOG_SOURCES} ${CMAKE_CURRENT_BINARY_DIR}/pmemlog_18.def)
			target_compile_options(pmemlog_18 PRIVATE /FI"${CMAKE_SOURCE_DIR}/${MAX_STABLE}/src/windows/include/platform.h")
		else()
			add_library(pmemlog_18 SHARED ${LOG_SOURCES})
		endif()

		target_compile_definitions(pmemlog_18 PRIVATE SRCVERSION="${MAX_STABLE}")
		target_compile_definitions(pmemlog_18 PRIVATE PMDK_UTF8_API)
		target_compile_definitions(pmemlog_18 PRIVATE _PMEMLOG_INTRNL)

		target_include_directories(pmemlog_18 PRIVATE ${CMAKE_SOURCE_DIR}/${MAX_STABLE}/src/include)
		target_include_directories(pmemlog_18 PRIVATE ${CMAKE_SOURCE_DIR}/${MAX_STABLE}/src/libpmemlog)
		target_include_directories(pmemlog_18 PRIVATE ${CMAKE_SOURCE_DIR}/${MAX_STABLE}/src/common)
		target_include_directories(pmemlog_18 PRIVATE ${PMEM_INCLUDE_DIRS})

		if(WIN32)
			target_include_directories(pmemlog_18 PRIVATE ${CMAKE_SOURCE_DIR}/${MAX_STABLE}/src/windows/include/)
			target_include_directories(pmemlog_18 PRIVATE ${CMAKE_SOURCE_DIR}/include/)
		endif()

		target_link_libraries(pmemlog_18 PRIVATE libpmem-convert)
		target_link_libraries(pmemlog_18 PRIVATE ${CMAKE_DL_LIBS})
		target_link_libraries(pmemlog_18 PRIVATE ${CMAKE_THREAD_LIBS_INIT})

		if(NOT MSVC)
			FILE(READ ${CMAKE_SOURCE_DIR}/${MAX_STABLE}/src/libpmemlog/libpmemlog.link.in LINKFILE)
			if(DEFINED ENV{FAULT_INJECTION})
				string(REPLACE "fault_injection;" "libpmemblk_inject_fault_at;\n\t\tlibpmemblk_fault_injection_enabled;" LINKFILE "${LINKFILE}")
			else()
				string(REPLACE "fault_injection;" "" LINKFILE "${LINKFILE}")
			endif()
			FILE(WRITE ${CMAKE_SOURCE_DIR}/${MAX_STABLE}/src/libpmemlog/libpmemlog.link "${LINKFILE}")
			target_link_libraries(pmemlog_18 PRIVATE -Wl,--version-script=${CMAKE_SOURCE_DIR}/${MAX_STABLE}/src/libpmemlog/libpmemlog.link)
		endif()
	endfunction(build_libpmemlog_18)

	build_libpmemlog_18()

	function(build_pmemspoil)
		set(PMEMSPOIL_SOURCES
			${CMAKE_SOURCE_DIR}/${MAX_STABLE}/src/test/tools/pmemspoil/spoil.c
			${CMAKE_SOURCE_DIR}/${MAX_STABLE}/src/tools/pmempool/common.c
			${CMAKE_SOURCE_DIR}/${MAX_STABLE}/src/libpmemblk/btt.c
			${CMAKE_SOURCE_DIR}/${MAX_STABLE}/src/tools/pmempool/output.c
			${COMMON_SOURCES}
			)

		if(WIN32)
			set(PMEMSPOIL_SOURCES
				${PMEMSPOIL_SOURCES}
				${CMAKE_SOURCE_DIR}/${MAX_STABLE}/src/windows/getopt/getopt.c
				)
		endif()

		add_executable(pmemspoil ${PMEMSPOIL_SOURCES})

		if(WIN32)
			target_include_directories(pmemspoil PRIVATE ${CMAKE_SOURCE_DIR}/${MAX_STABLE}/src/windows/getopt)
			target_include_directories(pmemspoil PRIVATE ${CMAKE_SOURCE_DIR}/${MAX_STABLE}/src/windows/include)
		endif()

		if(MSVC)
			target_include_directories(pmemspoil PRIVATE ${CMAKE_SOURCE_DIR}/include/)
			target_compile_options(pmemspoil PRIVATE /FI"${CMAKE_SOURCE_DIR}/${MAX_STABLE}/src/windows/include/platform.h")
		endif()

		target_compile_definitions(pmemspoil PRIVATE SRCVERSION="${MAX_STABLE}")
		target_compile_definitions(pmemspoil PRIVATE PMDK_UTF8_API)
		target_compile_definitions(pmemspoil PRIVATE LIBDIR="${CMAKE_INSTALL_PREFIX}/${CMAKE_INSTALL_LIBDIR}")

		target_include_directories(pmemspoil PRIVATE ${CMAKE_SOURCE_DIR}/${MAX_STABLE}/src/include)
		target_include_directories(pmemspoil PRIVATE ${CMAKE_SOURCE_DIR}/${MAX_STABLE}/src/common)
		target_include_directories(pmemspoil PRIVATE ${CMAKE_SOURCE_DIR}/${MAX_STABLE}/src/libpmemobj)
		target_include_directories(pmemspoil PRIVATE ${CMAKE_SOURCE_DIR}/${MAX_STABLE}/src/libpmemlog)
		target_include_directories(pmemspoil PRIVATE ${CMAKE_SOURCE_DIR}/${MAX_STABLE}/src/libpmemblk)
		target_include_directories(pmemspoil PRIVATE ${CMAKE_SOURCE_DIR}/${MAX_STABLE}/src/tools/pmempool)

		target_link_libraries(pmemspoil PRIVATE ${CMAKE_THREAD_LIBS_INIT})
		target_link_libraries(pmemspoil PRIVATE libpmem-convert)
		target_link_libraries(pmemspoil PRIVATE pmemobj_18)

		if(WIN32)
			# on windows dll and executable must be in the same dir
			add_custom_command(TARGET pmemspoil POST_BUILD
				COMMAND ${CMAKE_COMMAND} -E copy_if_different
				$<TARGET_FILE:libpmem-convert>
				$<TARGET_FILE_DIR:pmemspoil>)
		endif()
	endfunction(build_pmemspoil)

	build_pmemspoil()

	test("spoiled_pool_hdr" spoiled_pool_hdr)

	function(build_pmempool_convert)
		set(PMEMPOOL_SOURCES
			${CMAKE_SOURCE_DIR}/${MAX_STABLE}/src/tools/pmempool/pmempool.c
			${CMAKE_SOURCE_DIR}/${MAX_STABLE}/src/tools/pmempool/check.c
			${CMAKE_SOURCE_DIR}/${MAX_STABLE}/src/tools/pmempool/common.c
			${CMAKE_SOURCE_DIR}/${MAX_STABLE}/src/tools/pmempool/convert.c
			${CMAKE_SOURCE_DIR}/${MAX_STABLE}/src/tools/pmempool/create.c
			${CMAKE_SOURCE_DIR}/${MAX_STABLE}/src/tools/pmempool/dump.c
			${CMAKE_SOURCE_DIR}/${MAX_STABLE}/src/tools/pmempool/feature.c
			${CMAKE_SOURCE_DIR}/${MAX_STABLE}/src/tools/pmempool/info.c
			${CMAKE_SOURCE_DIR}/${MAX_STABLE}/src/tools/pmempool/info_blk.c
			${CMAKE_SOURCE_DIR}/${MAX_STABLE}/src/tools/pmempool/info_obj.c
			${CMAKE_SOURCE_DIR}/${MAX_STABLE}/src/tools/pmempool/info_log.c
			${CMAKE_SOURCE_DIR}/${MAX_STABLE}/src/tools/pmempool/output.c
			${CMAKE_SOURCE_DIR}/${MAX_STABLE}/src/tools/pmempool/rm.c
			${CMAKE_SOURCE_DIR}/${MAX_STABLE}/src/tools/pmempool/synchronize.c
			${CMAKE_SOURCE_DIR}/${MAX_STABLE}/src/tools/pmempool/transform.c

			${CMAKE_SOURCE_DIR}/${MAX_STABLE}/src/libpmemblk/btt.c

			${CMAKE_SOURCE_DIR}/${MAX_STABLE}/src/libpmemobj/critnib.c
			${CMAKE_SOURCE_DIR}/${MAX_STABLE}/src/libpmemobj/ulog.c
			${CMAKE_SOURCE_DIR}/${MAX_STABLE}/src/libpmemobj/memblock.c
			${CMAKE_SOURCE_DIR}/${MAX_STABLE}/src/libpmemobj/alloc_class.c
			${CMAKE_SOURCE_DIR}/${MAX_STABLE}/src/libpmemobj/memops.c
			${CMAKE_SOURCE_DIR}/${MAX_STABLE}/src/libpmemobj/heap.c
			${CMAKE_SOURCE_DIR}/${MAX_STABLE}/src/libpmemobj/bucket.c
			${CMAKE_SOURCE_DIR}/${MAX_STABLE}/src/libpmemobj/recycler.c
			${CMAKE_SOURCE_DIR}/${MAX_STABLE}/src/libpmemobj/container_seglists.c
			${CMAKE_SOURCE_DIR}/${MAX_STABLE}/src/libpmemobj/container_ravl.c
			${CMAKE_SOURCE_DIR}/${MAX_STABLE}/src/libpmemobj/ravl.c
			${COMMON_SOURCES}
			)

		if(NOT WIN32)
			set(PMEMPOOL_SOURCES ${PMEMPOOL_SOURCES}
				${CMAKE_SOURCE_DIR}/${MAX_STABLE}/src/librpmem/rpmem_util.c)
		endif()

		if(WIN32)
			add_executable(pmempool-convert ${PMEMPOOL_SOURCES} ${CMAKE_SOURCE_DIR}/${MAX_STABLE}/src/windows/getopt/getopt.c)
			target_include_directories(pmempool-convert PRIVATE ${CMAKE_SOURCE_DIR}/${MAX_STABLE}/src/windows/getopt/)
			target_include_directories(pmempool-convert PRIVATE ${CMAKE_SOURCE_DIR}/${MAX_STABLE}/src/windows/include/)
		else()
			add_executable(pmempool-convert ${PMEMPOOL_SOURCES})
		endif()

		if(MSVC)
			target_include_directories(pmempool-convert PRIVATE ${CMAKE_SOURCE_DIR}/include/)
			target_compile_options(pmempool-convert PRIVATE /FI"${CMAKE_SOURCE_DIR}/${MAX_STABLE}/src/windows/include/platform.h")
		endif()

		target_compile_definitions(pmempool-convert PRIVATE SRCVERSION="${MAX_STABLE}")
		target_compile_definitions(pmempool-convert PRIVATE PMDK_UTF8_API)
		target_compile_definitions(pmempool-convert PRIVATE LIBDIR="${CMAKE_INSTALL_PREFIX}/${CMAKE_INSTALL_LIBDIR}")

		target_include_directories(pmempool-convert PRIVATE ${CMAKE_SOURCE_DIR}/${MAX_STABLE}/src/include)
		target_include_directories(pmempool-convert PRIVATE ${CMAKE_SOURCE_DIR}/${MAX_STABLE}/src/common)
		target_include_directories(pmempool-convert PRIVATE ${CMAKE_SOURCE_DIR}/${MAX_STABLE}/src/libpmemlog)
		target_include_directories(pmempool-convert PRIVATE ${CMAKE_SOURCE_DIR}/${MAX_STABLE}/src/libpmemblk)
		target_include_directories(pmempool-convert PRIVATE ${CMAKE_SOURCE_DIR}/${MAX_STABLE}/src/libpmemobj)
		target_include_directories(pmempool-convert PRIVATE ${CMAKE_SOURCE_DIR}/${MAX_STABLE}/src/libpmempool)
		target_include_directories(pmempool-convert PRIVATE ${CMAKE_SOURCE_DIR}/${MAX_STABLE}/src/rpmem_common)
		target_include_directories(pmempool-convert PRIVATE ${CMAKE_SOURCE_DIR}/${MAX_STABLE}/src/librpmem)
		target_include_directories(pmempool-convert PRIVATE ${CMAKE_SOURCE_DIR}/${MAX_STABLE}/src/tools/pmempool)

		target_link_libraries(pmempool-convert PRIVATE ${CMAKE_THREAD_LIBS_INIT})
		target_link_libraries(pmempool-convert PRIVATE ${CMAKE_DL_LIBS})
		target_link_libraries(pmempool-convert PRIVATE libpmem-convert)
		target_link_libraries(pmempool-convert PRIVATE libpmempool-convert)
		target_link_libraries(pmempool-convert PRIVATE pmemobj_18)
		target_link_libraries(pmempool-convert PRIVATE pmemblk_18)
		target_link_libraries(pmempool-convert PRIVATE pmemlog_18)

		if(WIN32)
			# on windows dll and executable must be in the same dir
			add_custom_command(TARGET pmempool-convert POST_BUILD
				COMMAND ${CMAKE_COMMAND} -E copy_if_different
				$<TARGET_FILE:libpmempool-convert>
				$<TARGET_FILE_DIR:pmempool-convert>)
		endif()
	endfunction(build_pmempool_convert)

	build_pmempool_convert()

	test("compare_replicas" compare_replicas)
endif()

if(NOT WIN32)
	if(BUILD_CONVERT_V1)
		add_executable(transaction_10 transaction.c)
		disable_optimization(transaction_10)
		target_include_directories(transaction_10 PRIVATE ${CMAKE_SOURCE_DIR}/${VER10}/src/include)
		target_link_libraries(transaction_10 PRIVATE libpmem-convert)
		target_link_libraries(transaction_10 PRIVATE pmemobj_10)
	endif()

	if(BUILD_CONVERT_V2)
		add_executable(transaction_11 transaction.c)
		disable_optimization(transaction_11)
		target_include_directories(transaction_11 PRIVATE ${CMAKE_SOURCE_DIR}/${VER11}/src/include)
		target_link_libraries(transaction_11 PRIVATE libpmem-convert)
		target_link_libraries(transaction_11 PRIVATE pmemobj_11)
	endif()

	if(BUILD_CONVERT_V3)
		add_executable(transaction_12 transaction.c)
		disable_optimization(transaction_12)
		target_include_directories(transaction_12 PRIVATE ${CMAKE_SOURCE_DIR}/${VER12}/src/include)
		target_link_libraries(transaction_12 PRIVATE libpmem-convert)
		target_link_libraries(transaction_12 PRIVATE pmemobj_12)
	endif()

	if(BUILD_13)
		add_executable(transaction_13 transaction.c)
		disable_optimization(transaction_13)
		target_include_directories(transaction_13 PRIVATE ${CMAKE_SOURCE_DIR}/${VER13}/src/include)
		target_link_libraries(transaction_13 PRIVATE libpmem-convert)
		target_link_libraries(transaction_13 PRIVATE pmemobj_13)
	endif()
endif()

if(BUILD_CONVERT_V4)
	add_executable(transaction_14 transaction.c)
	disable_optimization(transaction_14)
	target_include_directories(transaction_14 PRIVATE ${CMAKE_SOURCE_DIR}/${VER14}/src/include)
	target_link_libraries(transaction_14 PRIVATE libpmem-convert)
	target_link_libraries(transaction_14 PRIVATE pmemobj_14)
endif()

if(BUILD_15)
	add_executable(transaction_15 transaction.c)
	disable_optimization(transaction_15)
	target_include_directories(transaction_15 PRIVATE ${CMAKE_SOURCE_DIR}/${VER15}/src/include)
	target_link_libraries(transaction_15 PRIVATE libpmem-convert)
	target_link_libraries(transaction_15 PRIVATE pmemobj_15)
endif()

if(BUILD_CONVERT_V5)
	add_executable(transaction_16 transaction.c)
	disable_optimization(transaction_16)
	target_include_directories(transaction_16 PRIVATE ${CMAKE_SOURCE_DIR}/${VER16}/src/include)
	target_link_libraries(transaction_16 PRIVATE libpmem-convert)
	target_link_libraries(transaction_16 PRIVATE pmemobj_16)
endif()

if(BUILD_17)
	add_executable(transaction_17 transaction.c)
	disable_optimization(transaction_17)
	target_include_directories(transaction_17 PRIVATE ${CMAKE_SOURCE_DIR}/${VER17}/src/include)
	target_link_libraries(transaction_17 PRIVATE libpmem-convert)
	target_link_libraries(transaction_17 PRIVATE pmemobj_17)
endif()

if(BUILD_CONVERT_V6)
	add_executable(transaction_18 transaction.c)
	disable_optimization(transaction_18)
	target_include_directories(transaction_18 PRIVATE ${CMAKE_SOURCE_DIR}/${VER18}/src/include)
	target_link_libraries(transaction_18 PRIVATE libpmem-convert)
	target_link_libraries(transaction_18 PRIVATE pmemobj_18)
endif()

if(MSVC)
	if(BUILD_CONVERT_V4)
		# project has to be compile as c++ project so we need to change the flag
		SET_SOURCE_FILES_PROPERTIES(transaction.c PROPERTIES LANGUAGE CXX)
		# on windows dll and executable must be in the same dir
		add_custom_command(TARGET transaction_14 POST_BUILD
			COMMAND ${CMAKE_COMMAND} -E copy_if_different
			$<TARGET_FILE:pmemobj_14>
			$<TARGET_FILE_DIR:transaction_14>)

		add_custom_command(TARGET transaction_14 POST_BUILD
			COMMAND ${CMAKE_COMMAND} -E copy_if_different
			$<TARGET_FILE_DIR:pmemobj_14>/libpmem-convert.dll
			$<TARGET_FILE_DIR:transaction_14>)
	endif()

	if(BUILD_CONVERT_V5)
		# project has to be compile as c++ project so we need to change the flag
		SET_SOURCE_FILES_PROPERTIES(transaction.c PROPERTIES LANGUAGE CXX)
		# on windows dll and executable must be in the same dir
		add_custom_command(TARGET transaction_16 POST_BUILD
			COMMAND ${CMAKE_COMMAND} -E copy_if_different
			$<TARGET_FILE:pmemobj_16>
			$<TARGET_FILE_DIR:transaction_16>)

		add_custom_command(TARGET transaction_16 POST_BUILD
			COMMAND ${CMAKE_COMMAND} -E copy_if_different
			$<TARGET_FILE_DIR:pmemobj_16>/libpmem-convert.dll
			$<TARGET_FILE_DIR:transaction_16>)
	endif()

	if(BUILD_CONVERT_V6)
		# project has to be compile as c++ project so we need to change the flag
		SET_SOURCE_FILES_PROPERTIES(transaction.c PROPERTIES LANGUAGE CXX)
		# on windows dll and executable must be in the same dir
		add_custom_command(TARGET transaction_18 POST_BUILD
			COMMAND ${CMAKE_COMMAND} -E copy_if_different
			$<TARGET_FILE:pmemobj_18>
			$<TARGET_FILE_DIR:transaction_18>)

		add_custom_command(TARGET transaction_18 POST_BUILD
			COMMAND ${CMAKE_COMMAND} -E copy_if_different
			$<TARGET_FILE_DIR:pmemobj_18>/libpmem-convert.dll
			$<TARGET_FILE_DIR:transaction_18>)
	endif()
endif()

test("cli_test" cli)

add_executable(clean_pool clean.c)
target_include_directories(clean_pool PRIVATE ${LIBPMEMPOOL_INCLUDE_DIRS})
target_link_libraries(clean_pool PRIVATE ${LIBPMEMPOOL_LIBRARIES})
file(WRITE ${TEST_DIR}/../tx_intr.lock "")

if(NOT WIN32)
	if(DEVICE_DAX_PATHS)
		file(WRITE ${TEST_DIR}/../devdax.lock "")
	endif()

	test("tx_intr_pool_regfile" tx_intr_pool_regfile)
	test("tx_intr_poolset_local_2regfiles" tx_intr_poolset_local_2regfiles)
	test("tx_intr_poolset_local_regfile" tx_intr_poolset_local_regfile)
	test("tx_intr_poolset_local_regfile_rep_regfile"
		tx_intr_poolset_local_regfile_rep_regfile)
	test("tx_intr_poolset_local_2regfiles_rep_3regfiles"
		tx_intr_poolset_local_2regfiles_rep_3regfiles)
	test("poolsets_with_remote_replica" remote)

	if(NOT DEVICE_DAX_PATHS)
		test("tx_intr_pool_devdax_SKIP_BECAUSE_OF_NO_DEVDAX_DEFINED"
			skip_test)
		test("tx_intr_poolset_local_devdax_devdax_SKIP_BECAUSE_OF_NO_DEVDAX_DEFINED"
			skip_test)
	else()
		test("tx_intr_pool_devdax" tx_intr_pool_devdax)
		test("tx_intr_poolset_local_devdax" tx_intr_poolset_local_devdax)
	endif()

	if(devdax_num LESS 2)
		test("tx_intr_poolset_local_2devdaxes_SKIP_BECAUSE_OF_NOT_ENOUGH_DEVDAX_DEFINED"
			skip_test)
		test("tx_intr_poolset_local_devdax_rep_devdax_SKIP_BECAUSE_OF_NOT_ENOUGH_DEVDAX_DEFINED"
			skip_test)
		test("tx_intr_poolset_local_2devdaxes_rep_2regfiles_SKIP_BECAUSE_OF_NOT_ENOUGH_DEVDAX_DEFINED"
			skip_test)
		test("tx_intr_poolset_local_2regfiles_rep_2devdaxes_SKIP_BECAUSE_OF_NOT_ENOUGH_DEVDAX_DEFINED"
			skip_test)
	else()
		test("tx_intr_poolset_local_2devdaxes" tx_intr_poolset_local_2devdaxes)
		test("tx_intr_poolset_local_devdax_rep_devdax"
			tx_intr_poolset_local_devdax_rep_devdax)
		test("tx_intr_poolset_local_2devdaxes_rep_2regfiles"
			tx_intr_poolset_local_2devdaxes_rep_2regfiles)
		test("tx_intr_poolset_local_2regfiles_rep_2devdaxes"
			tx_intr_poolset_local_2regfiles_rep_2devdaxes)
	endif()
else()
	if(CMAKE_BUILD_TYPE STREQUAL Debug OR CMAKE_BUILD_TYPE STREQUAL RelWithDebInfo)
		set_cdb_executable()
		if(CDB_EXE)
			test("tx_intr_pool_regfile" tx_intr_pool_regfile)
			test("tx_intr_poolset_local_2regfiles" tx_intr_poolset_local_2regfiles)
			test("tx_intr_poolset_local_regfile" tx_intr_poolset_local_regfile)
			test("tx_intr_poolset_local_regfile_rep_regfile" tx_intr_poolset_local_regfile_rep_regfile)
		else()
			test("tx_intr_pool_regfile_SKIP_BECAUSE_OF_NO_CDB_DEBUGGER" skip_test)
			test("tx_intr_poolset_local_2regfiles_SKIP_BECAUSE_OF_NO_CDB_DEBUGGER" skip_test)
			test("tx_intr_poolset_local_regfile_SKIP_BECAUSE_OF_NO_CDB_DEBUGGER" skip_test)
			test("tx_intr_poolset_local_regfile_rep_regfile_SKIP_BECAUSE_OF_NO_CDB_DEBUGGER" skip_test)
		endif()
	else()
		test("tx_intr_pool_regfile_SKIP_BECAUSE_OF_NO_DEBUG_BUILD_TYPE" skip_test)
		test("tx_intr_poolset_local_2regfiles_SKIP_BECAUSE_OF_NO_DEBUG_BUILD_TYPE" skip_test)
		test("tx_intr_poolset_local_regfile_SKIP_BECAUSE_OF_NO_DEBUG_BUILD_TYPE" skip_test)
		test("tx_intr_poolset_local_regfile_rep_regfile_SKIP_BECAUSE_OF_NO_DEBUG_BUILD_TYPE" skip_test)
	endif()
endif()
