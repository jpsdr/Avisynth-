CMAKE_MINIMUM_REQUIRED( VERSION 3.6.2...3.11 )

set(PluginName "DirectShowSource")
set(ProjectName "Plugin${PluginName}")

# Sensible defaults that should just work if WINSDK is installed and baseclasses built
set(DEFAULT_BASECLASSES_PATH "F:/PRG/Visual_2019/AviSynth")
if(CMAKE_SIZEOF_VOID_P EQUAL 4) # 32-bit
  set(DEFAULT_BASECLASSES_LIB "${DEFAULT_BASECLASSES_PATH}/Deps/DSS/lib/x86/strmbase.lib")
else() # 64-bit
  set(DEFAULT_BASECLASSES_LIB "${DEFAULT_BASECLASSES_PATH}/Deps/DSS/lib/x64/strmbase.lib")
endif()

# We need these variables set by the user to compile successfully
set(DSHOWSRC_BASECLASSES_PATH "${DEFAULT_BASECLASSES_PATH}/DSS_Baseclasses/Build" CACHE STRING   "Folder path to the DirectShow example baseclasses.")
set(DSHOWSRC_BASECLASSES_LIB  "${DEFAULT_BASECLASSES_LIB}"  CACHE FILEPATH "File path to the DirectShow example baseclasses precompiled static library ('strmbase.lib').")

# Create library
project(${ProjectName} VERSION ${PROJECT_VERSION} LANGUAGES CXX)
list (APPEND SourceFiles
    "directshow_source.cpp"
    "directshow_source.h"
)
add_library(${ProjectName} SHARED ${SourceFiles})
set_target_properties(${ProjectName} PROPERTIES "OUTPUT_NAME" ${PluginName})
if (MINGW)
  set_target_properties(${ProjectName} PROPERTIES PREFIX "")
  set_target_properties(${ProjectName} PROPERTIES IMPORT_PREFIX "")
endif()

# Library dependencies
target_link_libraries(${ProjectName} "Winmm.lib" "Quartz.lib" "Ole32.lib" "User32.lib" "Oleaut32.lib" "Advapi32.lib" ${DSHOWSRC_BASECLASSES_LIB})

# Include directories
target_include_directories(${ProjectName} PRIVATE ${AvsCore_SOURCE_DIR} ${DSHOWSRC_BASECLASSES_PATH})

if (MSVC_IDE)
  # Old v7 SDK code won't compile otherwise
  set(DELETE_THIS "/permissive-")
  STRING( REPLACE "${DELETE_THIS}" "" CMAKE_C_FLAGS "${CMAKE_C_FLAGS}")
  STRING( REPLACE "${DELETE_THIS}" "" CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS}")

  # Copy output to a common folder for easy deployment
  add_custom_command(
    TARGET ${ProjectName}
    POST_BUILD
    COMMAND xcopy /Y \"$(TargetPath)\" \"${CMAKE_BINARY_DIR}/Output/plugins\"
  )
endif()

INSTALL(TARGETS "${ProjectName}"
        DESTINATION "${CORE_PLUGIN_INSTALL_PATH}/avisynth")
