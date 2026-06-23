# CLAUDE.md — ioquake3 项目说明

> 本文件供 AI 助手（Claude / CodeBuddy 等）快速理解项目。修改项目结构、构建方式或约定时同步更新。

## 1. 项目来源

- **上游仓库**：https://github.com/ioquake/ioq3（master 分支）
- **官方网站**：https://ioquake3.org
- **下载方式**：从 GitHub 上游下载源码包（非 git clone，本目录**不是 git 仓库**，无 `.git/`）。
- **性质**：ioquake3 — id Software Quake III Arena 引擎的开源现代化维护版本，基于 GPL v2。
- **本地路径**：`D:\3rd-party-projects\ioquake3`

## 2. 项目结构

```
ioquake3/
├── code/              # 引擎与游戏逻辑源码（C）
│   ├── client/        # 客户端（renderer 接口、输入、声音）
│   ├── server/        # 服务器
│   ├── qcommon/       # 公共引擎核心
│   ├── renderer/      # 渲染器（renderer_opengl1 / renderer_opengl2）
│   ├── cgame/         # 客户端游戏逻辑（QVM/DLL）
│   ├── game/          # 服务器游戏逻辑（QVM/DLL）
│   ├── ui/            # 菜单 UI
│   ├── botlib/        # Bot AI
│   └── ...
├── ui/                # 菜单脚本源
├── misc/              # 图标、脚本、打包资源
├── cmake/             # CMake 模块
├── docs/              # 项目文档（见第 6 节）
├── build/Release/     # 构建产物（不入库的本地输出）
├── CMakeLists.txt     # 构建入口
├── COPYING.txt        # GPL v2
├── README.md          # 上游 README
└── CLAUDE.md          # 本文件
```

## 3. 构建环境（已验证）

- **工具链**：MSVC 2022 + CMake
- **关键选项**：`-DUSE_INTERNAL_LIBS=ON`（内部打包 SDL2/OpenAL 等，无需额外安装依赖）
- **构建类型**：Release
- **构建日期**：2026-06-19
- **详细记录**：见 `docs/build-windows.md`

### 构建产物（`build/Release/`）

| 文件 | 说明 |
|------|------|
| `ioquake3.exe` | 游戏客户端 |
| `ioq3ded.exe` | 独立专用服务器 |
| `renderer_opengl1.dll` | GL1 渲染器（固定管线） |
| `renderer_opengl2.dll` | GL2 渲染器（现代着色器，默认） |
| `SDL2.dll` | 内部打包的 SDL2 运行库 |
| `baseq3/` | cgame.dll + qagame.dll + ui.dll + vm/*.qvm |
| `missionpack/` | Team Arena 游戏逻辑（dll + qvm） |

## 4. 游戏资源

- **商业数据**：`D:\games\Quake_III_Arena\baseq3\pak0.pk3`（372 MB，完整版）
- **原游戏目录**含大量第三方地图 pk3，但**无 missionpack**（Team Arena 不可玩）
- **部署方式**：零复制启动，用 `+set fs_basepath "D:\games\Quake_III_Arena"` 指向原目录

## 5. 运行方式

### 推荐启动命令（已实测，性能最优）

```bash
ioquake3.exe +set fs_basepath "D:\games\Quake_III_Arena" +set r_fullscreen 0 +set r_mode 8 +set r_postProcess 0 +set r_finish 0 +set com_maxfps 125 +set cl_renderer opengl2
```

关键点（用户机器：3.2K 屏 + Arc 140T 集显 + RTX 5060 独显）：
- `r_mode 8`（1280×1024）替代 `-2`（桌面 3200×2000），避免老引擎像素填充瓶颈
- `r_postProcess 0` 关闭 GL2 后处理
- `r_finish 0` 解除每帧 CPU/GPU 同步阻塞

完整使用说明见 `docs/usage.md`。

## 6. 项目文档

| 文件 | 内容 |
|------|------|
| `docs/build-windows.md` | Windows MSVC + CMake 构建记录 |
| `docs/usage.md` | 使用说明（部署/启动/控制台/性能调优/FAQ） |
| `docs/opengl2-readme.md` | GL2 渲染器说明（上游） |
| `README.md` | 上游项目 README |
| `CLAUDE.md` | 本文件（AI 助手项目指南） |

## 7. 重要约定

1. **本目录非 git 仓库**：不要执行 git 命令，不要假设存在 `.git/`。
2. **构建产物不入库**：`build/` 是本地输出，重构会覆盖。
3. **商业数据不复制进项目**：通过 `fs_basepath` 指向 `D:\games\Quake_III_Arena`。
4. **用户配置位置**：`%APPDATA%\Quake3\`（`fs_homepath`），与项目分离。
5. **修改源码后**：用 CMake 重新构建，产物在 `build/Release/`，无需复制数据。
6. **文档语言**：中文为主，命令/参数/cvar 用英文原文。
7. **路径规范**：回复中引用项目内文件用相对路径（如 `docs/usage.md`），引用项目外用绝对路径。

## 8. 常见任务

| 任务 | 命令/参考 |
|------|----------|
| 重新构建 | 见 `docs/build-windows.md` |
| 启动游戏 | 见第 5 节推荐命令 |
| 调整性能 | `docs/usage.md` 第 7 章 |
| 添加第三方地图 | 放入 `D:\games\Quake_III_Arena\baseq3\`，启动时 `sv_pure 0` |
| 启动专用服务器 | `docs/usage.md` 第 8 章 |

---

最后更新：2026-06-23
