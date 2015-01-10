cmake_minimum_required(VERSION 2.8)

project(flam3boinc-build NONE)

include(ExternalProject)

if(WIN32)
    include(patch_win32.cmake)
else()
    set(PATCH_PROGRAM patch)
    add_custom_target(patch) # for "DEPENDS patch" to work
endif()

ExternalProject_Add(ZLIB
    URL http://download.sourceforge.net/libpng/zlib/1.2.8/zlib-1.2.8.tar.gz
    URL_HASH SHA1=a4d316c404ff54ca545ea71a27af7dbc29817088
    PATCH_COMMAND ${PATCH_PROGRAM} -p1 < ${CMAKE_SOURCE_DIR}/zlib-build-system.diff
    CMAKE_CACHE_ARGS
        -DCMAKE_INSTALL_PREFIX:PATH=<INSTALL_DIR>
        -DCMAKE_BUILD_TYPE:STRING=${CMAKE_BUILD_TYPE}
    DEPENDS patch
)
ExternalProject_Get_Property(ZLIB INSTALL_DIR)
set(ZLIB_PREFIX ${INSTALL_DIR})

ExternalProject_Add(LibPNG
    URL http://download.sourceforge.net/libpng/libpng-1.2.52.tar.gz
    URL_HASH SHA1=596a7dbc178861dab5761d74e6a4bfe66c0c31d1
    CMAKE_CACHE_ARGS
        -DCMAKE_INSTALL_PREFIX:PATH=<INSTALL_DIR>
        -DCMAKE_PREFIX_PATH:STRING=${ZLIB_PREFIX}
        -DCMAKE_BUILD_TYPE:STRING=${CMAKE_BUILD_TYPE}
        -DPNG_SHARED:BOOL=OFF
        -DPNG_STATIC:BOOL=ON
    DEPENDS ZLIB
)
ExternalProject_Get_Property(LibPNG INSTALL_DIR)
set(PNG_PREFIX ${INSTALL_DIR})

ExternalProject_Add(LibXml2
    URL ftp://xmlsoft.org/libxml2/libxml2-2.8.0.tar.gz
    URL_HASH SHA1=a0c553bd51ba79ab6fff26dc700004c6a41f5250
    PATCH_COMMAND ${PATCH_PROGRAM} -p1 < ${CMAKE_SOURCE_DIR}/libxml2-2.8.0.diff
          COMMAND ${PATCH_PROGRAM} -p1 < ${CMAKE_SOURCE_DIR}/libxml2-build-static.diff
    CMAKE_CACHE_ARGS
        -DCMAKE_INSTALL_PREFIX:PATH=<INSTALL_DIR>
        -DCMAKE_PREFIX_PATH:STRING=${ZLIB_PREFIX}
        -DCMAKE_BUILD_TYPE:STRING=${CMAKE_BUILD_TYPE}
        -DBUILD_tests:BOOL=OFF
    DEPENDS ZLIB
)
ExternalProject_Get_Property(LibXml2 INSTALL_DIR)
set(XML2_PREFIX ${INSTALL_DIR})

ExternalProject_Add(BOINC
    GIT_REPOSITORY git://github.com/nicolas17/boinc
    GIT_TAG cmake
    CMAKE_CACHE_ARGS
        -DCMAKE_INSTALL_PREFIX:PATH=<INSTALL_DIR>
        -DCMAKE_BUILD_TYPE:STRING=${CMAKE_BUILD_TYPE}
)
ExternalProject_Get_Property(BOINC INSTALL_DIR)
set(BOINC_PREFIX ${INSTALL_DIR})

ExternalProject_Add(flam3-boinc
    GIT_REPOSITORY git://github.com/nicolas17/flam3-boinc
    CMAKE_CACHE_ARGS
        -DCMAKE_PREFIX_PATH:STRING=${ZLIB_PREFIX};${PNG_PREFIX};${XML2_PREFIX};${BOINC_PREFIX}
        -DCMAKE_BUILD_TYPE:STRING=${CMAKE_BUILD_TYPE}
    INSTALL_COMMAND true
    DEPENDS LibXml2 LibPNG BOINC
)
