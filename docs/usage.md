# ioquake3 使用说明

本仓库已用 MSVC + CMake 完成 Release 构建，游戏数据 `pak0.pk3`（372 MB，完整版）已就位。
本文档说明如何把构建产物与游戏数据组合并运行。编译过程见 [`build-windows.md`](./build-windows.md)。

## 1. 运行前提

| 项目 | 要求 |
|------|------|
| 操作系统 | Windows 10/11 x64 |
| 图形 API | 支持 OpenGL 2.1+ 的 GPU 与驱动（GL2 渲染器为默认） |
| 游戏数据 | `D:\games\Quake_III_Arena\baseq3\pak0.pk3`（已就绪） |
| 构建产物 | `D:\3rd-party-projects\ioquake3\build\Release\`（已生成） |

> 提示：Quake III Arena 的商业数据受版权保护，ioquake3 源码本身不含地图/贴图/模型。需自行持有合法副本。

## 2. 构建产物清单

```
build/Release/
├── ioquake3.exe              # 游戏客户端
├── ioq3ded.exe               # 独立专用服务器
├── SDL2.dll                  # 内部打包的 SDL2 运行库
├── renderer_opengl1.dll      # GL1 渲染器（经典管线）
├── renderer_opengl2.dll      # GL2 渲染器（现代着色器，默认）
├── baseq3/
│   ├── cgame.dll  qagame.dll  ui.dll     # 游戏逻辑（本地编译版）
│   └── vm/        cgame.qvm  qagame.qvm  ui.qvm   # 虚拟机字节码
└── missionpack/
    ├── cgame.dll  qagame.dll  ui.dll
    └── vm/        cgame.qvm  qagame.qvm  ui.qvm
```

引擎二进制与自带的 `baseq3` / `missionpack` 游戏逻辑均由本次构建产出，无需额外安装。运行时只差商业数据 `pak0.pk3`。

## 3. 部署游戏数据（二选一）

ioquake3 通过 `fs_basepath`（基础数据路径，默认 = 可执行文件所在目录）定位 `baseq3/`，会按文件名顺序加载其中的 `.pk3`。提供两种部署方式：

### 方式一：指定 `fs_basepath`（推荐，零复制）

直接让引擎读取原游戏目录，不复制任何文件，且原目录下的第三方地图 pk3 也会一并加载：

```bash
# Git Bash / CMD
ioquake3.exe +set fs_basepath "D:\games\Quake_III_Arena"
```

- 优点：不占额外空间；自动加载 `D:\games\Quake_III_Arena\baseq3\` 下全部 pk3（含第三方地图）。
- 注意：引擎自带的 `cgame.dll` 等位于 `build\Release\baseq3\`，仍会被加载（ioquake3 同时搜索 exe 同目录的 baseq3）；原目录若存在旧版同名 dll，以引擎自带版本为准。
- 建议做成快捷方式：右键 `ioquake3.exe` → 创建快捷方式 → 在“目标”末尾追加 ` +set fs_basepath "D:\games\Quake_III_Arena"`。

### 方式二：复制 `pak0.pk3` 到构建目录

把核心数据复制到 `build\Release\baseq3\`，之后双击 exe 即可运行，无需参数：

```bash
cp "D:/games/Quake_III_Arena/baseq3/pak0.pk3" "D:/3rd-party-projects/ioquake3/build/Release/baseq3/"
```

- 优点：启动简单，双击即可；路径自洽，便于整体搬迁。
- 缺点：多占用约 372 MB；仅加载 pak0，不含原目录的第三方地图（如需可一并复制对应 pk3）。
- 节省空间可用符号链接（需管理员/开发者模式）：
  ```powershell
  mklink "D:\3rd-party-projects\ioquake3\build\Release\baseq3\pak0.pk3" "D:\games\Quake_III_Arena\baseq3\pak0.pk3"
  ```

> 用户配置与存档默认写入 `%APPDATA%\Quake3\`（`fs_homepath`），与游戏数据分离，卸载/升级引擎不会丢失。可用 `+set fs_homepath <path>` 自定义。

## 4. 启动客户端

### 直接运行
双击 `build\Release\ioquake3.exe`（方式二），或用方式一的带参快捷方式。

### 常用启动参数

| 参数 | 作用 |
|------|------|
| `+set fs_basepath "<路径>"` | 指定游戏数据根目录 |
| `+set fs_homepath "<路径>"` | 指定用户配置/存档目录 |
| `+set cl_renderer opengl2` | 使用 GL2 渲染器（默认，画质更好） |
| `+set cl_renderer opengl1` | 使用 GL1 渲染器（兼容老旧 GPU） |
| `+set r_mode -2` | 自动使用桌面分辨率 |
| `+set r_fullscreen 0` | 窗口模式（1 = 全屏） |
| `+set com_basegame baseq3` | 指定基础 mod（默认 baseq3） |
| `+set com_homepath Quake3` | 指定用户数据目录名 |
| `+connect <ip:port>` | 直接连接服务器 |
| `+devmap <mapname>` | 开发模式加载地图（带作弊） |
| `+set sv_pure 0` | 关闭纯服务器校验（加载第三方 pk3 时常用） |

示例——窗口模式 + GL2 + 读取原游戏目录：

```bash
ioquake3.exe +set fs_basepath "D:\games\Quake_III_Arena" +set r_fullscreen 0 +set r_mode -2 +set cl_renderer opengl2
```

## 5. 控制台与常用命令

按 `~`（或 `` ` ``）打开控制台。常用命令：

