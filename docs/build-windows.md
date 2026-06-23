# ioquake3 编译记录

## 环境

| 项目 | 版本 / 路径 |
|------|-------------|
| 系统 | Windows 11 x64 (10.0.26200) |
| 编译器 | MSVC 19.51.36246.0 (VS 2022 BuildTools) |
| CMake | 3.x (VS 内置) |
| CMake 路径 | `C:\Program Files (x86)\Microsoft Visual Studio\18\BuildTools\Common7\IDE\CommonExtensions\Microsoft\CMake\CMake\bin\cmake.exe` |
| MSBuild | 18.6.3+84d3e95b4 |
| Windows SDK | 10.0.26100.0 |
| 源码 | `D:\3rd-party-projects\ioquake3` (ioquake/ioq3 master) |

## 依赖

使用内部打包库 (`USE_INTERNAL_LIBS=ON`)，无需额外安装 SDL2、OpenAL、libjpeg、zlib、ogg、vorbis、opus 等。全部源码自带。

## 编译步骤

```powershell
# 1. 获取源码（GitHub 被墙，用 ghfast.top 镜像）
aria2c -x 16 -s 16 -o ioquake3.zip "https://ghfast.top/https://github.com/ioquake/ioq3/archive/refs/heads/master.zip"
Expand-Archive ioquake3.zip -DestinationPath D:\3rd-party-projects
Rename-Item D:\3rd-party-projects\ioq3-main D:\3rd-party-projects\ioquake3

# 2. CMake 配置 (x64 Release)
$cmake = "C:\Program Files (x86)\Microsoft Visual Studio\18\BuildTools\Common7\IDE\CommonExtensions\Microsoft\CMake\CMake\bin\cmake.exe"
& $cmake -S D:\3rd-party-projects\ioquake3 -B D:\3rd-party-projects\ioquake3\build `
    -G "Visual Studio 18 2026" -A x64 `
    -DUSE_INTERNAL_LIBS=ON

# 3. 编译
& $cmake --build D:\3rd-party-projects\ioquake3\build --config Release --parallel
```

## 产物

```
D:\3rd-party-projects\ioquake3\build\Release\
├── ioquake3.exe              # 游戏客户端
├── ioq3ded.exe               # 独立服务器
├── renderer_opengl1.dll      # GL1 渲染器（经典）
├── renderer_opengl2.dll      # GL2 渲染器（现代着色器）
├── baseq3/
│   ├── cgame.dll
│   ├── qagame.dll
│   └── ui.dll
├── baseq3/vm/
│   ├── cgame.qvm
│   ├── qagame.qvm
│   └── ui.qvm
└── missionpack/
    ├── cgame.dll
    ├── qagame.dll
    └── ui.dll
```

## 警告

仅 2 个 C4701/C4703 警告（`tr_image_pvr.c:340`，可能未初始化的局部指针），不影响运行。

## 运行条件

需要在 `baseq3/` 目录放置 `pak0.pk3` 等游戏数据文件（约 450MB），可从以下渠道获取：

1. Steam 购买 Quake III Arena 后复制 pak 文件
2. Quake III Arena Demo 版 `pak0.pk3`（阉割版，地图较少）

放置后运行 `ioquake3.exe` 即可。

---

编译日期：2026-06-19
