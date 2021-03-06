cmake_minimum_required(VERSION 3.0)
project(LIBMODBUS VERSION 3.1.4)

#### CONFIGURATION ####

set(LIBMODBUS_VERSION_MICRO ${LIBMODBUS_VERSION_PATCH})
configure_file(${CMAKE_CURRENT_SOURCE_DIR}/src/modbus-version.h.in ${CMAKE_CURRENT_BINARY_DIR}/modbus-version.h)

include(CheckIncludeFile)
check_include_file(arpa/inet.h HAVE_ARPA_INET_H)
check_include_file(byteswap.h HAVE_BYTESWAP_H)
check_include_file(errno.h HAVE_ERRNO_H)
check_include_file(fcntl.h HAVE_FCNTL_H)
check_include_file(limits.h HAVE_LIMITS_H)
check_include_file(linux/serial.h HAVE_LINUX_SERIAL_H)
check_include_file(netdb.h HAVE_NETDB_H)
check_include_file(netinet/in.h HAVE_NETINET_IN_H)
check_include_file(netinet/tcp.h HAVE_NETINET_TCP_H)
check_include_file(sys/ioctl.h HAVE_SYS_IOCTL_H)
check_include_file(sys/params.h HAVE_SYS_PARAMS_H)
check_include_file(sys/socket.h HAVE_SYS_SOCKET_H)
check_include_file(sys/time.h HAVE_SYS_TIME_H)
check_include_file(sys/types.h HAVE_SYS_TYPES_H)
check_include_file(termios.h HAVE_TERMIOS_H)
check_include_file(time.h HAVE_TIME_H)
check_include_file(unistd.h HAVE_UNISTD_H)

include(CheckFunctionExists)
check_function_exists(accept4 HAVE_ACCEPT4)
check_function_exists(fork HAVE_FORK)
check_function_exists(getaddrinfo HAVE_GETADDRINFO)
check_function_exists(gettimeofday HAVE_GETTIMEOFDAY)
check_function_exists(inet_ntoa HAVE_INET_NTOA)
check_function_exists(malloc HAVE_MALLOC)
check_function_exists(memset HAVE_MEMSET)
check_function_exists(select HAVE_SELECT)
check_function_exists(socket HAVE_SOCKET)
check_function_exists(strerror HAVE_STRERROR)
check_function_exists(strlcpy HAVE_STRLCPY)

include(CheckSymbolExists)
check_symbol_exists(TIOCSRS485 sys/ioctl.h HAVE_TIOCRS485)
check_symbol_exists(TIOCM_RTS sys/ioctl.h HAVE_TIOCM_RTS)

configure_file(${CMAKE_CURRENT_SOURCE_DIR}/config.h.cmake ${CMAKE_CURRENT_BINARY_DIR}/config.h)

if(CMAKE_COMPILER_IS_GNUCC)
add_definitions(-D_GNU_SOURCE)
endif()

#### BUILD OPTIONS ####

set(MODBUS_SRC_DIR ${CMAKE_CURRENT_SOURCE_DIR}/src)

set(MODBUS_SOURCES
    ${MODBUS_SRC_DIR}/modbus-data.c
    ${MODBUS_SRC_DIR}/modbus-rtu.c
    ${MODBUS_SRC_DIR}/modbus-tcp.c
    ${MODBUS_SRC_DIR}/modbus.c
)

set(MODBUS_PUBLIC_HEADERS
    ${MODBUS_SRC_DIR}/modbus-rtu.h
    ${MODBUS_SRC_DIR}/modbus-tcp.h
    ${MODBUS_SRC_DIR}/modbus.h
    ${CMAKE_CURRENT_BINARY_DIR}/modbus-version.h
)

include_directories(${CMAKE_CURRENT_BINARY_DIR})

add_library(modbus STATIC ${MODBUS_SOURCES})

target_include_directories (modbus PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/src)

set_target_properties(modbus PROPERTIES PUBLIC_HEADER "${MODBUS_PUBLIC_HEADERS}")
if(WIN32)
  target_link_libraries(modbus wsock32 ws2_32)
endif()

set(LIBMODBUS_SOVERSION_VERSION 5.1.0)
set(LIBMODBUS_SOVERSION_SOVERSION 5)
set_target_properties(modbus PROPERTIES
                        VERSION ${LIBMODBUS_SOVERSION_VERSION}
                        SOVERSION ${LIBMODBUS_SOVERSION_SOVERSION})

#### INSTALL ####

include(GNUInstallDirs)

install(TARGETS modbus
        LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
        PUBLIC_HEADER DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/modbus)

set(prefix ${CMAKE_INSTALL_PREFIX})
set(exec_prefix ${CMAKE_INSTALL_PREFIX})
set(libdir "\${prefix}/${CMAKE_INSTALL_LIBDIR}")
set(includedir "\${prefix}/${CMAKE_INSTALL_INCLUDEDIR}")

configure_file(${CMAKE_CURRENT_SOURCE_DIR}/libmodbus.pc.in ${CMAKE_CURRENT_BINARY_DIR}/libmodbus.pc @ONLY)

install(FILES ${CMAKE_CURRENT_BINARY_DIR}/libmodbus.pc DESTINATION ${CMAKE_INSTALL_LIBDIR}/pkgconfig)