| 命令 | 作用 |
|------|------|
| `/map <name>` | 加载地图，如 `/map q3dm1` |
| `/devmap <name>` | 加载地图并启用作弊 |
| `/connect <ip>` | 连接服务器 |
| `/disconnect` | 断开当前连接 |
| `/quit` | 退出 |
| `/screenshot` | 保存 PNG 截图 |
| `/screenshotJPEG` | 保存 JPEG 截图（质量见 `r_screenshotJpegQuality`） |
| `/record <name>` | 开始录制 demo |
| `/stoprecord` | 停止录制 |
| `/demo <name>` | 回放 demo |
| `/video [file]` | demo 回放时开始视频捕获（AVI） |
| `/stopvideo` | 停止视频捕获 |
| `/addbot <name>` | 添加 bot（`addbot random` 随机） |
| `/kickall` | 踢出所有玩家 |
| `/cvarlist <filter>` | 列出 cvar（可过滤，如 `r*`） |
| `/set <cvar> <val>` | 设置 cvar |

### 实用 cvar

- `com_maxfps` — 帧率上限（如 `125`）
- `r_fullscreen` / `r_mode` — 全屏与分辨率（`r_mode -2` = 桌面分辨率）
- `s_useOpenAL` — 1 用 OpenAL（多声道、音质更好），0 用 SDL 后端
- `con_scale` — 控制台文字缩放（高分屏建议 `1.5`~`2`）
- `sv_pure` — 1 校验 pk3 哈希（比赛），0 允许第三方文件

完整 cvar/命令清单见仓库根目录 `README.md` 的 Console 章节。

## 6. Team Arena（missionpack）

