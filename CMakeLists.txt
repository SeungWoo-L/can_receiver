### SETUP ###
cmake_minimum_required(VERSION 3.15)
project(Can_Receiver VERSION 0.1 LANGUAGES CXX)

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -pthread -std=c++11")
set(CMAKE_INCLUDE_CURRENT_DIR ON)
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

### PACKAGES ###
find_package(CommonAPI REQUIRED)
find_package(CommonAPI-SomeIP REQUIRED)

#### VARIABLES ####
set(Can_Receiver_SOURCES
src/main.cpp
src/SpeedSensorStubImpl.cpp
src/CanDataRegister.cpp
src/MovingAverageFilter.cpp
src/CanReceiver.cpp
)

set(INSTALL_LIB_DIR lib CACHE PATH "Installation directory for libraries")
set(INSTALL_BIN_DIR bin CACHE PATH "Installation directory for binaries")
set(INSTALL_INCLUDE_DIR include/CanReceiver CACHE PATH "Installation directory for header files")
set(DEF_INSTALL_CMAKE_DIR lib/cmake/CanReceiver-someip)
set(INSTALL_CMAKE_DIR ${DEF_INSTALL_CMAKE_DIR} CACHE PATH "Installation directory for CMake files")



FILE(GLOB PRJ_SOMEIP_LIB_SRCS ./src-gen/someip/v0/commonapi/*.cpp)

### DIRECTORIES ###
include_directories(
	./include
    ./src-gen/core
    ./src-gen/someip
    ${COMMONAPI_INCLUDE_DIRS}
    ${COMMONAPI_SOMEIP_INCLUDE_DIRS}
)

link_directories(
    ${COMMONAPI_LIBDIR}
    ${COMMONAPI_SOMEIP_CMAKE_DIR}/build
)
#### BUILD ####
# Service
add_executable(CanReceiver 
    ${Can_Receiver_SOURCES}
)
target_link_libraries(CanReceiver CommonAPI)

add_library(CanReceiver-someip
    SHARED ${PRJ_SOMEIP_LIB_SRCS}
)
target_link_libraries(CanReceiver-someip
    CommonAPI
    CommonAPI-SomeIP
)

#### INSTALLATION RULES ####

# Install the CanReceiver executable to the bin directory
install(TARGETS CanReceiver
    RUNTIME DESTINATION ${INSTALL_BIN_DIR}
)

# Install the CanReceiver-someip library to the lib directory
install(TARGETS CanReceiver-someip
    LIBRARY DESTINATION ${INSTALL_LIB_DIR}
)

# Install header files
install(DIRECTORY include/
    DESTINATION ${INSTALL_INCLUDE_DIR}
)

# Install CMake project configuration files to make the project importable
install(EXPORT CanReceiverTargets
    FILE CanReceiverTargets.cmake
    DESTINATION ${INSTALL_CMAKE_DIR}
)

#### EXPORTING TARGETS ####

# Create and install the Config file
include(CMakePackageConfigHelpers)
write_basic_package_version_file(
    CanReceiverConfigVersion.cmake
    VERSION ${PROJECT_VERSION}
    COMPATIBILITY SameMajorVersion
)

install(FILES CanReceiverConfig.cmake ${CMAKE_CURRENT_BINARY_DIR}/CanReceiverConfigVersion.cmake
    DESTINATION ${INSTALL_CMAKE_DIR}
)

export(EXPORT CanReceiverTargets
    FILE ${CMAKE_CURRENT_BINARY_DIR}/CanReceiverTargets.cmake
)