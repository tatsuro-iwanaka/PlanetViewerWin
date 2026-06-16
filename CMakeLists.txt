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
    set(CMAKE_MSVC_RUNTIME_LIBRARY "MultiThreaded")
endif()

add_executable(PlanetViewer ${SRCS} ${WIN_RESOURCES})

# 依存パッケージの探索
find_package(glfw3 REQUIRED)
find_package(OpenCV REQUIRED)
target_link_libraries(PlanetViewer PRIVATE glfw ${OpenCV_LIBS})

# CSPICE の探索（環境別にロジックを完全隔離）
if(WIN32)
    # vcpkg環境：サブフォルダ(PATH_SUFFIXES)および小文字名の可能性を網羅して自動探索
    find_path(CSPICE_INCLUDE_DIR 
        NAMES SpiceUsr.h spiceusr.h 
        PATH_SUFFIXES cspice
    )
    find_library(CSPICE_LIBRARY 
        NAMES cspice libcspice
    )
else()
    # Mac環境：Homebrewの固定パスを走査
    find_path(CSPICE_INCLUDE_DIR NAMES SpiceUsr.h PATHS "/opt/homebrew/Cellar/cspice/67/include")
    find_library(CSPICE_LIBRARY NAMES cspice libcspice.a PATHS "/opt/homebrew/Cellar/cspice/67/lib")
endif()

# 検知成否のバリデーション
if(NOT CSPICE_INCLUDE_DIR OR NOT CSPICE_LIBRARY)
    message(FATAL_ERROR "CSPICE infrastructure missing. INCLUDE: ${CSPICE_INCLUDE_DIR}, LIB: ${CSPICE_LIBRARY}")
endif()

target_include_directories(PlanetViewer PRIVATE ${CSPICE_INCLUDE_DIR})
target_link_libraries(PlanetViewer PRIVATE ${CSPICE_LIBRARY})

target_compile_definitions(PlanetViewer PRIVATE GL_SILENCE_DEPRECATION)

if(WIN32)
    target_link_options(PlanetViewer PRIVATE "-static")
endif()

if(APPLE)
    target_link_libraries(PlanetViewer PRIVATE "-framework OpenGL" "-framework Cocoa" "-framework IOKit" "-framework CoreVideo")
endif()
