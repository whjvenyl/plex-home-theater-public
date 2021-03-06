if(NOT WIN32)
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fPIC -fno-common")
  set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -fPIC -fno-common")
  add_definitions(-D_DLL -DXBMC)
  include_directories(${dependdir}/include)
else()
  set(COMPILE_DEFINITIONS "")
  add_definitions(-D_DLL -DWIN32 -D_WINDOWS -D_USRDLL -D_AFXDLL -DJAS_WIN_MSVC_BUILD -D_CRT_SECURE_NO_DEPRECATE -D_CONSOLE)
  set(CMAKE_C_FLAGS_DEBUG "/MP /Od /Oy- /D_DEBUG /Gm- /MDd /GS /arch:SSE /fp:precise /Zc:wchar_t /Zc:forScope /wd\"4996\"")
  set(CMAKE_CXX_FLAGS_DEBUG ${CMAKE_C_FLAGS_DEBUG})
  set(CMAKE_C_FLAGS_RELEASE " /WX- /O2 /Ob1 /Oy- /DNDEBUG /GF /Gm- /EHa /MD /GS- /Gy /arch:SSE /fp:precise /Zc:wchar_t /Zc:forScope")
  set(CMAKE_CXX_FLAGS_RELEASE ${CMAKE_C_FLAGS_RELEASE})
endif()

if(APPLE)
  set(CMAKE_MODULE_LINKER_FLAGS "${CMAKE_MODULE_LINKER_FLAGS} -Wl,-alias_list ${root}/xbmc/cores/DllLoader/exports/wrapper_mach_alias")
elseif(UNIX)
  set(CMAKE_MODULE_LINKER_FLAGS "${CMAKE_MODULE_LINKER_FLAGS} -Wl,--unresolved-symbols=ignore-all -Wl,-wrap,_IO_getc -Wl,-wrap,_IO_getc_unlocked -Wl,-wrap,_IO_putc -Wl,-wrap,__fgets_chk -Wl,-wrap,__fprintf_chk -Wl,-wrap,__fread_chk -Wl,-wrap,__fxstat64 -Wl,-wrap,__lxstat64 -Wl,-wrap,__printf_chk -Wl,-wrap,__read_chk -Wl,-wrap,__vfprintf_chk -Wl,-wrap,__xstat64 -Wl,-wrap,_stat -Wl,-wrap,calloc -Wl,-wrap,clearerr -Wl,-wrap,close -Wl,-wrap,closedir -Wl,-wrap,dlopen -Wl,-wrap,fclose -Wl,-wrap,fdopen -Wl,-wrap,feof -Wl,-wrap,ferror -Wl,-wrap,fflush -Wl,-wrap,fgetc -Wl,-wrap,fgetpos -Wl,-wrap,fgetpos64 -Wl,-wrap,fgets -Wl,-wrap,fileno -Wl,-wrap,flockfile -Wl,-wrap,fopen -Wl,-wrap,fopen64 -Wl,-wrap,fprintf -Wl,-wrap,fputc -Wl,-wrap,fputs -Wl,-wrap,fread -Wl,-wrap,free -Wl,-wrap,freopen -Wl,-wrap,fseek -Wl,-wrap,fseeko64 -Wl,-wrap,fsetpos -Wl,-wrap,fsetpos64 -Wl,-wrap,fstat -Wl,-wrap,ftell -Wl,-wrap,ftello64 -Wl,-wrap,ftrylockfile -Wl,-wrap,funlockfile -Wl,-wrap,fwrite -Wl,-wrap,getc -Wl,-wrap,getc_unlocked -Wl,-wrap,getmntent -Wl,-wrap,ioctl -Wl,-wrap,lseek -Wl,-wrap,lseek64 -Wl,-wrap,malloc -Wl,-wrap,open -Wl,-wrap,open64 -Wl,-wrap,opendir -Wl,-wrap,popen -Wl,-wrap,printf -Wl,-wrap,read -Wl,-wrap,readdir -Wl,-wrap,readdir64 -Wl,-wrap,realloc -Wl,-wrap,rewind -Wl,-wrap,rewinddir -Wl,-wrap,setvbuf -Wl,-wrap,ungetc -Wl,-wrap,vfprintf -Wl,-wrap,write")
endif()

#        $(CXX) $(CFLAGS) $(LDFLAGS) -shared -o $(SLIB) -Wl,--whole-archive $(SLIBS) \
#                -Wl,--no-whole-archive -lpng -ljasper -ljpeg -ltiff -lz -Wl,--unresolved-symbols=ignore-all \
#                                `cat ../../xbmc/cores/DllLoader/exports/wrapper.def` \
#                                                ../../xbmc/cores/DllLoader/exports/wrapper.o

if(WIN32)
  # Use the zlib that comes with cximage
  include_directories(zlib zlib/win32)
endif()

add_subdirectory(CxImage)
add_subdirectory(jbig)
add_subdirectory(raw)
add_subdirectory(jasper)

set(OBJECTS
  cximage
  cx_jbig
  cx_raw
  cx_jasper
)
set(SRC CxImage/DllInterface.cpp)

if(NOT WIN32)
  list(APPEND OBJECTS wrapper)
  add_library(ImageLib-${ARCH} MODULE ${SRC})
  target_link_libraries(ImageLib-${ARCH} ${PLEX_LINK_WHOLEARCHIVE} ${OBJECTS} ${PLEX_LINK_NOWHOLEARCHIVE} ${CONFIG_LIBRARY_Z} ${CONFIG_LIBRARY_PNG} ${CONFIG_LIBRARY_JPEG} ${CONFIG_LIBRARY_TIFF})
else()
  add_subdirectory(zlib)
  add_subdirectory(jpeg)
  add_subdirectory(mng)
  add_subdirectory(tiff)
  add_subdirectory(png)

  set(OBJECTS ${OBJECTS}
    cx_mng
    cx_jpeg
    cx_png
    cx_tiff
    cx_zlib
  )

  add_library(ImageLib SHARED ${SRC})
  target_link_libraries(ImageLib ${OBJECTS})

  set(LIBS kernel32 user32 gdi32 winspool comdlg32 advapi32 shell32 ole32 oleaut32 uuid odbc32 odbccp32)
  foreach(l ${LIBS})
    find_library(t ${l})
    target_link_libraries(ImageLib ${t})
  endforeach()
endif()

if(APPLE)
  install(TARGETS ImageLib-${ARCH} DESTINATION ${RESOURCEPATH}/system COMPONENT RUNTIME)
elseif(UNIX)
  install(TARGETS ImageLib-${ARCH} DESTINATION ${BINPATH}/system COMPONENT RUNTIME)
elseif(WIN32)
  install(TARGETS ImageLib RUNTIME DESTINATION ${RESOURCEPATH}/system LIBRARY DESTINATION ${RESOURCEPATH}/system COMPONENT RUNTIME)
endif()
