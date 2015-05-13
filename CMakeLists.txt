IF(BIICODE)
	SET(SRC_OS_PATH src/os)
	SET(TESTS_PATH tests)
	SET(EXAMPLES_PATH examples)
	IF(WIN32)
		LIST(REMOVE_ITEM BII_LIB_SRC 
			${SRC_OS_PATH}/aix/aix_sigar.c 
			${SRC_OS_PATH}/darwin/darwin_sigar.c  
			${SRC_OS_PATH}/hpux/hpux_sigar.c 
			${SRC_OS_PATH}/linux/linux_sigar.c 
			${SRC_OS_PATH}/solaris/kstats.c 
			${SRC_OS_PATH}/solaris/get_mib2.c 
			${SRC_OS_PATH}/solaris/procfs.c 
			${SRC_OS_PATH}/solaris/solaris_sigar.c)
	ELSEIF(CMAKE_SYSTEM_NAME STREQUAL "Linux")
		LIST(REMOVE_ITEM BII_LIB_SRC 
			${SRC_OS_PATH}/aix/aix_sigar.c 
			${SRC_OS_PATH}/darwin/darwin_sigar.c  
			${SRC_OS_PATH}/hpux/hpux_sigar.c 
			${SRC_OS_PATH}/win32/peb.c
			${SRC_OS_PATH}/win32/win32_sigar.c 
			${SRC_OS_PATH}/win32/wmi.cpp 
			${SRC_OS_PATH}/solaris/kstats.c 
			${SRC_OS_PATH}/solaris/get_mib2.c 
			${SRC_OS_PATH}/solaris/procfs.c 
			${SRC_OS_PATH}/solaris/solaris_sigar.c)
		# ToDo: Cleanup lists for other operating systems
	ENDIF()
	
	# Remove examples and tests:
	#LIST(REMOVE_ITEM BII_BLOCK_EXES ${EXAMPLES_PATH}_cpuinfo ${EXAMPLES_PATH}_sigar_ps)
	#LIST(REMOVE_ITEM BII_BLOCK_EXES ${TESTS_PATH}_t_sigar_cpu ${TESTS_PATH}_t_sigar_fs ${TESTS_PATH}_t_sigar_loadavg ${TESTS_PATH}_t_sigar_mem ${TESTS_PATH}_t_sigar_netconn ${TESTS_PATH}_t_sigar_netif ${TESTS_PATH}_t_sigar_pid ${TESTS_PATH}_t_sigar_proc ${TESTS_PATH}_t_sigar_reslimit ${TESTS_PATH}_t_sigar_swap ${TESTS_PATH}_t_sigar_sysinfo ${TESTS_PATH}_t_sigar_uptime ${TESTS_PATH}_t_sigar_version)
	
	ADD_BII_TARGETS()
	
	IF(WIN32)
		TARGET_INCLUDE_DIRECTORIES(${BII_LIB_TARGET} PUBLIC ${SRC_OS_PATH}/win32)
		TARGET_LINK_LIBRARIES(${BII_LIB_TARGET} PUBLIC ws2_32 netapi32 version)
	ELSEIF(CMAKE_SYSTEM_NAME STREQUAL "Linux")
		TARGET_INCLUDE_DIRECTORIES(${BII_LIB_TARGET} PUBLIC ${SRC_OS_PATH}/linux)	
	ENDIF()
	
	RETURN()
ENDIF()

IF(WIN32) 
	PROJECT(sigar)
ELSE()
	PROJECT(sigar C)
ENDIF()

cmake_minimum_required(VERSION 2.6)

INCLUDE(CheckIncludeFiles)
INCLUDE(CheckFunctionExists)
INCLUDE(CheckTypeSize)
INCLUDE(CTest)
INCLUDE(InstallRequiredSystemLibraries)

ENABLE_TESTING()

SET(CPACK_PACKAGE_VERSION_MAJOR "1")
SET(CPACK_PACKAGE_VERSION_MINOR "6")
SET(CPACK_PACKAGE_VERSION_PATCH "2")
SET(CPACK_PACKAGE_VENDOR "Hyperic")
## SET(CPACK_*) before the INCLUDE(CPack)
INCLUDE(CPack)

IF(WIN32) 
	## make sure we only use the smallest set of 
	## headers on win32. Otherwise we get clashes 
	## between winsock2.h and winsock.h
	ADD_DEFINITIONS(-DWIN32_LEAN_AND_MEAN)

	# force the correct version for the redist manifest
	ADD_DEFINITIONS(-D_BIND_TO_CURRENT_MFC_VERSION=1 -D_BIND_TO_CURRENT_CRT_VERSION=1)
	# turn off security warnings for system calls
	ADD_DEFINITIONS(-D_CRT_SECURE_NO_WARNINGS)
ENDIF(WIN32)

ADD_SUBDIRECTORY(src build-src)
ADD_SUBDIRECTORY(tests build-tests)
ADD_SUBDIRECTORY(include build-include)

