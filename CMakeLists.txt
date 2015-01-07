cmake_minimum_required(VERSION 2.8)

project(flam3boinc-build NONE)

include(ExternalProject)

if(WIN32)
    message(FATAL_ERROR "This build system only works on Unix for now")
else()
    ExternalProject_Add(ZLIB
        URL http://zlib.net/zlib-1.2.8.tar.gz
        CONFIGURE_COMMAND ./configure --static --prefix=<INSTALL_DIR>
        BUILD_IN_SOURCE 1
    )
    ExternalProject_Get_Property(ZLIB INSTALL_DIR)
    set(ZLIB_PREFIX ${INSTALL_DIR})

    ExternalProject_Add(LibPNG
        URL http://download.sourceforge.net/libpng/libpng-1.2.52.tar.gz
        CONFIGURE_COMMAND <SOURCE_DIR>/configure --prefix=<INSTALL_DIR> --disable-shared --with-zlib=${ZLIB_PREFIX}
        DEPENDS ZLIB
    )
    ExternalProject_Get_Property(LibPNG INSTALL_DIR)
    set(PNG_PREFIX ${INSTALL_DIR})

    ExternalProject_Add(LibXml2
        URL ftp://xmlsoft.org/libxml2/libxml2-2.9.2.tar.gz
        CONFIGURE_COMMAND <SOURCE_DIR>/configure --prefix=<INSTALL_DIR> --disable-shared --without-lzma --with-zlib=${ZLIB_PREFIX} --without-python
        DEPENDS ZLIB
    )
    ExternalProject_Get_Property(LibXml2 INSTALL_DIR)
    set(XML2_PREFIX ${INSTALL_DIR})

    ExternalProject_Add(BOINC
        GIT_REPOSITORY git://boinc.berkeley.edu/boinc-v2.git
        CONFIGURE_COMMAND bash -c "(cd <SOURCE_DIR> && autoreconf -if) && <SOURCE_DIR>/configure --prefix=<INSTALL_DIR> --disable-shared --disable-client --disable-server --disable-manager"
    )
    ExternalProject_Get_Property(BOINC INSTALL_DIR)
    set(BOINC_PREFIX ${INSTALL_DIR})

    ExternalProject_Add(flam3-boinc
        GIT_REPOSITORY git://github.com/nicolas17/flam3-boinc
        CMAKE_CACHE_ARGS -DCMAKE_PREFIX_PATH:STRING=${ZLIB_PREFIX};${PNG_PREFIX};${XML2_PREFIX};${BOINC_PREFIX}
        INSTALL_COMMAND true
        DEPENDS LibXml2 LibPNG BOINC
    )
endif()
