cmake_minimum_required(VERSION 3.22)
project(PlanetViewer LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# ソースファイル群の定義
set(IMGUI_SRCS
    imgui/imgui.cpp
    imgui/imgui_draw.cpp
    imgui/imgui_widgets.cpp
    imgui/imgui_tables.cpp
    imgui/backends/imgui_impl_glfw.cpp
    imgui/backends/imgui_impl_opengl3.cpp
)
set(SRCS main.cpp ${IMGUI_SRCS})

if(WIN32)
    set(WIN_RESOURCES resource.rc)
    # 完全に独立したスタティックバイナリを生成するフラグ（/MT）
    set(CMAKE_MSVC_RUNTIME_LIBRARY "MultiThreaded")
endif()

add_executable(PlanetViewer ${SRCS} ${WIN_RESOURCES})

# 依存パッケージの探索
find_package(glfw3 REQUIRED)
find_package(OpenCV REQUIRED)
target_link_libraries(PlanetViewer PRIVATE glfw ${OpenCV_LIBS})

# CSPICE の探索
if(WIN32)
    find_path(CSPICE_INCLUDE_DIR 
        NAMES SpiceUsr.h spiceusr.h 
        PATH_SUFFIXES cspice
    )
    find_library(CSPICE_LIBRARY 
        NAMES cspice libcspice
    )
else()
    find_path(CSPICE_INCLUDE_DIR NAMES SpiceUsr.h PATHS "/opt/homebrew/Cellar/cspice/67/include")
    find_library(CSPICE_LIBRARY NAMES cspice libcspice.a PATHS "/opt/homebrew/Cellar/cspice/67/lib")
endif()

if(NOT CSPICE_INCLUDE_DIR OR NOT CSPICE_LIBRARY)
    message(FATAL_ERROR "CSPICE infrastructure missing. INCLUDE: ${CSPICE_INCLUDE_DIR}, LIB: ${CSPICE_LIBRARY}")
endif()

# インクルードパスの設定
target_include_directories(PlanetViewer PRIVATE 
    ${CMAKE_CURRENT_SOURCE_DIR}
    ${CMAKE_CURRENT_SOURCE_DIR}/imgui
    ${CMAKE_CURRENT_SOURCE_DIR}/imgui/backends
    ${CSPICE_INCLUDE_DIR}
)

target_link_libraries(PlanetViewer PRIVATE ${CSPICE_LIBRARY})

# Windows特有のmin/maxマクロ汚染の防止
target_compile_definitions(PlanetViewer PRIVATE GL_SILENCE_DEPRECATION NOMINMAX)

# OS別のシステムグラフィックスライブラリ結合
if(WIN32)
    # 【修正】Windows標準のOpenGLライブラリを明示的に結合
    target_link_libraries(PlanetViewer PRIVATE opengl32)
endif()

if(APPLE)
    target_link_libraries(PlanetViewer PRIVATE "-framework OpenGL" "-framework Cocoa" "-framework IOKit" "-framework CoreVideo")
endif()
