#
# Copyright (C) 2015-2015 Oleg Alexeenkov
#
# All rights reserved.
#
# Redistribution and use in source and binary forms, with or without
# modification, are permitted provided that the following conditions
# are met:
# 1. Redistributions of source code must retain the above copyright
#    notice, this list of conditions and the following disclaimer.
# 2. Redistributions in binary form must reproduce the above copyright
#    notice, this list of conditions and the following disclaimer in the
#    documentation and/or other materials provided with the distribution.
# 3. Neither the name of the project nor the names of its contributors
#    may be used to endorse or promote products derived from this software
#    without specific prior written permission.
#
# THIS SOFTWARE IS PROVIDED BY THE PROJECT AND CONTRIBUTORS ``AS IS'' AND
# ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
# IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE
# ARE DISCLAIMED. IN NO EVENT SHALL THE PROJECT OR CONTRIBUTORS BE LIABLE
# FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL
# DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS
# OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION)
# HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT
# LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY
# OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF
# SUCH DAMAGE.
#

cmake_minimum_required(VERSION 2.6)

project(usrsctplib)

include(CheckFunctionExists)
include(CheckStructHasMember)
include(CheckIncludeFile)
include(CMakePushCheckState)

include(CheckTypeSize)

SET (VERSION "1.0.0")

SET (prefix ${CMAKE_INSTALL_PREFIX})
SET (exec_prefix "\${prefix}")
SET (libdir "\${exec_prefix}/lib")
SET (includedir "\${prefix}/include/usrsctp")

check_type_size("size_t" HAVE_SIZE_T )
check_type_size("ssize_t" HAVE_SSIZE_T)

check_include_file(sys/socket.h HAVE_SYS_SOCKET_H)
check_include_file(sys/types.h HAVE_SYS_TYPES_H)
check_include_file(sys/queue.h HAVE_SYS_QUEUE_H)
check_include_file(linux/if_addr.h HAVE_LINUX_IF_ADDR_H)
check_include_file(linux/rtnetlink.h HAVE_LINUX_RTNETLINK_H)
check_include_file(netinet/ip_icmp.h HAVE_NETINET_IP_ICMP_H)

check_function_exists("socket" HAVE_SOCKET)
check_function_exists("inet_addr" HAVE_INET_ADDR)

add_definitions(-D__Userspace__ "-D__Userspace_os_${CMAKE_SYSTEM_NAME}")

check_struct_has_member("struct sockaddr" "sa_len" "sys/types.h;sys/socket.h" HAVE_SA_LEN)
if(HAVE_SA_LEN)
	add_definitions(-DHAVE_SA_LEN)
endif()

check_struct_has_member("struct sockaddr_in" "sin_len" "sys/types.h;netinet/in.h" HAVE_SIN_LEN)
if(HAVE_SIN_LEN)
	add_definitions(-DHAVE_SIN_LEN)
endif()

check_struct_has_member("struct sockaddr_in6" "sin6_len" "sys/types.h;netinet/in.h" HAVE_SIN6_LEN)
if(HAVE_SIN6_LEN)
	add_definitions(-DHAVE_SIN6_LEN)
endif()

cmake_push_check_state()
set(CMAKE_REQUIRED_INCLUDES "${CMAKE_SOURCE_DIR}/src/network")
check_struct_has_member("struct sockaddr_conn" "sconn_len" "usrsctplib/usrsctp.h" HAVE_SCONN_LEN)
if(HAVE_SCONN_LEN)
	add_definitions(-DHAVE_SCONN_LEN)
endif()
cmake_pop_check_state()


if (${CMAKE_SYSTEM_NAME} MATCHES "FreeBSD")
	add_definitions(-U__FreeBSD__)
endif()

if (${CMAKE_SYSTEM_NAME} MATCHES "Darwin")
	add_definitions(-U__APPLE__ -D__APPLE_USE_RFC_2292)
endif()

if (${CMAKE_SYSTEM_NAME} MATCHES "DragonFly")
	add_definitions(-U__DragonFly__)
endif()

if (${CMAKE_SYSTEM_NAME} MATCHES "NetBSD")
	add_definitions(-U__NetBSD__)
endif()

if (${CMAKE_SYSTEM_NAME} MATCHES "OpenBSD")
	add_definitions(-U__OpenBSD__)
endif()


option(SCTP_DEBUG "Provide debug information" 0)
if (SCTP_DEBUG)
	add_definitions(-DSCTP_DEBUG)
endif()

option(INET "Support IPv4 " 1)
if (INET)
	add_definitions(-DINET)
endif()

option(INET6 "Support IPv6 " 1)
if (INET6)
	add_definitions(-DINET6)
endif()

option(SCTP_SIMPLE_ALLOCATOR " " 1)
if (SCTP_SIMPLE_ALLOCATOR)
	add_definitions(-DSCTP_SIMPLE_ALLOCATOR)
endif()

option(SCTP_PROCESS_LEVEL_LOCKS " " 1)
if (SCTP_PROCESS_LEVEL_LOCKS)
	add_definitions(-DSCTP_PROCESS_LEVEL_LOCKS)
endif()

option(SCTP_WITH_NO_CSUM " " 1)
if (SCTP_WITH_NO_CSUM)
	add_definitions(-DSCTP_WITH_NO_CSUM)
endif()

option(SCTP_MBUF_LOGGING " " 0)
if (SCTP_MBUF_LOGGING)
	add_definitions(-DSCTP_MBUF_LOGGING)
endif()

option(SCTP_PACKET_LOGGING " " 0)
if (SCTP_PACKET_LOGGING)
	add_definitions(-DSCTP_PACKET_LOGGING)
endif()

