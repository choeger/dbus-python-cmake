project(dbus_python CXX C)
cmake_minimum_required(VERSION 2.8)
  
set(CMAKE_MODULE_PATH "${CMAKE_SOURCE_DIR}/cmake")

set(PACKAGE "dbus-python")
set(PACKAGE_BUGREPORT "http://bugs.freedesktop.org/enter_bug.cgi?product=dbus&component=python")
set(PACKAGE_NAME "dbus-python")
set(PACKAGE_URL "")
set(PACKAGE_VERSION "1.1.0")
set(PACKAGE_STRING "${PACKAGE_NAME} ${PACKAGE_VERSION}")
set(PACKAGE_TARNAME "${PACKAGE_NAME}")

configure_file(${CMAKE_CURRENT_SOURCE_DIR}/config.h.cmake ${CMAKE_CURRENT_BINARY_DIR}/config.h)
include_directories(${CMAKE_CURRENT_BINARY_DIR})
						       
#DBUS
find_package(DBUS REQUIRED)

#GLIB
find_package(GLIB REQUIRED)
			       
# Find Python executable
find_package(PythonLibs REQUIRED)
find_package(PythonInterp REQUIRED)
if(NOT PYTHONLIBS_FOUND OR NOT PYTHON_EXECUTABLE)
  message(SEND_ERROR "You need Python to build the dbus python bindings")
endif(NOT PYTHONLIBS_FOUND OR NOT PYTHON_EXECUTABLE)

# The code below prints the Python extension for the current system
set(get_python_suffix "import imp ; print(filter(lambda s : s[1] == 'rb' and s[0][0] == '.', imp.get_suffixes())[0][0])")

execute_process(
  COMMAND ${PYTHON_EXECUTABLE} -c "${get_python_suffix}"
  OUTPUT_VARIABLE _modsuffix
)
string(REPLACE "\n" "" _modsuffix ${_modsuffix}) 
message(STATUS "Python module suffix is: ${_modsuffix}")

# The code below returns the site-packages directory for the current system
set(get_site_package_dir "from distutils.sysconfig import get_python_lib; print get_python_lib()")
execute_process(
  COMMAND ${PYTHON_EXECUTABLE} -c "${get_site_package_dir}"
  OUTPUT_VARIABLE site_pkg
)
string(REPLACE "\n" "" site_pkg ${site_pkg}) 
message(STATUS "Python module path is: ${site_pkg}")

include_directories(include/ ${PYTHON_INCLUDE_DIRS} ${DBUS_INCLUDES})

include_directories(_dbus_bindings/)
file(GLOB dbus_binding_sources _dbus_bindings/*.c)
add_library(_dbus_bindings SHARED ${dbus_binding_sources})
target_link_libraries(_dbus_bindings ${PYTHON_LIBRARIES} ${DBUS_LIBRARY})

include_directories(_dbus_glib_bindings/ ${GLIB_INCLUDE_DIR} ${GLIB_CONFIG_INCLUDE_DIR})
file(GLOB dbus_glib_binding_sources _dbus_glib_bindings/*.c)
add_library(_dbus_glib_bindings SHARED ${dbus_glib_binding_sources})
target_link_libraries(_dbus_glib_bindings ${PYTHON_LIBRARIES} ${DBUS_LIBRARY} ${DBUS_GLIB_LIBRARY} ${GLIB_LIBRARIES})

file(GLOB dbus_py_test_sources test/*.c)
add_library(dbus_py_test SHARED ${dbus_py_test_sources})
target_link_libraries(dbus_py_test ${PYTHON_LIBRARIES} ${DBUS_LIBRARY})

set_target_properties(_dbus_bindings _dbus_glib_bindings dbus_py_test
 PROPERTIES
   PREFIX "" # There is no prefix even on UNIXes
   SUFFIX "${_modsuffix}" # The extension got from Python libraries
)

set(PYTHON ${PYTHON_EXECUTABLE})
set(abs_top_srcdir ${CMAKE_CURRENT_SOURCE_DIR})
set(abs_top_builddir ${CMAKE_CURRENT_BINARY_DIR})

configure_file(${CMAKE_CURRENT_SOURCE_DIR}/test/tmp-session-bus.conf.in  ${CMAKE_CURRENT_BINARY_DIR}/test/tmp-session-bus.conf)
configure_file(${CMAKE_CURRENT_SOURCE_DIR}/test/TestSuitePythonService.service.in  ${CMAKE_CURRENT_BINARY_DIR}/test/TestSuitePythonService.service)
