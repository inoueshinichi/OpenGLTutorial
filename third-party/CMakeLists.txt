cmake_minimum_required(VERSION 3.16.3)

# ===== Compile Options of GLFW =====
# https://www.glfw.org/docs/latest/compile_guide.html#compile_generate
# BUILD_SHARED_LIBS    : OFF
# GLFW_VULKAN_STATIC   : OFF
# -----------------------------------
# Windows options
# USE_MSVC_RUNTIME_LIBRARY_DLL : ON
# GLFW_USE_HYBRID_HPG          : OFF

set(GLFW_TARGET_NAME "glfw-3.3.2")

string(APPEND GLFW_COMMAND_LINE_OPTIONS " -DBUILD_SHARED_LIBS=OFF")
string(APPEND GLFW_COMMAND_LINE_OPTIONS " -DGLFW_VULKAN_STATIC=OFF")


if(MSVC)
    string(APPEND GLFW_COMMAND_LINE_OPTIONS " -DUSE_MSVC_RUNTIME_LIBRARY_DLL=ON")
    string(APPEND GLFW_COMMAND_LINE_OPTIONS " -DGLFW_USE_HYBRID_HPG=OFF")
elseif(UNIX)
    # Install xorg-dev package by apt pakcage manager.
else()

endif()

# Install prefix
set(GLFW_PREFIX ${CMAKE_CURRENT_BINARY_DIR}/${GLFW_TARGET_NAME})
set(GLFW_INSTALL_PATH ${GLFW_PREFIX}/install)
string(APPEND GLFW_COMMAND_LINE_OPTIONS " -DCMAKE_INSTALL_PREFIX:PATH=${GLFW_INSTALL_PATH}")
string(APPEND GLFW_COMMAND_LINE_OPTIONS " -DGLFW_INSTALL=ON")

# ############################################################################
# # Compile & Install with ExternalProject about GLFW
# ############################################################################

# # Enable ExternalProject
# include(ExternalProject)

# ExternalProject_Add(
#     # Target Name
#     ${GLFW_TAGET_NAME}

#     #################### No.0 Common Directory Path ####################
#     # 一時ファイルを保存するディレクトリパス
#     TMP_DIR ${GLFW_PREFIX}/tmp
#     # 各ステップで生成されるタイムスタンプを保存するディレクトリパス
#     STAMP_DIR ${GLFW_PREFIX}/stamp
#     # 各ステップでのLogファイルを保存するディレクトリパス
#     LOG_DIR ${GLFW_PREFIX}/log
#     # 解凍前のダウンロード内容(zipなど)を保存するディレクトリパス(※ gitでは, SOURCE_DIRに直接保存される)
#     DOWNLOAD_DIR ${GLFW_PREFIX}/archive
#     # (解凍後の)ダウンロード内容を保存するディレクトリパス
#     SOURCE_DIR ${GLFW_PREFIX}/src
#     # ビルド後の成果物を保存するディレクトリパス
#     BINARY_DIR ${GLFW_PREFIX}/build
#     # ビルド成果物のインストール先のディレクトリパス
#     # (※ 実際にはmake installのインストール先に、このプロパティは使われない! make install DESTDIR=~/path/toで指定するかCMakeの-Dオプションで指定)
#     INSTALL_DIR ${GLFW_INSTALL_PATH} #${GTEST_PREFIX}/install 

#     #################### No.1 Download Step ####################
#     # 原因不明で前回, git cloneを行っていないのに、すでにgit cloneされていると判断されて、git cloneが実行されない.
#     # Git Repository URL
#     GIT_REPOSITORY https://github.com/glfw/glfw
#     # Git branch-name, tag or commit's hash id
#     GIT_TAG 3.3.2
#     # Git Remote Name (default: origin)
#     GIT_REMOTE_NAME origin
#     # Git Remote Clone Progress
#     GIT_REMOTE_PROGRESS ON
#     # Timeout
#     TIMEOUT 10 # 10秒

