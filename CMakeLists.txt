project(bluetoothserialport)
cmake_minimum_required(VERSION 3.1)
set(CMAKE_CXX_STANDARD 11)

add_library(bluetoothserialport STATIC)

target_sources(bluetoothserialport
    PRIVATE
    ${CMAKE_CURRENT_SOURCE_DIR}/src/Enums.cc
)

if(WIN32) # windows
    set(PLATFORM windows)
    target_sources(bluetoothserialport
        PRIVATE
        ${CMAKE_CURRENT_SOURCE_DIR}/src/windows/BluetoothHelpers.cc
        ${CMAKE_CURRENT_SOURCE_DIR}/src/windows/BTSerialPortBinding.cc
        ${CMAKE_CURRENT_SOURCE_DIR}/src/windows/DeviceINQ.cc
    )
    target_link_libraries(bluetoothserialport ws2_32 bthprops)

elseif(APPLE) # MacOSX
    set(PLATFORM osx)
    target_sources(bluetoothserialport
        PRIVATE
        ${CMAKE_CURRENT_SOURCE_DIR}/src/osx/BluetoothDeviceResources.mm
        ${CMAKE_CURRENT_SOURCE_DIR}/src/osx/BluetoothWorker.mm
        ${CMAKE_CURRENT_SOURCE_DIR}/src/osx/BTSerialPortBinding.mm
        ${CMAKE_CURRENT_SOURCE_DIR}/src/osx/DeviceINQ.mm
        ${CMAKE_CURRENT_SOURCE_DIR}/src/osx/pipe.c
    )
    find_library(FOUNDATION Foundation)
    find_library(IOBLUETOOTH IOBluetooth)
    target_link_libraries(bluetoothserialport ${FOUNDATION} ${IOBLUETOOTH} -fobjc-arc)

    add_executable(btScan src/osx/btScan.mm)
    target_link_libraries(btScan ${FOUNDATION} ${IOBLUETOOTH})
else() # Linux
    set(PLATFORM linux)
    target_sources(bluetoothserialport
        PRIVATE
        ${CMAKE_CURRENT_SOURCE_DIR}/src/linux/BTSerialPortBinding.cc
        ${CMAKE_CURRENT_SOURCE_DIR}/src/linux/DeviceINQ.cc
    )
    target_link_libraries(bluetoothserialport bluetooth)
endif()

# Platform-specific compile flags
if(${CMAKE_CXX_COMPILER_ID} STREQUAL "GNU") # G++
    target_compile_options(bluetoothserialport PRIVATE -Wall -Wextra)
elseif(${CMAKE_CXX_COMPILER_ID} STREQUAL "MSVC") # MSVC
    target_compile_options(bluetoothserialport PRIVATE /EHsc /W2 /c)
endif()

target_include_directories(bluetoothserialport PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/src>
    $<INSTALL_INTERFACE:include>
    PRIVATE
    ${CMAKE_CURRENT_SOURCE_DIR}/src/${PLATFORM}
)

# Test
add_executable(inquiretest EXCLUDE_FROM_ALL ${CMAKE_CURRENT_SOURCE_DIR}/test/main.cpp)

set_target_properties(bluetoothserialport
    PROPERTIES
    POSITION_INDEPENDENT_CODE ON
)

target_link_libraries(inquiretest bluetoothserialport ${FOUNDATION} ${IOBLUETOOTH})

# Install
set_target_properties(bluetoothserialport
    PROPERTIES
    ARCHIVE_OUTPUT_DIRECTORY ${BINARY_DIR}/lib
    LIBRARY_OUTPUT_DIRECTORY ${BINARY_DIR}/lib
    RUNTIME_OUTPUT_DIRECTORY ${BINARY_DIR}
    VERSION ${PROJECT_VERSION}
    DEBUG_POSTFIX "d"
)

install(TARGETS bluetoothserialport EXPORT bluetoothserialportConfig
    ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR})

install(FILES ${CMAKE_CURRENT_SOURCE_DIR}/src/BluetoothException.h DESTINATION ${CMAKE_INSTALL_PREFIX}/include)
install(FILES ${CMAKE_CURRENT_SOURCE_DIR}/src/BTSerialPortBinding.h DESTINATION ${CMAKE_INSTALL_PREFIX}/include)
install(FILES ${CMAKE_CURRENT_SOURCE_DIR}/src/DeviceINQ.h DESTINATION ${CMAKE_INSTALL_PREFIX}/include)
install(FILES ${CMAKE_CURRENT_SOURCE_DIR}/src/Enums.h DESTINATION ${CMAKE_INSTALL_PREFIX}/include)
install(EXPORT bluetoothserialportConfig DESTINATION share/bluetoothserialportConfig/cmake)
export(TARGETS bluetoothserialport FILE bluetoothserialportConfig.cmake)
