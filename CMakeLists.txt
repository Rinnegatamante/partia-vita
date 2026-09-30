cmake_minimum_required(VERSION 2.8)

if(NOT DEFINED CMAKE_TOOLCHAIN_FILE)
  if(DEFINED ENV{VITASDK})
    set(CMAKE_TOOLCHAIN_FILE "$ENV{VITASDK}/share/vita.toolchain.cmake" CACHE PATH "toolchain file")
  else()
    message(FATAL_ERROR "Please define VITASDK to point to your SDK path!")
  endif()
endif()

project(Partia C CXX)
include("${VITASDK}/share/vita.cmake" REQUIRED)
find_package(PkgConfig REQUIRED)
pkg_check_modules(PARTIA_DEPS REQUIRED SDL2_mixer SDL2_image sndfile vorbisfile openal)
set(VITA_APP_NAME "Partia: The Granian Trilogy")
set(VITA_TITLEID  "PARTIAVGL")
set(VITA_MKSFOEX_FLAGS "-d ATTRIBUTE2=12")

set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -g -Wl,-q,--wrap,memcpy,--wrap,memmove,--wrap,memset,--allow-multiple-definition -Wall -O3 -fdiagnostics-color=always -fno-optimize-sibling-calls -mfloat-abi=softfp")
set(CMAKE_CXX_FLAGS "${CMAKE_C_FLAGS} -std=c++11 -Wno-write-strings")

add_executable(Partia
  loader/main.c
  loader/dialog.c
  loader/so_util.c
  loader/sha1.c
  loader/ctype_patch.c
)

set_target_properties(Partia PROPERTIES LINKER_LANGUAGE CXX)
target_compile_options(Partia PRIVATE ${PARTIA_DEPS_STATIC_CFLAGS})

target_link_libraries(Partia
  -pthread
  -Wl,--start-group
  ${PARTIA_DEPS_STATIC_LDFLAGS}
  SDL2_mixer
  SDL2_image
  SDL2
  png
  webp
  jpeg
  mikmod
  z
  m
  stdc++
  OpenSLES
  sndfile
  vorbis
  vorbisenc
  vorbisfile
  ogg
  FLAC
  openal
  imgui
  vitaGL
  vitashark
  SceShaccCgExt
  mathneon
  mpg123
  taihen_stub
  kubridge_stub
  SceAppMgr_stub
  SceAppUtil_stub
  SceAudio_stub
  SceAudioIn_stub
  SceCtrl_stub
  SceCommonDialog_stub
  SceDisplay_stub
  SceFios2_stub
  SceGxm_stub
  SceShaccCg_stub
  SceSysmodule_stub
  ScePower_stub
  SceTouch_stub
  SceMotion_stub
  SceHid_stub
  SceVshBridge_stub
  SceKernelDmacMgr_stub
  -Wl,--end-group
)

add_executable(companion
  selector/main.c
  loader/dialog.c
)

set_target_properties(companion PROPERTIES LINKER_LANGUAGE CXX)
target_compile_options(companion PRIVATE ${PARTIA_DEPS_STATIC_CFLAGS})

# Reserve space for vita-elf-create import metadata between load segments.
set_property(TARGET companion APPEND_STRING PROPERTY LINK_FLAGS " -Wl,--defsym=__sce_headroom=0x10000")

target_link_libraries(companion
  -pthread
  -Wl,--start-group
  ${PARTIA_DEPS_STATIC_LDFLAGS}
  SDL2_mixer
  SDL2_image
  SDL2
  png
  webp
  jpeg
  mikmod
  z
  m
  stdc++
  OpenSLES
  sndfile
  vorbis
  vorbisenc
  vorbisfile
  ogg
  FLAC
  openal
  imgui
  vitaGL
  vitashark
  SceShaccCgExt
  mathneon
  mpg123
  taihen_stub
  kubridge_stub
  SceAppMgr_stub
  SceAppUtil_stub
  SceAudio_stub
  SceAudioIn_stub
  SceCtrl_stub
  SceCommonDialog_stub
  SceDisplay_stub
  SceFios2_stub
  SceGxm_stub
  SceShaccCg_stub
  SceSysmodule_stub
  ScePower_stub
  SceTouch_stub
  SceMotion_stub
  SceHid_stub
  SceVshBridge_stub
  SceKernelDmacMgr_stub
  -Wl,--end-group
)

vita_create_self(eboot.bin companion UNSAFE)

vita_create_self(game.bin Partia UNSAFE)
vita_create_vpk(Partia.vpk ${VITA_TITLEID} eboot.bin
  VERSION ${VITA_VERSION}
  NAME ${VITA_APP_NAME}
  FILE ${CMAKE_SOURCE_DIR}/sce_sys/icon0.png sce_sys/icon0.png
       ${CMAKE_SOURCE_DIR}/sce_sys/pic0.png sce_sys/pic0.png
       ${CMAKE_SOURCE_DIR}/sce_sys/livearea/contents/bg0.png sce_sys/livearea/contents/bg0.png
       ${CMAKE_SOURCE_DIR}/sce_sys/livearea/contents/conquest.png sce_sys/livearea/contents/conquest.png
       ${CMAKE_SOURCE_DIR}/sce_sys/livearea/contents/template.xml sce_sys/livearea/contents/template.xml
	   ${CMAKE_SOURCE_DIR}/images/game1.png images/game1.png
	   ${CMAKE_SOURCE_DIR}/images/game2.png images/game2.png
	   ${CMAKE_SOURCE_DIR}/images/game3.png images/game3.png
	   ${CMAKE_BINARY_DIR}/game.bin game.bin
)

# Keep legacy invalid returns diagnostic-only instead of inventing results.
set_property(SOURCE loader/so_util.c APPEND PROPERTY COMPILE_OPTIONS -Wno-error=return-mismatch -Wno-error=incompatible-pointer-types)