`build\Release\missionpack\` 已构建好扩展包的游戏逻辑（dll + qvm），但运行 Team Arena **还需要商业数据文件**（`missionpack\pak0.pk3` 等）。

当前 `D:\games\Quake_III_Arena\` 下**没有 missionpack 目录**，因此：

- 暂时无法运行 Team Arena 内容。
- 如需游玩，需另行获取 Team Arena 资源放入 `D:\games\Quake_III_Arena\missionpack\`，再用方式一启动（basepath 自动覆盖 missionpack）。
- 启动 Team Arena：`ioquake3.exe +set fs_game missionpack +set fs_basepath "D:\games\Quake_III_Arena"`

## 7. 性能调优（实测）

### 7.1 卡顿根因（按影响排序）

在 3.2K 屏笔记本（Intel Arc 140T 主显 + RTX 5060 独显）上实测，ioquake3 出现严重卡顿的根因：

| 排名 | 因素 | 默认/常见值 | 问题 |
|------|------|------|------|
| **0** | **GPU 选择** | **ioquake3.exe 默认跑集显** | **隐藏主因**。NVIDIA Optimus 不识别 `ioquake3.exe`，自动走集显；原版 `quake3.exe` 在驱动游戏数据库中，自动走独显 |
| 1 | `r_mode` | `-2`（桌面分辨率） | 3200×2000 渲染 640 万像素/帧，单线程引擎像素填充瓶颈 |
| 2 | `r_postProcess` | `1` | GL2 后处理（bloom + tonemap）全分辨率多次采样 FBO |
| 3 | `r_finish` | `1` | 每帧 `glFinish()` 强制 CPU 阻塞等待 GPU |

> ioquake3 是 1999 年单线程引擎，瓶颈在 CPU 命令提交与像素着色总量。但**首要前提是让游戏跑在独显上**——否则再怎么调 cvar 也救不回来。详见 7.9 节。

### 7.2 推荐启动命令（实测可用）

```bash
ioquake3.exe +set fs_basepath "D:\games\Quake_III_Arena" +set r_fullscreen 0 +set r_mode 8 +set r_postProcess 0 +set r_finish 0 +set com_maxfps 125 +set cl_renderer opengl2
```

参数说明：

| 参数 | 值 | 作用 |
|------|------|------|
| `r_mode` | `8` | 1280×1024 窗口，像素量从 640 万降到 130 万（约 5× 提升） |
| `r_postProcess` | `0` | 关闭 GL2 后处理（bloom/tonemap） |
| `r_finish` | `0` | 解除 CPU/GPU 帧间同步阻塞 |
| `com_maxfps` | `125` | 标准竞技帧率上限（Q3 物理引擎最佳帧率） |
| `r_fullscreen` | `0` | 窗口模式，便于切换调试 |

### 7.3 分辨率档位（`r_mode`）

| 值 | 分辨率 | 适用场景 |
|----|--------|----------|
| `4` | 800×600 | 最稳，老旧机器首选 |
| `6` | 1024×768 | 画质/性能平衡 |
| `8` | 1280×1024 | **推荐**，现代屏够清晰 |
| `-1` | 自定义 | 配合 `r_customwidth` / `r_customheight`，如 1920×1200 |
| `-2` | 桌面分辨率 | 高 DPI 屏慎用，性能开销大 |

自定义示例（1920×1200 窗口）：

```bash
ioquake3.exe +set fs_basepath "D:\games\Quake_III_Arena" +set r_fullscreen 0 +set r_mode -1 +set r_customwidth 1920 +set r_customheight 1200 +set r_postProcess 0 +set r_finish 0 +set com_maxfps 125
```

### 7.4 持久化设置

启动参数只对本次会话生效。要在控制台写入配置文件（重启后保留）：

```
/r_mode 8
/r_postProcess 0
/r_finish 0
/com_maxfps 125
/writeconfig q3config.cfg
```

或直接编辑 `%APPDATA%\Quake3\baseq3\q3config.cfg`，把对应行改为 `seta r_mode "8"` 等。

### 7.5 进一步排查

1. **GPU 选择（关键前提）**：ioquake3.exe 默认跑集显，必须强制走独显。三种方法按推荐度排序见 7.9 节，**改名法最简单**（实测可行）。
2. **切换 GL1 渲染器对比**：`+set cl_renderer opengl1`（更轻量，无后处理路径，用于定位是否为 GL2 路径问题）。
3. **抗锯齿**：`r_ext_multisample` 默认 `0`，若开了 MSAA 在高分辨率下开销显著，建议保持 0。
4. **纹理过滤**：`r_textureMode` 默认 `GL_LINEAR_MIPMAP_NEAREST`（双线性+最近邻 mipmap），性能优先；改 `GL_LINEAR_MIPMAP_LINEAR`（三线性）画质更好但略慢。
5. **驱动**：Intel Arc 的 OpenGL 驱动偶有性能回归，可对比 NVIDIA 独显路径。

### 7.6 关键 cvar 速查

| cvar | 默认 | 说明 |
|------|------|------|
| `r_mode` | `3` (640×480) | 分辨率模式，见 7.3 |
| `r_fullscreen` | `1` | 0=窗口，1=全屏 |
| `cl_renderer` | `opengl2` | `opengl1`（经典固定管线）/ `opengl2`（现代着色器） |
| `r_postProcess` | `1` | GL2 后处理（bloom+tonemap），关掉可显著提速 |
| `r_finish` | `1` | 每帧 glFinish 同步，关掉减少 CPU 阻塞 |
| `r_swapInterval` | `0` | 垂直同步（0=关，1=开） |
| `com_maxfps` | `85` | 帧率上限，竞技常用 125 |
| `r_picmip` | `1` | 纹理压缩等级（0=最高画质，越大越糙） |
| `r_gamma` | `1` | 伽马（1.5~2 高分屏更舒服） |
| `r_ext_multisample` | `0` | MSAA 采样数（0=关） |
| `r_textureMode` | `GL_LINEAR_MIPMAP_NEAREST` | 纹理过滤模式 |

### 7.7 与原版 quake3.exe 的对比

**疑问**：原版 `D:\games\Quake_III_Arena\quake3.exe` 直接双击全屏运行很流畅，ioquake3 却卡。原版用了什么参数？

**结论**：原版没有隐藏参数，流畅的原因是配置组合不同（读取原版 `baseq3\q3config.cfg` 实测）：

| cvar | 原版 quake3.exe | ioquake3 默认 | 影响 |
|------|------|------|------|
| `r_mode` | **`6` (1024×768)** | `-2` (桌面分辨率) | **主因**，像素量差 8 倍 |
| `r_fullscreen` | `1` (全屏独占) | `0` (窗口) | 全屏独占走驱动游戏优化路径 |
| `r_postProcess` | 不存在（GL1 无后处理） | `1` (GL2 bloom+tonemap) | ioquake3 新增的 GL2 后处理 |
| `cl_renderer` | 不存在（仅固定管线） | `opengl2` | 原版无 GL2 代码路径 |
| `r_finish` | `1` | `1` | 相同，1024×768 下非瓶颈 |
| `com_maxfps` | `85` | `85` | 相同 |
| `r_picmip` | `1` | `1` | 相同 |

> 原版 `r_lastValidRenderer` = "NVIDIA GeForce RTX 5060 Laptop GPU/PCIe/SSE2"，说明原版跑在独显上。

**核心差异**：原版跑 **1024×768 全屏独占**，像素量仅 80 万；ioquake3 若用 `r_mode -2` 在 3.2K 屏上是 640 万像素，差 8 倍。加上 ioquake3 默认开 GL2 后处理，进一步放大开销。原版引擎根本没有 `r_postProcess` / `cl_renderer` 这些 cvar——它只有固定管线一条路径。

### 7.8 复刻原版体验

**方案 A：ioquake3 + GL2（推荐，画质更好，性能无忧）**

```bash
ioquake3.exe +set fs_basepath "D:\games\Quake_III_Arena" +set r_fullscreen 1 +set r_mode 6 +set cl_renderer opengl2 +set r_postProcess 0 +set com_maxfps 125
```

- `r_mode 6` = 1024×768 全屏，与原版一致（全屏自动拉伸到桌面）
- `r_postProcess 0` 关 GL2 后处理
- 保留 GL2 渲染器：1024×768 下毫无压力，且支持现代着色器画质提升
- `com_maxfps 125` 比原版 85 更顺滑

**方案 B：ioquake3 + GL1（与原版渲染路径完全相同）**

```bash
ioquake3.exe +set fs_basepath "D:\games\Quake_III_Arena" +set r_fullscreen 1 +set r_mode 6 +set cl_renderer opengl1 +set com_maxfps 125
```

加 `cl_renderer opengl1` → 走固定管线，与原版 quake3.exe 渲染路径完全相同，无任何后处理。适合追求"原汁原味"或排查 GL2 兼容性问题。

**关于 `r_finish`**：原版也是 `1`，但在 1024×768 下 CPU/GPU 同步开销可忽略，只有高分辨率（3.2K）下 `r_finish 1` 才成为瓶颈。因此 1024×768 场景下保持 `1` 即可（避免画面撕裂），7.2 节推荐 `r_finish 0` 是针对 3.2K 高分屏的应急方案。

### 7.9 GPU 选择（关键前提）：让 ioquake3 走独显

实测发现：原版 `quake3.exe` 直接双击就走 RTX 5060 独显，而 `ioquake3.exe` 默认走集显（`r_lastValidRenderer` 显示为 Intel HD Graphics 4000）。这是 ioquake3 在双 GPU 笔记本上卡顿的**隐藏主因**。

#### 原因：NVIDIA Optimus 按 exe 名识别

NVIDIA 驱动内置一个**已知游戏 exe 名数据库**（随驱动下发），常见游戏名直接命中自动走独显：

| exe 文件名 | 驱动识别 | 默认 GPU |
|------|------|------|
| `quake3.exe` | ✅ 命中"Quake III Arena" | 独显 |
| `quake4.exe` / `hl2.exe` / `csgo.exe` 等 | ✅ 命中 | 独显 |
| `ioquake3.exe` | ❌ 不在数据库 | 集显（按省电策略） |

ioquake3 引擎本身**没有指定 GPU 的命令行参数**——OpenGL 上下文由 GPU 驱动创建，应用层无法干预。必须从系统层面解决。

#### 三种解决方案（实测均可，按推荐度排序）

**方法 1：改名法（最简单，实测首选）**

把 `ioquake3.exe` 复制/重命名为 `quake3.exe`：

```powershell
Copy-Item "D:\3rd-party-projects\ioquake3\build\Release\ioquake3.exe" `
          "D:\3rd-party-projects\ioquake3\build\Release\quake3.exe"
```

