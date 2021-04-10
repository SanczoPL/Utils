cmake_minimum_required(VERSION 3.9 FATAL_ERROR)
project(Utils VERSION 0.0.1 DESCRIPTION "Utils" LANGUAGES C CXX)

set(TARGET_NAME "Utils")
set(DEFAULT_BUILD_TYPE "Debug")
set(CMAKE_BUILD_TYPE "Debug")

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${FLAGS_COMMON}")

set(CMAKE_AUTOMOC ON)

file(MAKE_DIRECTORY "${CMAKE_BINARY_DIR}/bin")
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/bin")

macro(copy_json jsonname)
	configure_file(${jsonname} ${CMAKE_BINARY_DIR}/${jsonname})
	configure_file(${jsonname} ${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/bin/${jsonname})
	configure_file(${jsonname} ${CMAKE_BINARY_DIR}/bin/${jsonname})
	configure_file(${jsonname} ${CMAKE_BINARY_DIR}/bin/${CMAKE_BUILD_TYPE}/${jsonname})
endmacro()
copy_json("utils.json")

if (WIN32)
	list(APPEND CMAKE_PREFIX_PATH "E:\\Qt\\5.12.8\\msvc2017_64")
endif (WIN32)


macro(clone_build libname)
	configure_file(clone_build/${libname}.in ${libname}-download/CMakeLists.txt)
	execute_process(COMMAND "${CMAKE_COMMAND}" -G "${CMAKE_GENERATOR}" .
    WORKING_DIRECTORY "${CMAKE_BINARY_DIR}/${libname}-download")
	execute_process(COMMAND "${CMAKE_COMMAND}" --build .
	WORKING_DIRECTORY "${CMAKE_BINARY_DIR}/${libname}-download")
	add_subdirectory("${CMAKE_BINARY_DIR}/${libname}-src"
                 "${CMAKE_BINARY_DIR}/${libname}-build")
endmacro()

clone_build(googletest)
# Prevent GoogleTest from overriding our compiler/linker options
# when building with Visual Studio
set(gtest_force_shared_crt ON CACHE BOOL "" FORCE)
clone_build(spdlog)

#find_package(gtest)
#find_package(gmock)
find_package(Qt5Test REQUIRED)
find_package(Qt5Core REQUIRED)
find_package(Threads REQUIRED)

if(NOT CMAKE_BUILD_TYPE AND NOT CMAKE_CONFIGURATION_TYPES)
    message(STATUS "Setting build type to '${DEFAULT_BUILD_TYPE}' as none was specified")
    set(CMAKE_BUILD_TYPE ${DEFAULT_BUILD_TYPE} CACHE STRING "Choose the type of build." FORCE)
    set_property(CACHE CMAKE_BUILD_TYPE PROPERTY STRINGS "Debug" "Release" "MinSizeRel" "RelWithDebInfo")
endif()

set(SRC src/configreader.cc src/includespdlog.cc)
set(HEADERS include/configreader.h include/configreader.h)

#file(GLOB_RECURSE SRC_GLOB src/*.h src/*.hpp)
#file(GLOB_RECURSE HEADERS_GLOB include/*.h include/*.hpp)


add_library(${PROJECT_NAME} STATIC ${SRC} ${HEADERS})

add_executable(g_test utils.json gtest/gtest.cc)

add_executable(q_test utils.json qtest/qtest.cc qtest/qtest.h)


target_include_directories(${PROJECT_NAME} PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
    $<INSTALL_INTERFACE:include>
    PRIVATE src)

target_link_libraries(${PROJECT_NAME} Qt5::Core Threads::Threads)

target_link_libraries(g_test ${PROJECT_NAME} gtest gmock)
target_link_libraries(g_test ${PROJECT_NAME} Qt5::Core Threads::Threads)

target_link_libraries(q_test ${PROJECT_NAME})
target_link_libraries(q_test Qt5::Test Qt5::Core Threads::Threads) 

if (CMAKE_INSTALL_PREFIX_INITIALIZED_TO_DEFAULT)
    set (CMAKE_INSTALL_PREFIX "/usr" CACHE PATH "default install path" FORCE )
endif()

install(TARGETS ${PROJECT_NAME} EXPORT ${PROJECT_NAME}Config
    ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
)

install(TARGETS ${PROJECT_NAME} DESTINATION lib)
install(FILES  ${HEADERS} DESTINATION include)
install(EXPORT ${PROJECT_NAME}Config DESTINATION share/${PROJECT_NAME}/cmake)

export(TARGETS ${PROJECT_NAME} FILE ${PROJECT_NAME}Config.cmake)