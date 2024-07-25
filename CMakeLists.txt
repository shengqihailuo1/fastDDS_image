cmake_minimum_required(VERSION 3.20)

project(my_hk_FastDDS)

if(NOT fastcdr_FOUND)
    find_package(fastcdr 2 REQUIRED)
endif()
if(NOT fastrtps_FOUND)
    find_package(fastrtps 2.12 REQUIRED)
endif()
find_package(PkgConfig REQUIRED)
pkg_check_modules(YAML REQUIRED yaml-cpp)

find_package(OpenCV 4.2.0 REQUIRED)


#用于告知编译器去哪里寻找需要包含的头文件。
include_directories(
  ${YAML_INCLUDE_DIRS}
  ${OpenCV_INCLUDE_DIRS}   #OpenCV 库的头文件路径
  include                  #项目中的一个本地 include 目录
  /opt/MVS/include/        #安装在系统里面的海康SDK的头文件
)

#指定了编译器在链接阶段应该查找库文件的路径
LINK_DIRECTORIES(
  ${YAML_LIBRARY_DIRS}
  /opt/MVS/lib/64/
)

include(CheckCXXCompilerFlag)
if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_COMPILER_IS_CLANG OR
        CMAKE_CXX_COMPILER_ID MATCHES "Clang")
    check_cxx_compiler_flag(-std=c++11 SUPPORTS_CXX11)
    if(SUPPORTS_CXX11)
        add_compile_options(-std=c++11)
    else()
        message(FATAL_ERROR "Compiler doesn't support C++11")
    endif()
endif()

message(STATUS "Configuring HelloWorld publisher/subscriber example...")
file(GLOB DDS_HK_IMAGE_SOURCES_CXX "src/*.cxx")

add_executable(my_hk_FastDDS_Publisher src/hk_image_Publisher.cpp ${DDS_HK_IMAGE_SOURCES_CXX})
# target_link_libraries(my_hk_FastDDS fastdds fastrtps fastcdr)
target_link_libraries(my_hk_FastDDS_Publisher
  fastrtps
  fastcdr
  ${YAML_LIBRARIES}
  #${OpenCV_LIBRARIES}
  #加载下面海康相机的.so文件并调用其API接口，实现对海康相机的控制和图像加载功能。
  MvCameraControl
  MVGigEVisionSDK
  MVRender
  MvUsb3vTL
  MediaProcess
  X11
)


add_executable(my_hk_FastDDS_Subscriber src/hk_image_Subscriber.cpp ${DDS_HK_IMAGE_SOURCES_CXX})
target_link_libraries(my_hk_FastDDS_Subscriber fastrtps fastcdr ${OpenCV_LIBRARIES})
