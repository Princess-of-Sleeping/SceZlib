cmake_minimum_required(VERSION 3.0)

if(NOT DEFINED CMAKE_TOOLCHAIN_FILE)
  if(DEFINED ENV{VITASDK})
    set(CMAKE_TOOLCHAIN_FILE "$ENV{VITASDK}/share/vita.toolchain.cmake" CACHE PATH "toolchain file")
  else()
    message(FATAL_ERROR "Please define VITASDK to point to your SDK path!")
  endif()
endif()

project(SceZlib)
include("${VITASDK}/share/vita.cmake" REQUIRED)
include(ExternalProject)

set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Wl,-q -Wall -O2 -fno-inline -fno-builtin")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fno-rtti -fno-exceptions")

enable_language(C ASM)

ExternalProject_add(libz_download
  URL https://zlib.net/zlib-1.2.13.tar.xz
  URL_HASH SHA256=d14c38e313afc35a9a8760dadf26042f51ea0f5d154b0630a31da0540107fb98
  DOWNLOAD_DIR ""
  CONFIGURE_COMMAND ""
  BUILD_COMMAND ""
  INSTALL_COMMAND ""
)

set(libz_src
  libz_download-prefix/src/libz_download/adler32.c
  libz_download-prefix/src/libz_download/compress.c
  libz_download-prefix/src/libz_download/crc32.c
  libz_download-prefix/src/libz_download/deflate.c
  # libz_download-prefix/src/libz_download/gzclose.c
  # libz_download-prefix/src/libz_download/gzlib.c
  # libz_download-prefix/src/libz_download/gzread.c
  # libz_download-prefix/src/libz_download/gzwrite.c
  # libz_download-prefix/src/libz_download/infback.c
  libz_download-prefix/src/libz_download/inffast.c
  libz_download-prefix/src/libz_download/inflate.c
  libz_download-prefix/src/libz_download/inftrees.c
  libz_download-prefix/src/libz_download/trees.c
  libz_download-prefix/src/libz_download/uncompr.c
  libz_download-prefix/src/libz_download/zutil.c
)

add_custom_command(
  OUTPUT ${libz_src}
  DEPENDS ${libz_download}
)

include_directories(
)

add_executable(${PROJECT_NAME}
  src/main.c
  ${libz_src}
)

target_link_libraries(${PROJECT_NAME}
  SceLibc_stub
)

set_target_properties(${PROJECT_NAME}
  PROPERTIES LINK_FLAGS "-nostdlib"
  COMPILE_FLAGS "-D__PSP2_USER__"
)

vita_create_self(zlib.suprx ${PROJECT_NAME}
  CONFIG exports.yml
  STRIPPED
  REL_OPTIMIZE
)

vita_create_stubs(${PROJECT_NAME}_stub ${PROJECT_NAME} ${CMAKE_CURRENT_SOURCE_DIR}/exports.yml KERNEL)

install(FILES build/${PROJECT_NAME}_stub/libSceZlib_stub.a
  DESTINATION lib
)
