cmake_minimum_required (VERSION 2.6) 
project (uuid)

include (CheckTypeSize)
include (CheckIncludeFiles)
include (CheckSymbolExists)
include (CheckLibraryExists)
include (CheckFunctionExists)

include_directories(${CMAKE_CURRENT_BINARY_DIR})

set(CMAKE_BUILD_TYPE Release)


#CHECK_LIBRARY_EXISTS(dmalloc "dmalloc_startup" "" HAVE_LIBDMALLOC)

CHECK_FUNCTION_EXISTS("clock_gettime" HAVE_CLOCK_GETTIME)
CHECK_FUNCTION_EXISTS("getifaddrs" HAVE_GETIFADDRS)
CHECK_FUNCTION_EXISTS("gettimeofday" HAVE_GETTIMEOFDAY)
CHECK_FUNCTION_EXISTS("nanosleep" HAVE_NANOSLEEP)
CHECK_FUNCTION_EXISTS("Sleep" HAVE_SLEEP)

CHECK_INCLUDE_FILES("arpa/inet.h" HAVE_ARPA_INET_H)
CHECK_INCLUDE_FILES("ifaddrs.h" HAVE_IFADDRS_H)
CHECK_INCLUDE_FILES("inttypes.h" HAVE_INTTYPES_H)
CHECK_INCLUDE_FILES("dlfcn.h" HAVE_DLFCN_H)
CHECK_INCLUDE_FILES("dmalloc.h" HAVE_DMALLOC_H)
CHECK_INCLUDE_FILES("memory.h" HAVE_MEMORY_H)
CHECK_INCLUDE_FILES("netdb.h" HAVE_NETDB_H)
CHECK_INCLUDE_FILES("netinet/in.h" HAVE_NETINET_IN_H)
CHECK_INCLUDE_FILES("net/if_arp.h" HAVE_NET_IF_ARP_H)
CHECK_INCLUDE_FILES("net/if_dl.h" HAVE_NET_IF_DL_H)
CHECK_INCLUDE_FILES("net/if.h" HAVE_NET_IF_H)
CHECK_INCLUDE_FILES("stdint.h" HAVE_STDINT_H)
CHECK_INCLUDE_FILES("stdlib.h" HAVE_STDLIB_H)
CHECK_INCLUDE_FILES("strings.h" HAVE_STRINGS_H)
CHECK_INCLUDE_FILES("string.h" HAVE_STRING_H)
CHECK_INCLUDE_FILES("sys/ioctl.h" HAVE_SYS_IOCTL_H)
CHECK_INCLUDE_FILES("sys/param.h" HAVE_SYS_PARAM_H)
CHECK_INCLUDE_FILES("sys/select.h" HAVE_SYS_SELECT_H)
CHECK_INCLUDE_FILES("sys/socket.h" HAVE_SYS_SOCKET_H)
CHECK_INCLUDE_FILES("sys/sockio.h" HAVE_SYS_SOCKIO_H)
CHECK_INCLUDE_FILES("sys/stat.h" HAVE_SYS_STAT_H)
CHECK_INCLUDE_FILES("sys/time.h" HAVE_SYS_TIME_H)
CHECK_INCLUDE_FILES("sys/types.h" HAVE_SYS_TYPES_H)
CHECK_INCLUDE_FILES("unistd.h" HAVE_UNISTD_H)

SET(CMAKE_EXTRA_INCLUDE_FILES "sys/time.h")
CHECK_TYPE_SIZE("struct timeval" STRUCT_TIMEVAL)
SET(CMAKE_EXTRA_INCLUDE_FILES)

CHECK_TYPE_SIZE("long double" LONG_DOUBLE)
CHECK_TYPE_SIZE("long long" LONG_LONG)