之后用 `quake3.exe` 启动，NVIDIA 驱动自动识别走独显：

```bash
quake3.exe +set fs_basepath "D:\games\Quake_III_Arena" +set r_fullscreen 1 +set r_mode 6 +set cl_renderer opengl2 +set r_postProcess 0 +set com_maxfps 125
```

- ✅ 零配置，驱动自动识别
- ✅ ioquake3 内部不依赖自身 exe 名（`fs_basepath` 默认 = exe 所在目录，与文件名无关）
- ⚠️ 重新构建后产物仍叫 `ioquake3.exe`，需再次复制
- ⚠️ 若之前为 `ioquake3.exe` 设过 NVIDIA 控制面板/注册表 GPU 偏好，改名后对 `quake3.exe` 不生效（但驱动自动识别已足够）

**方法 2：注册表 GpuPreference（持久，路径敏感）**

写入 Windows GPU 偏好注册表，对 `ioquake3.exe` 永久生效：

```powershell
$key = 'HKCU:\Software\Microsoft\DirectX\UserGpuPreferences'
$exe = 'D:\3rd-party-projects\ioquake3\build\Release\ioquake3.exe'
Set-ItemProperty -Path $key -Name $exe -Value 'GpuPreference=2;'
# GpuPreference=2 = 高性能（独显），=1 = 省电（集显）
```

