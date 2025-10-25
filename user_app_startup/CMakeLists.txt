cmake_minimum_required(VERSION 3.15)

# Interface library encapsulating the compiler and linker configuration
# required to build a user application against the rt-smart startup
# environment contained in this repository.
#
# 该接口库汇总了基于 rt-smart 启动环境进行用户程序构建所需的
# 编译器与链接器配置。使用方只需链接该库即可自动继承完整的
# 编译参数、链接脚本以及所需的库搜索路径，避免在项目中重复配置。
add_library(user_app_startup INTERFACE)
add_library(user_app_startup::rt_smart ALIAS user_app_startup)

set(USER_APP_STARTUP_SDK_ROOT
    "${CMAKE_CURRENT_LIST_DIR}"
    CACHE PATH "Path to the user_app_startup SDK root containing linker scripts and libraries.")

set(USER_APP_STARTUP_LINKER_SCRIPT
    "${USER_APP_STARTUP_SDK_ROOT}/linker_scripts/rt-smart_user_app.lds"
    CACHE FILEPATH "Linker script used for rt-smart user applications.")

# Headers shipped alongside the SDK.
#
# SDK 中包含的头文件路径。INTERFACE 目标会把该路径传播到所有依赖
# user_app_startup 的目标，确保使用者可以直接包含 RT-Thread/rt-smart
# 提供的接口。
target_include_directories(user_app_startup INTERFACE
    "${USER_APP_STARTUP_SDK_ROOT}/lib/rt-smart/rt-thread/include"
)

set(_user_app_startup_compile_flags
    -mcmodel=medany
    -march=rv64imafdcv
    -mabi=lp64d
)

# 为 C/C++ 源文件设置与 SDK 一致的编译选项。
# 这些选项指定了目标 RISC-V 处理器所支持的指令集扩展、数据模型
# 以及 ABI，保持与 SDK 内预编译库的二进制兼容性。
target_compile_options(user_app_startup INTERFACE
    $<$<OR:$<COMPILE_LANGUAGE:C>,$<COMPILE_LANGUAGE:CXX>>:${_user_app_startup_compile_flags}>
)

set(_user_app_startup_link_flags
    -mcmodel=medany
    -march=rv64imafdcv
    -mabi=lp64d
    -T
    "${USER_APP_STARTUP_LINKER_SCRIPT}"
    "SHELL:-Wl,--whole-archive -lrtthread -Wl,--no-whole-archive"
    -n
    --static
    "SHELL:-Wl,--start-group -lrtthread -Wl,--end-group"
)

# 链接阶段需要继承的 flags。除了保持与编译阶段相同的目标架构选项，
# 这里还显式指定了 rt-smart 用户程序所需的链接脚本，以及通过
# `--whole-archive` 与 `--start-group/--end-group` 控制器确保 rtthread 库
# 中的符号不会被链接器忽略。
target_link_options(user_app_startup INTERFACE
    ${_user_app_startup_link_flags}
)

# 指定 rt-smart SDK 提供的多个库搜索路径，供链接器在解析依赖时使用。
# 这些目录包含了用户程序启动所需的预编译静态库。
target_link_directories(user_app_startup INTERFACE
    "${USER_APP_STARTUP_SDK_ROOT}/lib/rt-smart/lib/risc-v/rv64"
    "${USER_APP_STARTUP_SDK_ROOT}/lib/rt-smart/rt-thread/lib"
    "${USER_APP_STARTUP_SDK_ROOT}/lib/rt-smart/rt-thread/lib/risc-v/rv64"
)

unset(_user_app_startup_compile_flags)
unset(_user_app_startup_link_flags)

