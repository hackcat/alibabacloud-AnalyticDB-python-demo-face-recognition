#

cmake_minimum_required(VERSION 2.8)

project(SeetaFaceTracker)

if(UNIX)
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -fPIC")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11 -fPIC")
endif()

add_definitions(-DSEETA_EXPORTS)

file(GLOB_RECURSE ROOT_SRC_FILES
        ${CMAKE_CURRENT_SOURCE_DIR}/src/*.c
        ${CMAKE_CURRENT_SOURCE_DIR}/src/*.cc
        ${CMAKE_CURRENT_SOURCE_DIR}/src/*.cpp
        )

file(GLOB_RECURSE ROOT_SEETA_FILES
        ${CMAKE_CURRENT_SOURCE_DIR}/seeta/*.c
        ${CMAKE_CURRENT_SOURCE_DIR}/seeta/*.cc
        ${CMAKE_CURRENT_SOURCE_DIR}/seeta/*.cpp
        )

file(GLOB INCLUDE_FILES
        ${CMAKE_CURRENT_SOURCE_DIR}/include/seeta/*.h
        ${CMAKE_CURRENT_SOURCE_DIR}/include/seeta/*.hh
        ${CMAKE_CURRENT_SOURCE_DIR}/include/seeta/*.hpp
        )

list(APPEND SRC_FILES ${ROOT_SRC_FILES})
list(APPEND SRC_FILES ${ROOT_SEETA_FILES})

# add library
add_library(${PROJECT_NAME} ${SRC_FILES})
target_include_directories(${PROJECT_NAME} PRIVATE
        ${CMAKE_CURRENT_SOURCE_DIR}
        ${PROJECT_SOURCE_DIR}/include
        ${PROJECT_SOURCE_DIR}/include/seeta
        ${PROJECT_SOURCE_DIR}/seeta
        )
target_include_directories(${PROJECT_NAME} PRIVATE
        ${CMAKE_SOURCE_DIR}/SeetaNet/include
        ${CMAKE_SOURCE_DIR}/FaceDetector/include
        )
target_link_libraries(${PROJECT_NAME} PUBLIC SeetaNet)
target_link_libraries(${PROJECT_NAME} PUBLIC SeetaFaceDetector)

GENERATE_EXPORT_HEADER(${PROJECT_NAME}
        EXPORT_MACRO_NAME SEETA_FACE_TRACKER_API
        EXPORT_FILE_NAME  SeetaFaceTrackerExport.h
        )
file(COPY ${CMAKE_CURRENT_BINARY_DIR}/SeetaFaceTrackerExport.h
        DESTINATION ${CMAKE_BINARY_DIR})
install(
        FILES
        ${CMAKE_CURRENT_BINARY_DIR}/SeetaFaceTrackerExport.h
        DESTINATION
        ${CMAKE_INSTALL_INCLUDEDIR}/seeta
)
target_include_directories(${PROJECT_NAME} PRIVATE
        ${CMAKE_BINARY_DIR}
        )

set_target_properties(${PROJECT_NAME} PROPERTIES RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)
set_target_properties(${PROJECT_NAME} PROPERTIES LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)
set_target_properties(${PROJECT_NAME} PROPERTIES ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)

set_target_properties(${PROJECT_NAME} PROPERTIES VERSION ${BUILD_VERSION})
# Be will to install header files
set_property(
        TARGET
        ${PROJECT_NAME}
        PROPERTY
        PUBLIC_HEADER ${INCLUDE_FILES}
)

INSTALL(TARGETS ${PROJECT_NAME}
        EXPORT ${PROJECT_NAME}Config
        RUNTIME DESTINATION "${CMAKE_INSTALL_BINDIR}"
        ARCHIVE DESTINATION "${CMAKE_INSTALL_LIBDIR}"
        LIBRARY DESTINATION "${CMAKE_INSTALL_BINDIR}"
        PUBLIC_HEADER DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/seeta
        INCLUDES DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
        )
export(TARGETS ${PROJECT_NAME}
        APPEND FILE ${CMAKE_BINARY_DIR}/${PROJECT_NAME}Config.cmake
        )
# Install cmake configure files
install(
        EXPORT ${PROJECT_NAME}Config
        DESTINATION "${CMAKE_INSTALL_LIBDIR}/cmake"
        NAMESPACE
        SeetaFace::
)
write_basic_package_version_file(
        "${CMAKE_BINARY_DIR}/${PROJECT_NAME}ConfigVersion.cmake"
        VERSION ${BUILD_VERSION}
        COMPATIBILITY AnyNewerVersion)
install(FILES "${CMAKE_BINARY_DIR}/${PROJECT_NAME}ConfigVersion.cmake" DESTINATION "${CMAKE_INSTALL_LIBDIR}/cmake")
