# 第三方组件与修改记录

本文件包含两部分：`OpenTTD.html` 中打包的第三方组件清单，以及相对 OpenTTD 17.0 原版源码的全部修改。

## 一、打包的组件

| 组件 | 版本 | 许可 | 版权 | 来源 |
| --- | --- | --- | --- | --- |
| OpenTTD | 17.0（master 快照，2026-11-08） | GNU GPL v2 | © 2003–2026 OpenTTD 开发团队及贡献者 | <https://github.com/OpenTTD/OpenTTD> |
| OpenGFX | 8.0 | GNU GPL v2 | © OpenGFX 团队 | <https://github.com/OpenTTD/OpenGFX> |
| OpenTTD 字体（Mono / Sans / Serif / Small） | 随 17.0 源码 | GNU GPL v2 | © OpenTTD 开发团队 | 同上 |
| SDL | 2.32.10 | zlib License | © 1997–2025 Sam Lantinga 及贡献者 | <https://github.com/libsdl-org/SDL> |
| zlib | 1.3.2 | zlib License | © 1995–2024 Jean-loup Gailly、Mark Adler | <https://zlib.net/> |
| libpng | 1.6.59 | libpng License | © 1995–2025 PNG Reference Library Authors | <http://www.libpng.org/> |
| XZ Utils / liblzma | 5.4.6 | 0BSD / LGPL-2.1（liblzma） | © 2009–2024 Lasse Collin；© 1999–2010 Igor Pavlov | <https://tukaani.org/xz/> |
| fmt | 11.1.4 | MIT | © 2012–2025 Victor Zverovich | <https://github.com/fmtlib/fmt> |
| Emscripten（WASM 运行时） | emsdk 7.0.0 | MIT / University of Illinois/NCSA | © 2010–2026 Emscripten authors | <https://emscripten.org/> |

完整许可文本见 [`licenses/`](licenses/)：

- `GPL-2.0.txt`（OpenTTD、OpenGFX、字体）
- `OpenGFX-license.txt`、`OpenGFX-readme.txt`
- `SDL2-license.txt`
- `zlib-license.txt`、`libpng-license.txt`
- `liblzma-COPYING.txt`、`LGPL-2.1.txt`
- `fmt-license.txt`
- `Emscripten-license.txt`

OpenTTD 与 OpenGFX 以 GPLv2 分发；本网页版包含的修改同样以 GPLv2 发布。对应源码完整提供于 `source/OpenTTD-17.0-web-src.tar.gz`，OpenGFX 8.0 的源码可从上述 OpenGFX 仓库获取（8.0 标签）。

## 二、相对 OpenTTD 17.0 的修改

除下列文件外，其余源码保持上游原样。

### 1. `CMakeLists.txt`

Emscripten 构建段：

- 所有 `--preload-file` 改为 `--embed-file`，把 baseset、语言文件、AI、游戏脚本与文档嵌入产物。
- 增加 `-sSINGLE_FILE=1`，把 WASM 以 base64 内联进 HTML。
- 增加 `-sENVIRONMENT=web`，只保留浏览器运行环境。

### 2. `os/emscripten/shell.html`

网页外壳，主要改动：

- 界面文字改为中文（载入中、错误提示、感谢游玩等）。
- 启动时在底部显示版权提示，首次交互后淡出。
- 新增触屏输入层（仅在 `navigator.maxTouchPoints > 0` 时启用）：
  - 触摸按下时先把游戏光标移动到触点，解决 SDL 在未悬停时忽略点击的问题；
  - 点按 = 左键；单指拖动 = 右键拖动（平移地图）；长按 450 ms = 右键；
  - 双指 = 右键拖动 + 捏合滚轮缩放；
  - 右下角按钮在"单指：移动"与"单指：建造"之间切换。
- 新增存档面板：列出 `/home/web_user/.openttd/save` 下的 `.sav`，单个/全部导出为下载文件，支持多选导入。
- 画布禁止滚动、缩放、长按菜单与文本选择，避免浏览器手势抢事件。

### 3. `os/emscripten/pre.js`

- 按浏览器窗口计算并追加 `-r 宽x高` 启动参数（上限 1600×1000，下限 640×480）。
- IndexedDB 可用性检测：不可用时跳过 IDBFS 挂载与同步，避免启动卡死。
- `syncfs` 读取增加完成保护与 5 秒超时。
- 暴露 `window.openttd_syncfs` 供页面主动写盘。

### 4. `src/openttd.cpp`

- 在 `__EMSCRIPTEN__` 下强制 `participate_survey = No`：网页版没有同意界面，不再弹出调查问卷，也不发送统计。

### 5. `src/core/enum_type.hpp`

- `DecayValueType` 增加 `typename`，兼容 MSVC 2019 对 C++20 依赖名的解析（本机交叉编译时使用）。

### 6. `src/3rdparty/fmt/format.h`

- 补充 `#include <cstdlib>`，修复本工具链下 `std::malloc` / `std::free` 未声明。

### 7. `.ottdrev`（新增）

- 源码快照不含 `.git`，写入版本信息 `17.0 / 20261108 / modified / web`，使游戏内显示 `OpenTTD 17.0` 而不是 `norev0000`。
