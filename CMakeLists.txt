cmake_minimum_required(VERSION 3.31)
set(CMAKE_PROJECT_TOP_LEVEL_INCLUDES .devcontainer/cmake/xproinc.cmake)
project(libiconv VERSION 1.18)
if(DEFINED XP_NAMESPACE)
  set(nameSpace ${XP_NAMESPACE}::)
endif()
set(libName ${nameSpace}${PROJECT_NAME})
if(WIN32)
  install(FILES iconv.h DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})
  install(FILES x64/ReleaseStatic/libiconvStatic.lib DESTINATION ${CMAKE_INSTALL_LIBDIR})
  file(GLOB dbgFiles x64/DebugStatic/libiconvStaticD.*)
  install(FILES ${dbgFiles} DESTINATION ${CMAKE_INSTALL_LIBDIR})
  string(JOIN "\n" EXT1
    "get_filename_component(PKG_ROOTDIR \${CMAKE_CURRENT_LIST_DIR}/../.. ABSOLUTE)"
    "get_filename_component(PKG_ROOTDIR \${PKG_ROOTDIR} ABSOLUTE) # remove relative parts"
    ""
    )
  string(JOIN "\n" EXT2
    "if(NOT TARGET ${libName})"
    "  add_library(${libName} STATIC IMPORTED)"
    "  set(libRelease \${PKG_ROOTDIR}/${CMAKE_INSTALL_LIBDIR}/libiconvStatic${CMAKE_STATIC_LIBRARY_SUFFIX})"
    "  set(libDebug \${PKG_ROOTDIR}/${CMAKE_INSTALL_LIBDIR}/libiconvStaticD${CMAKE_STATIC_LIBRARY_SUFFIX})"
    "  if(EXISTS \"\${libRelease}\" AND EXISTS \"\${libDebug}\")"
    "    set_property(TARGET ${libName} APPEND PROPERTY IMPORTED_CONFIGURATIONS RELEASE DEBUG)"
    "    set_target_properties(${libName} PROPERTIES"
    "      IMPORTED_LINK_INTERFACE_LANGUAGES_RELEASE \"C\""
    "      IMPORTED_LINK_INTERFACE_LANGUAGES_DEBUG \"C\""
    "      IMPORTED_LOCATION_RELEASE \"\${libRelease}\""
    "      IMPORTED_LOCATION_DEBUG \"\${libDebug}\""
    "      INTERFACE_INCLUDE_DIRECTORIES \"\${PKG_ROOTDIR}/${CMAKE_INSTALL_INCLUDEDIR}\""
    "      )"
    "  endif()"
    "endif()"
    ""
    )
else()
  string(JOIN "\n" EXT2
    "if(NOT TARGET ${libName})"
    "  add_library(${libName} INTERFACE IMPORTED)"
    "  find_package(Iconv REQUIRED)"
    "  target_link_libraries(${libName} INTERFACE Iconv::Iconv)"
    "endif()"
    ""
    )
endif()
if(DEFINED XP_NAMESPACE)
  xpExternPackage(NAMESPACE ${XP_NAMESPACE}
    LIBRARIES ${PROJECT_NAME} BASE v0 XPDIFF "bin"
    WEB "https://www.gnu.org/software/libiconv/" UPSTREAM "github.com/pffang/libiconv-for-Windows/releases/tag/1.18-eed6782"
    DESC "character set conversion library"
    LICENSE "[LGPL-2.1-or-later](https://savannah.gnu.org/projects/libiconv/ 'GNU Lesser General Public License v2.1 or later')"
    )
endif()