CHECK_TYPE_SIZE("char" SIZEOF_CHAR)
CHECK_TYPE_SIZE("int" SIZEOF_INT)
CHECK_TYPE_SIZE("long" SIZEOF_LONG)
CHECK_TYPE_SIZE("long long" SIZEOF_LONG_LONG)
CHECK_TYPE_SIZE("short" SIZEOF_SHORT)
CHECK_TYPE_SIZE("unsigned char" SIZEOF_UNSIGNED_CHAR)
CHECK_TYPE_SIZE("unsigned int" SIZEOF_UNSIGNED_INT)
CHECK_TYPE_SIZE("unsigned long" SIZEOF_UNSIGNED_LONG)
CHECK_TYPE_SIZE("unsigned long long" SIZEOF_UNSIGNED_LONG_LONG)
CHECK_TYPE_SIZE("unsigned short" SIZEOF_UNSIGNED_SHORT)

#CHECK_SYMBOL_EXISTS(va_copy "" HAVE_VA_COPY)
set(HAVE_VA_COPY 1) # TODO: really check for this.
set(WITH_CXX 1)

# I don't know these
#
#/* Define to 1 if you have the `nsl' library (-lnsl). */
#/* #cmakedefine HAVE_LIBNSL */

#/* Define to 1 if you have the `socket' library (-lsocket). */
#/* #cmakedefine HAVE_LIBSOCKET */

#/* Define to 1 if you have the ANSI C header files. */
#/* #cmakedefine STDC_HEADERS */

if(NOT DEFINED UUID_VERSION_HEX)
	file(READ uuid_vers.h UUID_VERS_CONTENT)
	string(REGEX MATCH "#define _UUID_VERSION ([0-9x]*)" UUID_VERSION_HEX_LINE ${UUID_VERS_CONTENT})
	#MESSAGE(STATUS "Found uuid version hex " ${CMAKE_MATCH_1})
	set(UUID_VERSION_HEX ${CMAKE_MATCH_1} CACHE INTERNAL "Internal uuid version hex, used by uuid.h")
endif()

CONFIGURE_FILE("config.h.cmake.in" "config.h")
CONFIGURE_FILE(${CMAKE_CURRENT_BINARY_DIR}/config.h ${CMAKE_CURRENT_SOURCE_DIR}/config.h COPYONLY)
CONFIGURE_FILE("uuid.h.in" "uuid.h")
CONFIGURE_FILE(${CMAKE_CURRENT_BINARY_DIR}/uuid.h ${CMAKE_CURRENT_SOURCE_DIR}/uuid.h COPYONLY)

list(APPEND uuid_CLEAN_FILES ${CMAKE_CURRENT_SOURCE_DIR}/config.h)
list(APPEND uuid_CLEAN_FILES ${CMAKE_CURRENT_SOURCE_DIR}/uuid.h)
list(APPEND uuid_CLEAN_FILES config.h;uuid.h)

# Remove the files generated in the Source directory on clean
set_directory_properties(PROPERTIES   
				ADDITIONAL_MAKE_CLEAN_FILES "${uuid_CLEAN_FILES}")

set(uuid_SOURCE_FILES
	uuid_dce.c
	uuid_mac.c
	uuid_md5.c
	uuid_prng.c
	uuid_sha1.c
	uuid_str.c
	uuid_time.c
	uuid_ui64.c
	uuid_ui128.c
	uuid.c
	uuid++.cc
	)

option(UUID_SHARED "Build uuid as a shared library?" YES)
if(UUID_SHARED)
	set(UUID_LIBTYPE SHARED)
else()
	set(UUID_LIBTYPE STATIC)
endif()


add_library(uuid ${UUID_LIBTYPE} ${uuid_SOURCE_FILES})
set_target_properties(uuid PROPERTIES VERSION 1.6.2)

set(UUID_LIBRARIES uuid CACHE INTERNAL "Libraries needed to link uuid")

install(TARGETS uuid
		EXPORT uuid
        LIBRARY DESTINATION lib
    	ARCHIVE DESTINATION lib)
		
install(EXPORT uuid
		DESTINATION lib/uuid)