- ✅ 不改文件名，对原 `ioquake3.exe` 生效
- ✅ 设置一次永久有效
- ⚠️ 路径敏感：exe 移动/重命名需重设
- ⚠️ 比 NVIDIA 驱动自动识别优先级低，若驱动已自动注入则不生效

**方法 3：Windows 图形设置 GUI（等价方法 2）**

`ms-settings:display-advancedgraphics` → 浏览添加 `ioquake3.exe` → 选项选"高性能" → 保存。底层写的还是方法 2 的注册表项。

#### 验证 GPU 是否切到独显

启动后控制台输入：

```
/r_lastValidRenderer
```

显示 `NVIDIA GeForce RTX 5060 Laptop GPU/PCIe/SSE2` 即生效。若仍是 Intel，说明设置未生效，换方法重试。

#### 为什么集显上报"Intel HD Graphics 4000"

实测 ioquake3 在集显路径下 `r_lastValidRenderer` 显示为 `Intel HD Graphics 4000`（2012 年型号），而非真实集显 Arc 140T 的名字。这是 Intel Arc OpenGL 驱动的**兼容性回退**——Arc 的 OpenGL 驱动在某些情况下回退到兼容模式，上报老设备名。这是 Intel Arc 驱动已知行为，不影响判断：只要不是 NVIDIA，就是没走独显。


## 8. 专用服务器

`ioq3ded.exe` 为无图形界面的专用服务器，适合挂机/局域网对战：

```bash
ioq3ded.exe +set fs_basepath "D:\games\Quake_III_Arena" +set dedicated 2 +exec server.cfg
```

- `dedicated 1` = 局域网，`dedicated 2` = 对外（上报主服务器）。
- 服务器配置写进 `%APPDATA%\Quake3\baseq3\server.cfg`（或 basepath 下），用 `+exec server.cfg` 加载。
- 玩家带宽限制见 `sv_dlRate`；自定义 pk3 的 HTTP 下载重定向见 `sv_dlURL`。
- 详见 https://ioquake3.org/help/sys-admin-guide/

## 9. 常见问题

**Q：启动报 "couldn't load default.cfg" / 找不到 pak0**
→ 游戏数据未就位。按第 3 节部署 `pak0.pk3`，或确认 `fs_basepath` 路径正确（注意路径用英文双引号包裹）。

**Q：黑屏 / 闪退 / OpenGL 报错**
→ 尝试切换渲染器：`+set cl_renderer opengl1`；或更新显卡驱动；或在控制台 `set r_mode 8` 改用 1280×1024（高 DPI 屏避免用 `r_mode -2`）。

**Q：第三方地图/模型不显示**
→ 以 `+set sv_pure 0` 启动，或把对应 pk3 放进 `baseq3\`。方式一（basepath 指向原游戏目录）会自动加载原目录全部 pk3。

**Q：没有声音**
→ 默认用 OpenAL，若系统缺 OpenAL 可 `+set s_useOpenAL 0` 回退 SDL 后端。

**Q：配置文件在哪**
→ Windows 默认 `%APPDATA%\Quake3\`（`fs_homepath`）。`q3config.cfg` 保存所有设置。

**Q：如何迁移/备份**
→ 备份 `%APPDATA%\Quake3\`（配置与存档）与 `build\Release\`（引擎）。游戏数据单独保管。

---

构建日期：2026-06-19 ｜ 文档更新：2026-06-23
