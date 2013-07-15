PROJECT(hts)

ENABLE_LANGUAGE(C)

CMAKE_MINIMUM_REQUIRED(VERSION 2.6)

SET(SOURCES htsbuf.c
            htsmsg_binary.c
            htsmsg.c
            htsstr.c
            net_posix.c
            sha1.c)

ADD_DEFINITIONS(-D_GNU_SOURCE)

SET(CMAKE_C_FLAGS ${CMAKE_C_FLAGS} -std=c99)

ADD_LIBRARY(hts STATIC ${SOURCES})
