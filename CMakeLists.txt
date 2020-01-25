cmake_minimum_required(VERSION 3.8)

include("cmake/HunterGate.cmake")
HunterGate(
    URL "https://github.com/Bjoe/hunter/archive/8ae4222ecc50eb4764a16a80c6965fdc8ee46c8d.tar.gz"
    SHA1 "7e118c65b9cb95cf22be3954124a1c76216aa2fe"
)

list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake")
project(ewig)

option(EWIG_ENABLE_DEBUG "Enable debug support in ewig" OFF)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED on)
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -Werror")
set(CMAKE_CXX_EXTENSIONS off)
set(CMAKE_EXPORT_COMPILE_COMMANDS on)

set(CMAKE_CURSES_NEED_WIDE true)

hunter_add_package(Boost COMPONENTS system)
hunter_add_package(scelta)
hunter_add_package(ncursesw)
hunter_add_package(Lager)
hunter_add_package(Immer)
hunter_add_package(cereal)
hunter_add_package(utf8)
find_package(cereal CONFIG REQUIRED)
find_package(ncursesw CONFIG REQUIRED)
#find_package(Curses REQUIRED)
find_package(Boost 1.56 REQUIRED system)
find_package(Threads)
find_package(Immer CONFIG REQUIRED)
find_package(Lager CONFIG REQUIRED)
find_package(utf8 CONFIG REQUIRED)
#find_path(SCELTA_INCLUDE_DIR scelta.hpp)
#find_path(UTFCPP_INCLUDE_DIR utf8.h)

set(ewig_sources
  src/ewig/application.cpp
  src/ewig/buffer.cpp
  src/ewig/draw.cpp
  src/ewig/keys.cpp
  src/ewig/terminal.cpp
  src/ewig/main.cpp)
set(ewig_include_directories
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/src>
  $<INSTALL_INTERFACE:include>)
set(ewig_system_include_directories
  ${CURSES_INCLUDE_DIR}
  ${Boost_INCLUDE_DIR}
#  ${SCELTA_INCLUDE_DIR}
#  ${UTFCPP_INCLUDE_DIR}
)
set(ewig_link_libraries
  immer
  lager
  ${CURSES_LIBRARIES}
  ${Boost_LIBRARIES}
  ${CMAKE_THREAD_LIBS_INIT}
  cereal::cereal)

add_executable(ewig ${ewig_sources})
target_include_directories(ewig PUBLIC ${ewig_include_directories})
target_include_directories(ewig SYSTEM PUBLIC ${ewig_system_include_directories})
target_link_libraries(ewig ${ewig_link_libraries})

install(TARGETS ewig DESTINATION bin)

if(EWIG_ENABLE_DEBUG)
    find_package(LibHttpServer)

    add_executable(ewig-debug ${ewig_sources})
    target_compile_definitions(ewig-debug PUBLIC -DEWIG_ENABLE_DEBUGGER=1)
    target_include_directories(ewig-debug PUBLIC ${ewig_include_directories})
    target_include_directories(ewig-debug SYSTEM PUBLIC ${ewig_system_include_directories}
      ${LIBHTTPSERVER_INCLUDE_DIRS})
    target_link_libraries(ewig-debug ${ewig_link_libraries}
      ${LIBHTTPSERVER_LIBRARIES})
endif()
