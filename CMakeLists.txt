cmake_minimum_required(VERSION 3.22)
project(PlanetViewer LANGUAGES CXX)

# C++20 標準規格の強制
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# ソースファイル群の定義（Makefileの内容を移植）
set(IMGUI_SRCS
    imgui/imgui.cpp
    imgui/imgui_draw.cpp
    imgui/imgui_widgets.cpp
    imgui/imgui_tables.cpp
    imgui/backends/imgui_impl_glfw.cpp
    imgui/backends/imgui_impl_opengl3.cpp
)
set(SRCS main.cpp ${IMGUI_SRCS})

# Windows環境固有のビルド設定 (リソース結合 + ランタイム静的化)
if(WIN32)
    set(WIN_RESOURCES resource.rc)
    # /MT フラグを強制し、VC++再頒布可能パッケージ(.dll)の依存をカット
    set(CMAKE_MSVC_RUNTIME_LIBRARY "MultiThreaded")
endif()

# 実行ファイルの生成定義
add_executable(PlanetViewer ${SRCS} ${WIN_RESOURCES})

# 依存外部パッケージの探索
find_package(glfw3 REQUIRED)
find_package(OpenCV REQUIRED)
target_link_libraries(PlanetViewer PRIVATE glfw ${OpenCV_LIBS})

# C-SPICE の探索ロジック（Windowsのvcpkg環境とMacのHomebrew環境への双方適合）
find_package(cspice CONFIG QUIET)
if(cspice_FOUND)
    target_link_libraries(PlanetViewer PRIVATE cspice::cspice)
else()
    # Macローカル（Homebrew）環境用のフォールバックパス設定
    find_path(CSPICE_INCLUDE_DIR NAMES SpiceUsr.h PATHS "/opt/homebrew/Cellar/cspice/67/include")
    find_library(CSPICE_LIBRARY NAMES cspice libcspice.a PATHS "/opt/homebrew/Cellar/cspice/67/lib")
    target_include_directories(PlanetViewer PRIVATE ${CSPICE_INCLUDE_DIR})
    target_link_libraries(PlanetViewer PRIVATE ${CSPICE_LIBRARY})
endif()

# 共通のマクロ定義
target_compile_definitions(PlanetViewer PRIVATE GL_SILENCE_DEPRECATION)

# Mac固有のシステムフレームワーク結合
if(APPLE)
    target_link_libraries(PlanetViewer PRIVATE "-framework OpenGL" "-framework Cocoa" "-framework IOKit" "-framework CoreVideo")
endif()