cmake_minimum_required(VERSION 2.8)

project(StreamExecutor)

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11 -std=c++14")
# Pick the C++ standard to compile with.
# Abseil currently supports C++11, C++14, and C++17.
set(CMAKE_CXX_STANDARD 14)


add_definitions(-DPLATFORM_POSIX)
include_directories(third_party/gpus/cuda/extras/CUPTI/include/)
include_directories(third_party/gpus/cuda/include/)

find_package(Protobuf REQUIRED)
find_package(Eigen3 REQUIRED NO_MODULE)
find_package(GTest REQUIRED)

# include_directories(${GTEST_INCLUDE_DIR})
include_directories(${EIGEN3_INCLUDE_DIRS})
include_directories(${Protobuf_INCLUDE_DIRS})

include_directories(${CMAKE_CURRENT_BINARY_DIR})

file(GLOB UtilsProtoFiles ${CMAKE_CURRENT_LIST_DIR}/stream_executor/protos/*.proto)
set(ProtoFiles ${CoreProtoFiles} ${UtilsProtoFiles})
protobuf_generate_cpp(ProtoSources ProtoHeaders ${ProtoFiles})


set(SRC_ROOT_PATH ${CMAKE_CURRENT_LIST_DIR}/stream_executor)
set(SRC_DIR
    ${SRC_ROOT_PATH}
    ${SRC_ROOT_PATH}/gpu
    ${SRC_ROOT_PATH}/cuda
    ${SRC_ROOT_PATH}/host
    ${SRC_ROOT_PATH}/platform
    ${SRC_ROOT_PATH}/platform/default
    ${SRC_ROOT_PATH}/lib
    )

set(INCLUDE_DIR ${CMAKE_CURRENT_LIST_DIR})


include_directories(${INCLUDE_DIR})


# *.c
foreach(file_path ${SRC_DIR})
    file(GLOB file_source_c ${file_path}/*.c ${file_path}/*.h)
    set(SRC_FILES ${SRC_FILES} ${file_source_c})
endforeach()
message(STATUS "[*] searching *.c")

# *.cc
foreach(file_path ${SRC_DIR})
    file(GLOB file_source_cc ${file_path}/*.cc ${file_path}/*.hpp)
    set(SRC_FILES ${SRC_FILES} ${file_source_cc})
endforeach()
message(STATUS "[*] searching *.cc")

# *.cpp
foreach(file_path ${SRC_DIR})
    file(GLOB file_source_cpp ${file_path}/*.cpp)
    set(SRC_FILES ${SRC_FILES} ${file_source_cpp})
endforeach()
message(STATUS "[*] searching *.cpp")

add_subdirectory(./third_party/abseil-cpp)
set(ABSL_LIBRARIES absl::base absl::synchronization
    absl::strings absl::str_format_internal absl::container
    absl::hash absl::hashtablez_sampler Threads::Threads
    absl::flat_hash_map absl::debugging)

add_subdirectory(./third_party/nsync)
set(NSYNC_LIBRARIES -lnsync -lnsync_cpp)


add_executable(main ${CMAKE_CURRENT_LIST_DIR}/main.cc ${SRC_FILES} ${ProtoSources})
target_link_libraries(main -pthread ${Protobuf_LIBRARIES}
    ${ABSL_LIBRARIES} ${GTEST_LIBRARIES} ${GTEST_MAIN_LIBRARIES}
    -ldl -ldouble-conversion ${NSYNC_LIBRARIES})


add_executable(test_main ${CMAKE_CURRENT_LIST_DIR}/test_main.cc ${SRC_FILES} ${ProtoSources})
target_link_libraries(test_main -pthread ${Protobuf_LIBRARIES}
    ${ABSL_LIBRARIES} ${GTEST_LIBRARIES} ${GTEST_MAIN_LIBRARIES}
    -ldl -ldouble-conversion ${NSYNC_LIBRARIES})
