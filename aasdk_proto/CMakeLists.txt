# Newer Protobuf starts requiring Abseil, and the old CMake's FindProtobuf
# module is not updated to reflect that. Thus, prefer upstream's config module
# if available.
set(CMAKE_FIND_PACKAGE_PREFER_CONFIG ON)
# protobuf_MODULE_COMPATIBLE enables FindProtobuf-compatible protoc generation
# CMake function from Protobuf's CMake config file. However, in Protobuf's
# infinite wisdom, they made this variable a CACHE variable which behaves
# incorrectly when CMakeLists.txt file sets it. CMP0077 fixes the behavior.
cmake_policy(SET CMP0077 NEW)
set(protobuf_MODULE_COMPATIBLE ON)
find_package(Protobuf REQUIRED)
unset(CMAKE_FIND_PACKAGE_PREFER_CONFIG)

# https://github.com/protocolbuffers/protobuf/issues/14576#issuecomment-2059422713
if(CMAKE_CROSSCOMPILING)
    find_program(_PROTOBUF_PROTOC_EXECUTABLE protoc)
    if(NOT TARGET protobuf::protoc)
        add_executable(protobuf::protoc IMPORTED)
    endif()
    set_target_properties(protobuf::protoc PROPERTIES
        IMPORTED_LOCATION_RELEASE "${_PROTOBUF_PROTOC_EXECUTABLE}"
        IMPORTED_LOCATION_DEBUG "${_PROTOBUF_PROTOC_EXECUTABLE}"
        IMPORTED_LOCATION_RELWIHDEBINFO "${_PROTOBUF_PROTOC_EXECUTABLE}"
        IMPORTED_LOCATION_NOCONFIG "${_PROTOBUF_PROTOC_EXECUTABLE}"
    )
endif()

file(GLOB_RECURSE proto_files ${CMAKE_CURRENT_SOURCE_DIR}/*.proto)
protobuf_generate_cpp(proto_sources proto_headers ${proto_files})
add_library(aasdk_proto SHARED ${proto_headers} ${proto_sources})
target_link_libraries(aasdk_proto protobuf::libprotobuf)

set(aasdk_VERSION_STRING ${aasdk_VERSION_MAJOR}.${aasdk_VERSION_MINOR}.${aasdk_VERSION_PATCH})
set_target_properties(aasdk_proto PROPERTIES VERSION ${aasdk_VERSION_STRING}
                                          SOVERSION ${aasdk_VERSION_MAJOR})

install(TARGETS aasdk_proto DESTINATION lib)
install(DIRECTORY ${CMAKE_CURRENT_BINARY_DIR} DESTINATION include
    FILES_MATCHING PATTERN *.h
    PATTERN CMakeFiles EXCLUDE )