#     #################### No.2 Update/Patch Step ####################
#     # ダウンロードした内容にパッチを当てるコマンド
#     UPDATE_COMMAND ""

#     #################### No.3 Configure Step ####################
#     # ジェネレータ
#     # CMAKE_GENERATOR "Unix Makefiles" # デフォルトのジェネレーターを使用する
#     # cmake コマンドに渡す引数
#     CMAKE_ARGS ${GLFW_COMMAND_LINE_OPTIONS}

#     #################### No.4 Build Step ####################
#     # ビルドコマンド
#     # BUILD_COMMAND make # デフォルトのビルドコマンドを使用する
#     # Build In Source or Build Out Source
#     BUILD_IN_SOURCE OFF
#     # Build Always or Not
#     BUILD_ALWAYS OFF

#     #################### No.5 Install Step ####################
#     # インストールコマンド
#     INSTALL_COMMAND make install # インストールステップをスキップ

#     #################### No.6 Test Step ####################

#     #################### No.7 Output Logging ####################
#     LOG_DOWNLOAD ON
#     LOG_UPDATE ON
#     LOG_PATCH ON
#     LOG_CONFIGURE ON
#     LOG_BUILD ON
#     LOG_INSTALL ON
#     LOG_TEST ON
#     LOG_MERGED_STDOUTERR ON
#     LOG_OUTPUT_ON_FAILURE ON

#     #################### No.8 Terminal Access ####################
#     #################### No.9 Target ####################
#     #################### No.10 Miscellaneous ####################
# )

# # ExternalProject_Get_Property(${GTEST_TARGET} INSTALL_DIR)

#############################################################################
# Compile & Install with download_and_extract_library about GLFW
#############################################################################

# ダウンロードと解凍を行う関数
function(download_and_extract_library NAME EXT URL)
    if(NOT IS_DIRECTORY ${CMAKE_SOURCE_DIR}/third-party/${NAME})
        message(STATUS "Downloading ${NAME}")
        file(DOWNLOAD
            ${URL}
            ${CMAKE_CURRENT_SOURCE_DIR}/${NAME}${EXT}
            SHOW_PROGRESS
            STATUS status
            LOG log)
        list(GET status 0 status_code)
        list(GET status 1 status_string)
        if(NOT status_code EQUAL 0)
            if(UNIX OR APPLE)
                message("Failed to download ${NAME}. Try to download with wget.")
                execute_process(
                    COMMAND wget ${URL} -O ${CMAKE_CURRENT_SOURCE_DIR}/${NAME}${EXT}
                )
            else()
                message(FATAL_ERROR "Failed to download ${NAME}. Please check network connection and proxy settings.
                status_code: ${status_code}
                status_string: ${status_string}
                log: ${log}
                ")
                error() # Place Unknown command to stop cmake process here.
            endif()
        endif()

        execute_process(
            COMMAND ${CMAKE_COMMAND} -E 
            chdir ${CMAKE_CURRENT_SOURCE_DIR}
            ${CMAKE_COMMAND} -E tar xf ${NAME}${EXT}
        )
    endif()
endfunction()


###############################################################################
# 以下に追加したいサードパーティ製のライブラリを明記
###############################################################################

# GLFW 3.3.2
message(STATUS "Download and Extract library: ${GLFW_TARGET_NAME}")
download_and_extract_library(${GLFW_TARGET_NAME} .zip https://github.com/glfw/glfw/releases/download/3.3.2/glfw-3.3.2.zip)

add_subdirectory(${GLFW_TARGET_NAME})

# message(STATUS "GLFW Compile Options: ${GLFW_COMMAND_LINE_OPTIONS}")
# execute_process(
#     COMMAND ${CMAKE_COMMAND} ${GLFW_COMMAND_LINE_OPTIONS} -S ${CMAKE_CURRENT_SOURCE_DIR}/${GLFW_TARGET_NAME} -B ${CMAKE_CURRENT_BINARY_DIR}/${GLFW_TARGET_NAME}
# )

