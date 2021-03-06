
project(MacFreeRDP)

set(MODULE_NAME "MacFreeRDP")
set(MODULE_PREFIX "FREERDP_CLIENT_MAC")

find_required_package(OpenSSL)

set(FRAMEWORK_HEADERS_PATH /System/Library/Frameworks/Cocoa.framework/Versions/A/Headers/)
include_directories(${FRAMEWORK_HEADERS_PATH} /System/Library/Frameworks) 
	
# set(CMAKE_OSX_SYSROOT MacOSX10.7.sdk) # uncomment to specify SDK version
set(CMAKE_SHARED_LINKER_FLAGS "${CMAKE_SHARED_LINKER_FLAGS} -mmacosx-version-min=10.4")
set(GUI_TYPE MACOSX_BUNDLE)
	
# Import libraries
find_library(FOUNDATION_LIBRARY Foundation)
message("+ Using foundation library ${FOUNDATION_LIBRARY}")
find_library(COCOA_LIBRARY Cocoa)
message("+ Using cocoa library ${COCOA_LIBRARY}")
find_library(APPKIT_LIBRARY AppKit)
message("+ Using appkit library ${APPKIT_LIBRARY}")
	
message(" Current source dir: ${CMAKE_CURRENT_SOURCE_DIR}")
# Set the OS X Bundle specific CMake variables which will be used to populate the plist for
# the application bundle
set(MACOSX_BUNDLE_INFO_STRING "MacFreeRDP")
set(MACOSX_BUNDLE_GUI_IDENTIFIER "com.freerdp.mac")
set(MACOSX_BUNDLE_BUNDLE_IDENTIFIER "FreeRDP.Mac")
set(MACOSX_BUNDLE_LONG_VERSION_STRING "MacFreeRDP Version 1.0.1")
set(MACOSX_BUNDLE_BUNDLE_NAME "MacFreeRDP")
set(MACOSX_BUNDLE_SHORT_VERSION_STRING 1.0.1)
set(MACOSX_BUNDLE_BUNDLE_VERSION 1.0.1)
set(MACOSX_BUNDLE_COPYRIGHT "Copyright 2012. All Rights Reserved.")

# Specific plist and NOT standard CMake variables
set(MACOSX_BUNDLE_NSMAIN_NIB_FILE "MainMenu")
set(MACOSX_BUNDLE_NSPRINCIPAL_CLASS "NSApplication")
	
mark_as_advanced(COCOA_LIBRARY FOUNDATION_LIBRARY APPKIT_LIBRARY)
set(EXTRA_LIBS ${COCOA_LIBRARY} ${FOUNDATION_LIBRARY} ${APPKIT_LIBRARY})
set(APP_TYPE MACOSX_BUNDLE)

# OS X Interface Builder files
file(GLOB MacFreeRDP_XIBS *.xib)

# Headers
file(GLOB MacFreeRDP_Headers *.h)

# Source
file(GLOB MacFreeRDP_Source *.m)

add_executable(MacFreeRDP
	${APP_TYPE} 
	${MacFreeRDP_Headers}
	${MacFreeRDP_Source}
	${MacFreeRDP_XIBS})

# This is necessary for the xib file part below
configure_file(${CMAKE_CURRENT_SOURCE_DIR}/Info.plist ${CMAKE_CURRENT_BINARY_DIR}/Info.plist)

# This allows for automatic xib to nib ibitool
set_target_properties(MacFreeRDP PROPERTIES RESOURCE "${MacFreeRDP_XIBS}")

# Automatic ref counting
# temporary turn off for x86_64 build issue
# set_target_properties(MacFreeRDP PROPERTIES XCODE_ATTRIBUTE_CLANG_ENABLE_OBJC_ARC YES)

# Support for automatic reference counting requires non-fragile abi.
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -fobjc-nonfragile-abi")

# XCode project architecture to native architecture of build machine
# -----------------------------------------------------------------------------------------------------
# Issue: Had some issues with FreeRDP project building only 64 bit and
# MacFreeRDP attempting to link to both 32 and 64 for dual target.
# In the future the FreeRDP Xcode project should be pulled in for a couple of reasons:
# 1) better step-into debugging 2) automatic dependency compilation and multi-arch compilation + linkage
# If you know the solutions for 1 and 2, please add below.
set_target_properties(MacFreeRDP PROPERTIES XCODE_ATTRIBUTE_ARCHS "$(NATIVE_ARCH_ACTUAL)")

# Set the info plist to the custom instance
set_target_properties(MacFreeRDP PROPERTIES MACOSX_BUNDLE_INFO_PLIST ${CMAKE_CURRENT_BINARY_DIR}/Info.plist)

#find_optional_package(MacAudio)

set(${MODULE_PREFIX}_LIBS ${${MODULE_PREFIX}_LIBS} ${EXTRA_LIBS})

set(${MODULE_PREFIX}_LIBS ${${MODULE_PREFIX}_LIBS} freerdp-client)

set_complex_link_libraries(VARIABLE ${MODULE_PREFIX}_LIBS MONOLITHIC ${MONOLITHIC_BUILD}
	MODULE freerdp
	MODULES freerdp-core freerdp-cache freerdp-gdi freerdp-codec freerdp-rail freerdp-utils)

target_link_libraries(${MODULE_NAME} ${${MODULE_PREFIX}_LIBS})

set_property(TARGET MacFreeRDP PROPERTY FOLDER "Client/Mac")
