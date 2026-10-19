# katago-linglong

把 [KaTrain](https://github.com/sanderland/katrain) 打成如意玲珑（linglong）包。

KaTrain 是一个用 Python 语言开发的围棋 GUI，必须外挂一个 KataGo 引擎才能工作，本玲珑包内置 KataGo 二进制文件和模型文件，开箱即用。

## 环境要求

- `ll-builder`（`linglong-builder` 包）
- `ll-cli`（`linglong-bin` 包）
- 构建时需要网络：拉 base、GitHub 取源码、apt 装依赖、pip 装 Python 依赖

## 构建

每个架构自包含一个 `linglong.yaml`（和自己的 patches），在对应目录下构建：

| 架构 | 目录 | 构建环境 |
|------|------|----------|
| amd64 | `amd64/` | x86_64 机器或 CI |
| arm64 | `arm64/` | arm64 机器或 CI |
| loong64 | `loong64/` | 龙芯机器本地构建 |

```bash
# amd64（arm64 同理，进入对应目录）
cd amd64
ll-builder build -f linglong.yaml
ll-builder export --ref main:com.github.sanderland.katrain/1.20.0.0/amd64
```

产物：`com.github.sanderland.katrain_1.20.0.0_<arch>_main.uab`

安装并运行：

```bash
ll-cli install ./com.github.sanderland.katrain_1.20.0.0_<arch>_main.uab
ll-cli run com.github.sanderland.katrain
```

GitHub Actions 在打 tag 时自动构建 amd64 和 arm64 两个架构的 UAB（见
`.github/workflows/build.yml`）；loong64 在龙芯机器上本地构建。

## 注意事项

- **amd64 / arm64 的 Python 依赖直接用 PyPI 预编译轮子**（kivy、ffpyplayer 均有
  manylinux aarch64/x86_64 轮子），构建较快。
- **loongarch64 没有 kivy / ffpyplayer 的二进制轮子**，两者都要从 sdist 现场编译，kivy 单独就要约 15 分钟。编好的 wheel 会落到 `wheels/` 复用，让后续重建缩到 6 分钟左右；目录不存在时构建脚本会自动回退到源码编译。
- **内置引擎是纯 CPU 的 Eigen 后端**，不依赖任何 GPU 驱动，通用性好但速度慢。想换 OpenCL/CUDA 后端改 `cmake` 的 `-DUSE_BACKEND` 即可，但要同时补上对应的构建/运行时依赖。
