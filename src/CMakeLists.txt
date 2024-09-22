cmake_minimum_required(VERSION 3.10)
project(biosim4)

# Set C++ standard
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# Build type (default is release)
if(NOT CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE Release)
endif()

set(CMAKE_CXX_COMPILER c++)
set(CMAKE_LINKER c++)

set(CMAKE_RUNTIME_OUTPUT_DIRECTORY_DEBUG ${CMAKE_BINARY_DIR}/bin/Debug)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY_RELEASE ${CMAKE_BINARY_DIR}/bin/Release)

set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY_DEBUG ${CMAKE_BINARY_DIR}/obj/Debug/src)
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY_RELEASE ${CMAKE_BINARY_DIR}/obj/Release/src)

file(GLOB_RECURSE SOURCES
  src/*.cpp
  src/userio/*.cpp
  src/userio/sfmlComponents/*.cpp
  src/userio/sfmlComponents/flowComponents/*.cpp
  src/userio/sfmlComponents/settingsComponents/*.cpp
  src/utils/*.cpp
  src/ai/*.cpp
  src/survivalCriteria/*.cpp
)

file(GLOB_RECURSE HEADERS
  src/*.h
  src/userio/*.h
  src/userio/sfmlComponents/*.h
  src/userio/sfmlComponents/flowComponents/*.h
  src/userio/sfmlComponents/settingsComponents/*.h
  src/utils/*.h
  src/ai/*.h
  src/survivalCriteria/*.h
)

include_directories(src/include)

find_package(OpenMP REQUIRED)
if(OpenMP_CXX_FOUND)
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${OpenMP_CXX_FLAGS}")
endif()

find_package(OpenCV REQUIRED)
find_package(SFML 2.6 COMPONENTS graphics window system REQUIRED)
find_package(TGUI REQUIRED)

add_executable(biosim4 ${SOURCES} ${HEADERS})

target_compile_options(biosim4 PRIVATE
  -Wall
  -pedantic
  -fexceptions
  -fopenmp
  $<$<CONFIG:Debug>:-g>
  $<$<CONFIG:Release>:-O3>
) 

target_link_libraries(biosim4
  OpenMP::OpenMP_CXX
  ${OpenCV_LIBS}
  sfml-graphics
  sfml-window
  sfml-system
  tgui
  -lgomp
  -lpthread
  $<$<CONFIG:Release>:-O3 -s>
)

add_custom_target(distclean
  COMMAND ${CMAKE_COMMAND} -E remove_directory ${CMAKE_BINARY_DIR}/Output/Images
  COMMAND ${CMAKE_COMMAND} -E remove_directory ${CMAKE_BINARY_DIR}/Output/Videos
  COMMAND ${CMAKE_COMMAND} -E remove_directory ${CMAKE_BINARY_DIR}/Output/Logs
  COMMAND ${CMAKE_COMMAND} -E remove_directory ${CMAKE_BINARY_DIR}/Output/Saves
  COMMAND ${CMAKE_COMMAND} -E remove_directory ${CMAKE_BINARY_DIR}/Output/Profiling
)

install(FILES ./biosim4.ini DESTINATION ./)
install(DIRECTORY ./tools/ DESTINATION tools)
install (DIRECTORY DESTINATION "logs" DIRECTORY_PERMISSIONS 
    OWNER_WRITE OWNER_READ OWNER_EXECUTE
    GROUP_READ GROUP_READ GROUP_EXECUTE
    GROUP_READ WORLD_READ WORLD_EXECUTE)
install (DIRECTORY DESTINATION "images" DIRECTORY_PERMISSIONS 
    OWNER_WRITE OWNER_READ OWNER_EXECUTE
    GROUP_READ GROUP_READ GROUP_EXECUTE
    GROUP_READ WORLD_READ WORLD_EXECUTE)

set(CPACK_PROJECT_NAME ${PROJECT_NAME})
set(CPACK_PROJECT_VERSION ${PROJECT_VERSION})
include(CPack)