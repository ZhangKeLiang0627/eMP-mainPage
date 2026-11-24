# eMP_mainPage

Allwinner T113-S3 板子的主界面（Launcher），基于 LVGL 8.3 + sunxifb，负责展示 eMP 系列应用入口并跳转。

## Snapshots

![](./pictures/image-1.jpg)
![](./pictures/image-2.png)

## 环境

```shell
# 交叉编译需要（按本机实际路径修改）
export STAGING_DIR=/home/hugokkl/tina-sdk/out/t113-pi/staging_dir/target
```

## 编译

- Makefile
```shell
# 交叉编译
./build.sh

# 本地编译
./build.sh 0
```

- CMake
```shell
# 交叉编译
export STAGING_DIR=/home/hugokkl/tina-sdk/out/t113-pi/staging_dir/target
mkdir build && cd build
cmake -DCMAKE_TOOLCHAIN_FILE=cmake/build_for_t113s3.cmake ..
make -j32

# 本地编译
mkdir build && cd build
cmake ..
make -j32
```

## 运行

可执行文件为：`eMP_mainPage`

> **注意：本地编译，这需要一些文件依赖和库依赖才可以编译/运行起来...**

### 本地编译的库依赖
```bash
sudo apt update

# 包含 gcc、g++、make 等基础工具
sudo apt install build-essential  

# SDL2库、Freetype
sudo apt install libsdl2-dev libfreetype6-dev libncurses5-dev libstdc++6 

# Plus: 如果cmake时遇到Wayland依赖找不到的情况时，请输入以下指令安装
sudo apt install -y wayland-protocols libxkbcommon-dev
```

### 本地编译的文件依赖

```bash
# 请创建对应文件夹
sudo mkdir /mnt/UDISK /mnt/UDISK/applications
```

并将[此处的内容](https://github.com/ZhangKeLiang0627/EasyMediaPlayer/tree/main/firmware)全部移入该文件夹中即可！

## 应用自动发现

每个应用在 `/mnt/UDISK/applications/` 放一个 `.desktop` 描述文件，启动时扫描、运行中用 inotify
监听目录变化，按可执行路径增量加载按钮；`./config/sysconfig.json` 的 `applications` 数组仍作兜底。

字段含义、解析规则与 Exec 安全校验见 [`docs/launch-system.md`](docs/launch-system.md#2-应用描述格式desktop)。

## 应用启动器（Launch 模块）

`src/Launch/` 负责描述解析、目录扫描与启动：fork + `setpgid` + subreaper，`waitpid` 收退出码，
flock 防重复启动，运行期间暂停 LVGL 渲染以免主界面覆盖应用画面。

实现细节与旧实现的缺陷对比见 [`docs/launch-system.md`](docs/launch-system.md#4-启动器app_runner)。

## 网络对时（ARM）

T113-S3 无 RTC（默认 1970），HTTPS 会因证书 "not yet valid" 失败。ARM 编译下启动时对时一次、
data 线程每 60s 一次，数据源 WorldTimeAPI → Baidu `Date` 头回退。

详见 [`docs/launch-system.md`](docs/launch-system.md#5-网络对时timesync)。

## 文件

- `./config/sysconfig.json`：亮度/音量等系统参数 + 旧版 applications 兜底
- `./inc/Launch/` + `./src/Launch/`：应用描述 / 扫描 / 启动器
- `./inc/TimeSync.h` + `./src/Net/TimeSync.cpp`：网络对时
- `./libs/nlohmann/json.hpp`：JSON 解析（替代 cJSON）

## 详细设计

见 [`docs/launch-system.md`](docs/launch-system.md)
