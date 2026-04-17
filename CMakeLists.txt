cmake_minimum_required(VERSION 3.16)
project(RPG_Roguelike LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# --- SFML ---
# Статическая линковка SFML: все его библиотеки встраиваются прямо в .exe,
# чтобы не таскать рядом sfml-graphics-2.dll / sfml-window-2.dll / sfml-system-2.dll.
# Это критично для Windows: иначе .exe падает с 0xC0000135 (DLL not found).
set(BUILD_SHARED_LIBS OFF CACHE BOOL "" FORCE)
set(SFML_USE_STATIC_STD_LIBS OFF CACHE BOOL "" FORCE)

include(FetchContent)
FetchContent_Declare(
    SFML
    GIT_REPOSITORY https://github.com/SFML/SFML.git
    GIT_TAG 2.6.1
)
FetchContent_MakeAvailable(SFML)

# --- Sources ---
set(SOURCES
    src/main.cpp
    src/AssetManager.cpp
    src/Entity.cpp
    src/Player.cpp
    src/Enemy.cpp
    src/Projectile.cpp
    src/Item.cpp
    src/Room.cpp
    src/Level.cpp
    src/HUD.cpp
    src/Game.cpp
)

add_executable(${PROJECT_NAME} ${SOURCES})
target_include_directories(${PROJECT_NAME} PRIVATE src)
target_link_libraries(${PROJECT_NAME} PRIVATE sfml-graphics sfml-window sfml-system sfml-audio)

# MSVC по умолчанию считает исходники Windows-1251, из-за чего русские строки
# в UTF-8 ломаются. /utf-8 = исходник и runtime-кодировка узких строк — UTF-8.
if(MSVC)
    target_compile_options(${PROJECT_NAME} PRIVATE /utf-8)
endif()

# Копируем ресурсы (шрифт и т.п.) рядом с .exe при каждой сборке.
# Вложенный if + file(GLOB) — чтобы не падать, если assets/ ещё пустая
# (CMake 4.x ругается на copy_directory с пустым источником).
file(GLOB ASSET_FILES "${CMAKE_SOURCE_DIR}/assets/*")
if(ASSET_FILES)
    add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD
        COMMAND ${CMAKE_COMMAND} -E make_directory
            $<TARGET_FILE_DIR:${PROJECT_NAME}>/assets
        COMMAND ${CMAKE_COMMAND} -E copy_directory
            ${CMAKE_SOURCE_DIR}/assets $<TARGET_FILE_DIR:${PROJECT_NAME}>/assets
    )
endif()
