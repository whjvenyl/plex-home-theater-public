set(helper_srcs
  XBMCDebugHelpers.h
  XBMCHelper.h
  XBMCHelper.m
  xbmcclientwrapper.h
  xbmcclientwrapper.mm
  xbmchelper_main.mm
  HIDRemote/HIDRemote.m
  HIDRemote/HIDRemote.h
)

add_executable(PlexHelper ${helper_srcs})
target_link_libraries(PlexHelper
  ${CONFIG_LIBRARY_COCOA}
  ${CONFIG_LIBRARY_IOKIT}
  ${CONFIG_LIBRARY_STDC++}
  ${CONFIG_LIBRARY_CARBON}
)

install(TARGETS PlexHelper DESTINATION "Plex Home Theater.app/Contents/Resources/XBMC/tools/darwin/runtime")
