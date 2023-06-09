# PytgVoIP - Telegram VoIP Library for Python
# Copyright (C) 2020 bakatrouble <https://github.com/bakatrouble>
#
# This file is part of PytgVoIP.
#
# PytgVoIP is free software: you can redistribute it and/or modify
# it under the terms of the GNU Lesser General Public License as published
# by the Free Software Foundation, either version 3 of the License, or
# (at your option) any later version.
#
# PytgVoIP is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU Lesser General Public License for more details.
#
# You should have received a copy of the GNU Lesser General Public License
# along with PytgVoIP.  If not, see <http://www.gnu.org/licenses/>.


cmake_minimum_required(VERSION 3.13)
project(pylibtgvoip)
set(CMAKE_CXX_STANDARD 14)
cmake_policy(SET CMP0028 NEW)

list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/src")
if(UNIX AND NOT APPLE)
    set(LINUX TRUE)
endif()
if(APPLE)
    set(OPENSSL_ROOT_DIR /usr/local/opt/openssl/)
endif()
add_definitions(
        -DTGVOIP_USE_CALLBACK_AUDIO_IO
        -DWITHOUT_ALSA
        -DWITHOUT_PULSE
        -DTGVOIP_NO_DSP
        -DDESKTOP_APP_DISABLE_WEBRTC_INTEGRATION
)

add_subdirectory(3rdparty/pybind11)
include_directories(${TGVOIP_INCLUDE_DIR})

if(WIN32)
    get_filename_component(libs_loc .. REALPATH)
else()
    set(DESKTOP_APP_USE_PACKAGED TRUE)
endif()
include(cmake/nice_target_sources.cmake)
include(cmake/target_link_frameworks.cmake)
include(cmake/target_link_static_libraries.cmake)
include(cmake/init_target.cmake)
include(cmake/options.cmake)
add_subdirectory(cmake/external/openssl)
add_subdirectory(cmake/external/opus)
get_filename_component(third_party_loc 3rdparty REALPATH)
include(src/libtgvoip.cmake)

nice_target_sources(lib_tgvoip ${tgvoip_loc}
        PRIVATE
        audio/AudioIOCallback.cpp
        audio/AudioIOCallback.h
)

list(APPEND SOURCES
        src/_tgvoip.cpp
        src/_tgvoip_module.cpp
)

pybind11_add_module(_tgvoip ${SOURCES})
target_link_libraries(_tgvoip PRIVATE lib_tgvoip)

if ((CMAKE_CXX_COMPILER_ID STREQUAL GNU) AND NOT (CMAKE_CXX_COMPILER_VERSION VERSION_LESS 9))
    target_compile_options(_tgvoip PRIVATE
            -Wno-error=deprecated-copy
    )
endif()

if (WIN32)
    target_compile_options(lib_tgvoip PRIVATE /wd4477 /wd4267)
endif()
# macOS and its weird paths
target_include_directories(lib_tgvoip PRIVATE /usr/local/include)
target_link_directories(_tgvoip PRIVATE /usr/local/lib)
