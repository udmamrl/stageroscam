cmake_minimum_required(VERSION 2.4.6)
include($ENV{ROS_ROOT}/core/rosbuild/rosbuild.cmake)
include(FindPkgConfig)
rosbuild_init()

rosbuild_make_distribution(1.6.6)
rosbuild_add_boost_directories()

# Build Stage
execute_process(COMMAND cmake -E chdir ${PROJECT_SOURCE_DIR} make -f Makefile.stage
                RESULT_VARIABLE _make_failed)
if(_make_failed)
  message(FATAL_ERROR "Build of Stage failed")
endif(_make_failed)

# Find the build flags for Stage
set(ENV{PKG_CONFIG_PATH} "${PROJECT_SOURCE_DIR}/lib64/pkgconfig:${PROJECT_SOURCE_DIR}/lib/pkgconfig:$ENV{PKG_CONFIG_PATH}")
pkg_check_modules(STAGE REQUIRED stage)

# Adding explicit stage include and link directories since pkgconfig may not be correct
# when building for a deb install.
include_directories(${STAGE_INCLUDE_DIRS} ${PROJECT_SOURCE_DIR}/include/Stage-4.1)
link_directories(${STAGE_LIBRARY_DIRS} ${PROJECT_SOURCE_DIR}/lib ${PROJECT_SOURCE_DIR}/lib64)

rosbuild_add_executable(bin/stageroscam src/stageros.cpp)

rosbuild_link_boost(bin/stageroscam thread)
add_dependencies(bin/stageroscam build_stage)

rosbuild_add_compile_flags(bin/stageroscam ${STAGE_CFLAGS_OTHERS})
target_link_libraries(bin/stageroscam ${STAGE_LIBRARIES})
rosbuild_add_link_flags(bin/stageroscam ${STAGE_LDFLAGS_OTHERS})


rosbuild_add_rostest(test/hztest.xml)