option(SCTP_SO_LOCK_TESTING " " 0)
if (SCTP_SO_LOCK_TESTING)
	add_definitions(-DSCTP_SO_LOCK_TESTING)
endif()

option(SCTP_EMBEDDED_V6_SCOPE " " 0)
if (SCTP_EMBEDDED_V6_SCOPE)
	add_definitions(-DSCTP_EMBEDDED_V6_SCOPE)
endif()

option(SCTP_KAME " " 0)
if (SCTP_KAME)
	add_definitions(-DSCTP_KAME)
endif()

add_definitions(-D_GNU_SOURCE)

CONFIGURE_FILE (
    usrsctp.pc.in
    usrsctp.pc
    @ONLY
)

include_directories(${PROJECT_SOURCE_DIR} ${PROJECT_SOURCE_DIR}/netinet ${PROJECT_SOURCE_DIR}/netinet6 )

LIST (APPEND usrsctp_root_HEADERS
user_atomic.h       user_inpcb.h    user_ip_icmp.h  user_mbuf.h   user_recv_thread.h  user_socketvar.h  usrsctp.h
user_environment.h  user_ip6_var.h  user_malloc.h   user_queue.h  user_route.h        user_uma.h
)

LIST (APPEND usrsctp_netinet_HEADERS
netinet/sctp_asconf.h    netinet/sctp_constants.h  netinet/sctp_indata.h          netinet/sctp_os_userspace.h  netinet/sctp_process_lock.h  netinet/sctp_timer.h
netinet/sctp_auth.h      netinet/sctp_crc32.h      netinet/sctp_input.h           netinet/sctp_output.h        netinet/sctp_sha1.h          netinet/sctp_uio.h
netinet/sctp_bsd_addr.h  netinet/sctp.h            netinet/sctp_lock_userspace.h  netinet/sctp_pcb.h           netinet/sctp_structs.h       netinet/sctputil.h
netinet/sctp_callout.h   netinet/sctp_header.h     netinet/sctp_os.h              netinet/sctp_peeloff.h       netinet/sctp_sysctl.h        netinet/sctp_var.h
)

LIST (APPEND usrsctp_netinet6_HEADERS
netinet6/sctp6_var.h
)

LIST (APPEND usrsctp_HEADERS
	${usrsctp_root_HEADERS}
	${usrsctp_netinet_HEADERS}
	${usrsctp_netinet6_HEADERS}
)

LIST (APPEND usrsctp_SOURCES
netinet6/sctp6_usrreq.c  netinet/sctp_callout.c       netinet/sctp_input.c    netinet/sctp_sha1.c          netinet/sctp_userspace.c  user_mbuf.c
netinet/sctp_asconf.c    netinet/sctp_cc_functions.c  netinet/sctp_output.c   netinet/sctp_ss_functions.c  netinet/sctp_usrreq.c     user_recv_thread.c
netinet/sctp_auth.c      netinet/sctp_crc32.c         netinet/sctp_pcb.c      netinet/sctp_sysctl.c        netinet/sctputil.c        user_socket.c
netinet/sctp_bsd_addr.c  netinet/sctp_indata.c        netinet/sctp_peeloff.c  netinet/sctp_timer.c         user_environment.c
)

add_library(usrsctp-static STATIC ${usrsctp_SOURCES} ${usrsctp_HEADERS})
add_library(usrsctp SHARED ${usrsctp_SOURCES} ${usrsctp_HEADERS})

SET_TARGET_PROPERTIES (usrsctp-static PROPERTIES OUTPUT_NAME "usrsctp")
SET_TARGET_PROPERTIES (usrsctp PROPERTIES IMPORT_SUFFIX "_import.lib")
SET_TARGET_PROPERTIES (usrsctp PROPERTIES SOVERSION 1 VERSION 1.0.0)

IF ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "Clang" OR "${CMAKE_CXX_COMPILER_ID}" STREQUAL "GNU")
    SET_PROPERTY (TARGET usrsctp APPEND_STRING PROPERTY COMPILE_FLAGS "-Wall -g -O3 -DPIC")
    SET_PROPERTY (TARGET usrsctp-static APPEND_STRING PROPERTY COMPILE_FLAGS "-Wall -g -O3" )
ENDIF ()
IF ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "MSVC")
    IF (CMAKE_CXX_FLAGS MATCHES "/W[0-4]")
        STRING(REGEX REPLACE "/W[0-4]" "/W3" CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS}")
    ELSE ()
        SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /W3")
    ENDIF ()
ENDIF ()

IF ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "MSVC90" OR "${CMAKE_CXX_COMPILER_ID}" STREQUAL "MSVC10")
    SET_SOURCE_FILES_PROPERTIES(${usrsctp_SOURCES} PROPERTIES LANGUAGE CXX)
ENDIF()

IF (NOT DEFINED CMAKE_INSTALL_LIBDIR)
    SET(CMAKE_INSTALL_LIBDIR lib)
ENDIF ()

INSTALL (TARGETS usrsctp usrsctp-static DESTINATION ${CMAKE_INSTALL_LIBDIR})
#INSTALL (DIRECTORY include DESTINATION ${CMAKE_INSTALL_PREFIX})
INSTALL (FILES ${usrsctp_root_HEADERS} DESTINATION include/usrsctp)
INSTALL (FILES ${usrsctp_netinet_HEADERS} DESTINATION include/usrsctp/netinet)
INSTALL (FILES ${usrsctp_netinet6_HEADERS} DESTINATION include/usrsctp/netinet6)
INSTALL (FILES ${CMAKE_CURRENT_BINARY_DIR}/usrsctp.pc DESTINATION ${CMAKE_INSTALL_LIBDIR}/pkgconfig)
