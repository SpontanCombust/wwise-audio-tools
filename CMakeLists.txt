cmake_minimum_required(VERSION 3.12)
project(wwise-audio-tools)

set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/lib")
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/lib")
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/bin")

file(REMOVE_RECURSE "${CMAKE_BINARY_DIR}/lib")

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_POSITION_INDEPENDENT_CODE ON)

include_directories(
    include
    libogg/include
    libvorbis/include
    ${CMAKE_CURRENT_BINARY_DIR}/libogg/include
  )

add_definitions(-DKS_STR_ENCODING_NONE)

set(SOURCE
    src/codebook.cpp
    src/crc.c
    src/revorb.cpp
    src/ww2ogg.cpp
    src/wwriff.cpp
    src/wwiseaudiotools.cpp
    src/packed_codebooks.cpp
		src/kaitaistream.cpp
		src/bnk.cpp
    src/wwt_bnk.cpp
    src/w3sc.cpp
    src/wwt_w3sc.cpp
  )

add_subdirectory(libogg EXCLUDE_FROM_ALL)
add_subdirectory(libvorbis EXCLUDE_FROM_ALL)

add_library(wwise-audio SHARED ${SOURCE})
add_library(wwise-audio_static STATIC ${SOURCE})
add_executable(wwtools ${SOURCE} src/main.cpp)

target_link_libraries(wwise-audio ogg vorbis)
target_link_libraries(wwise-audio_static ogg vorbis)

# Standard filesystem library needs additional linking options on older compilers
# https://en.cppreference.com/w/cpp/filesystem#Notes
if(CMAKE_CXX_COMPILER_ID STREQUAL "GNU")
  if(CMAKE_CXX_COMPILER_VERSION VERSION_LESS "9.1")
    target_link_libraries(wwtools ogg vorbis stdc++fs)
  else()
    target_link_libraries(wwtools ogg vorbis)
  endif()
elseif(CMAKE_CXX_COMPILER_ID STREQUAL "Clang")
  if(CMAKE_CXX_COMPILER_VERSION VERSION_LESS "9.0")
    target_link_libraries(wwtools ogg vorbis c++fs)
  else()
    target_link_libraries(wwtools ogg vorbis)
  endif()
else()
  target_link_libraries(wwtools ogg vorbis)
endif()

# To install to the default location, destination directive is unnecessary
# https://cmake.org/cmake/help/latest/command/install.html#installing-targets
install(TARGETS wwtools)



option(DOWNLOAD_CATCH2 "Download Catch2 framework if it won't be found locally" ON)

find_package(Catch2 3)

if(NOT Catch2_FOUND AND DOWNLOAD_CATCH2)
  include(FetchContent)

  FetchContent_Declare(
    catch2
    GIT_REPOSITORY https://github.com/catchorg/Catch2
    GIT_TAG v3.0.0-preview5
  )
  
  message("Catch2 v3 not found locally. Fetching it from the git repository...")
  FetchContent_MakeAvailable(catch2)
  set(FETCHCONTENT_UPDATES_DISCONNECTED_CATCH2 ON) # don't update

  set(Catch2_FOUND TRUE)
endif()

if(Catch2_FOUND)
  add_executable(tests ${SOURCE} tests/test.cpp)
  target_link_libraries(tests PRIVATE Catch2::Catch2WithMain ogg vorbis)
  file(COPY ${CMAKE_CURRENT_SOURCE_DIR}/tests/testdata DESTINATION ${CMAKE_RUNTIME_OUTPUT_DIRECTORY})
else()
  message("Catch2 v3 was not found and not dowloaded. Testing target won't be available.")
endif()
