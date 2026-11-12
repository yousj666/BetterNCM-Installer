<div align="center"><image width="140em" src="https://user-images.githubusercontent.com/66859419/183120498-1dede5b4-0666-4891-b95f-c3a812b3f12f.png" /></div>
<h1 align="center">BetterNCM Installer II</h1>
<h3 align="center">PC版网易云客户端插件管理器</h3>

一键安装 [BetterNCM V2](https://github.com/MicroCBer/BetterNCM)

**网易云版本必须 `>=2.10.2`**

![image](https://user-images.githubusercontent.com/66859419/204120743-a528b624-d016-4f6f-a0d7-e769cdd2dd74.png)

![Installer](https://user-images.githubusercontent.com/66859419/210129835-11ceea16-f5dd-43b7-ba83-625a3c4d920e.png)

# 手动安装流程
1. 从 BetterNCM 仓库下载最新版 `BetterNCMII.dll`
2. 打开网易云音乐安装目录，将上一步下载的 `BetterNCMII.dll` 复制进去并改名为 `msimg32.dll`

# 插件库
已在 BetterNCM 内置

# 构建
```bash
cargo +nightly build --release -Z build-std=core,alloc,std,panic_abort -Z build-std-features=panic_immediate_abort --target i686-pc-windows-msvc
```

# yousj-plus 增强版（本 Fork 的 yousj-plus 分支）

在上游安装器基础上的增强（改动都在 `src/main.rs`，数据源清单见 `installer_sources.json`）：

## 读上游代码得出的结论

- 安装器**运行时并不读 `latest_releases.json`**（全仓库无引用，那是老版安装器遗留文件）；
  它启动/切换通道时，从 gitcode 的 `bncm-data-pack2` 拉 `betterncm3.json` 版本表：
  按通道取 `versions`（正式）或 `test`（测试）对象，键是网易云版本的 semver 要求，
  值含 `version` / `url_x86` / `url_x64`，命中后下载对应 DLL。
- 安装动作：下载 DLL → 装 VC 运行时 → 结束网易云进程 →
  把 DLL 拷进网易云安装目录并改名为 `msimg32.dll` → 重启网易云。
- 上游安装器**本身不装任何插件**（插件库靠 BetterNCM 本体内置的 PluginMarket）；
  插件文件应放 `<BetterNCM 数据目录>/plugins/*.plugin`，数据目录规则与本体一致：
  环境变量 `BETTERNCM_PROFILE`，否则 `C:\betterncm`。

## 本分支的增强

1. **可切换安装源**：界面新增勾选「增强版源(yousj-plus)」。
   - 不勾（默认）= 上游官方源，行为与上游完全一致。
   - 勾上 = 版本表改拉本 Fork 配套的
     `yousj666/chromatic` 仓库 `yousj-plus` 分支里的 `installer/betterncm3-yousj.json`
     （与上游同格式），可独立更新、不依赖上游数据包。
     该表目前的 DLL 地址指向上游 1.3.4 官方 Release——因为 yousj-plus
     没有改本体 C++ 代码，增强都在插件层；以后 Fork 自行发布 DLL 时只换表里的 URL。
2. **预装 MV 下载插件**：用增强版源点「安装」或「重装/更新」时，
   本体装好后会把安装器内嵌的 `assets/MVDownloader.plugin`
   （`include_bytes!` 打进 exe）写到 `<数据目录>/plugins/MVDownloader.plugin`，
   客户端启动即自动加载。插件功能：网易云官方搜索 + MV 最高画质下载为 MP4，
   源码在 `yousj666/chromatic` 的 `yousj-plus` 分支 `builtin-plugins/MVDownloader/`。
   预装失败不影响本体安装，界面提示里会说明，并给出手动放置的方法。
3. 版本号标记为 `1.1.5-yousj.1`，与上游 1.1.4 区分。

## 构建（与上游一致，Windows）

```bash
cargo +nightly build --release -Z build-std=core,alloc,std,panic_abort -Z build-std-features=panic_immediate_abort --target i686-pc-windows-msvc
```

需要 nightly 工具链与 `-Z build-std`；代码还用了 `#![feature(fs_try_exists)]` 等
nightly 特性，并且依赖 Windows 专有 crate（winreg、`std::os::windows`），
**在 Linux 沙箱里无法完成 `cargo check`**（缺 nightly + Windows 目标），
本分支的改动只做了源码级核对，未在真机/Windows 工具链验证编译，见仓库提交说明。
