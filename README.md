# EasyTierServerTool —— EasyTier 服务器安装配置工具

一个**纯图形界面**的 EasyTier 服务器安装配置工具：不需要敲任何命令行，
即可完成 easytier-core 的下载/导入、参数配置、Windows 服务安装、启停管理、
状态监控与日志查看。**兼容 Windows Server 2008 R2**（基于 .NET Framework 3.5 开发）。

---

## 一、功能一览

| 功能 | 说明 |
| --- | --- |
| 一键下载/导入 | 内置按钮自动从 GitHub 下载官方 easytier-core（内置 9 个常用加速镜像可选，也支持自定义前缀），或导入本地 zip 安装包自动解压 |
| 图形化配置 | 网络名称/密钥、监听端口与协议（TCP/UDP/WebSocket/WireGuard）、IP 模式、安全模式、日志级别等全部图形化填写 |
| 实时命令预览 | 配置区实时显示将生成的 easytier-core 启动参数，一目了然 |
| 安装为 Windows 服务 | 一键注册系统服务（默认服务名 EasyTierServer），开机自启、进程崩溃 5 秒自动拉起 |
| 服务管理 | 安装 / 启动 / 停止 / 重启 / 卸载 / 状态刷新，全部按钮化 |
| 前台试运行 | 以当前配置在前台启动 easytier-core 调试，输出实时显示 |
| 日志查看 | 自动跟踪显示 easytier 日志（easytier.log / service.log），可一键打开日志目录 |
| 导出 TOML 配置 | 生成与界面参数一致的 easytier 配置文件，可用 `-c` 加载（格式已按 v2.6 实际导出结构验证） |
| 状态监控 | 底部状态栏实时显示服务状态、端口监听情况 |

## 二、环境要求

- **操作系统**：Windows 7 / Windows Server 2008 R2 及以上（x64/x86 均可）
- **运行时**：.NET Framework 3.5（Server 2008 R2 系统自带，无需安装）
- **权限**：安装/管理 Windows 服务需要管理员权限（程序启动时自动请求 UAC 提权）
- **EasyTier**：需准备 easytier-core.exe（v2.x，推荐 v2.6.4；官方下载包内含 easytier-core.exe、easytier-cli.exe、wintun.dll）

## 三、快速上手（3 步）

1. 将本工具 `EasyTierServerTool.exe` 复制到服务器任意目录（建议 `C:\easytier\`），双击运行；
2. 在「① 基本配置」页：
   - 点击 **下载最新版** 自动获取官方程序（或点击 **浏览...** 选择已有的 easytier-core.exe）；
   - 填写 **网络名称 / 网络密钥**（客户端接入凭证，建议点击“随机生成”）；
   - 确认 **监听端口**（默认 11010）与协议（建议 TCP + UDP）；
3. 切到「② 服务与运行」页，点击 **① 安装服务** → **② 启动服务**，完成。

客户端连接命令（在其他电脑上，管理员运行）：

```
easytier-core.exe -d --network-name 网络名称 --network-secret 网络密钥 -p tcp://服务器公网IP:11010
```

## 四、界面配置项说明

### ① 服务器程序
- **程序路径**：easytier-core.exe 位置；工具会自动探测同目录/`easytier\` 子目录。
- **下载最新版**：从 GitHub Releases 下载官方 Windows 包并解压到 `easytier\` 目录；GitHub 访问慢时可在「⑤ 日志」页填写镜像前缀（如 `https://ghfast.top/`）。
- **导入本地安装包**：离线环境下选择官方 zip 压缩包，自动解压并定位 easytier-core.exe。

### ② 网络设置
- **网络名称 / 网络密钥**：客户端必须使用完全相同的值才能加入本服务器组网。
- **实例名称 / 主机名**：本节点标识，默认取机器名。

### ③ 监听设置
- **监听端口**：默认 11010，客户端通过该端口连接（公网服务器需在防火墙/安全组放行）。
- **协议**：TCP、UDP（推荐都勾）、WebSocket（便于穿透复杂网络）、WireGuard。
- 下方实时显示将生成的 `-l` 监听参数。

### ④ 地址与安全
- **本机 IP 模式**：
  - `纯转发服务器（推荐）`：本机不创建虚拟网卡，只做数据转发与组网中转 —— 服务器首选；
  - `DHCP 自动分配 IP`：本机也作为网络节点获取虚拟 IP；
  - `手动指定 IP`：手动填写本机虚拟 IP（如 10.144.144.1）。
- **私有模式（推荐勾选）**：仅允许使用相同网络名称和密钥的节点经本服务器中转，防止被他人蹭网。
- **安全模式**：启用 Noise 安全握手；共享公网节点建议开启并点击“生成随机私钥”固定密钥。

### ⑤ 日志
- 控制台/文件日志级别（info 即可满足日常排障，问题排查可调 debug）。
- 文件日志自动写入 `logs\easytier.log`，按天滚动，最大 100MB×10 个。

### ⑥ Windows 服务
- **服务名**：默认 `EasyTierServer`，可自定义。
- **开机自动启动 / 崩溃自动重启**：默认开启；崩溃后 5 秒自动拉起，配合 SCM 的重启策略双重保障。
- 安装后服务由本工具托管，服务内部再拉起 easytier-core 并记录日志到 `logs\service.log`。

### ⑦ 运行与配置
- **前台试运行**：用当前配置前台启动 easytier-core，日志实时显示（便于调试，不注册服务）。
- **生成启动命令预览**：查看完整的 easytier-core 启动命令。
- **导出 TOML 配置**：生成 `easytier-server.toml`，可用 `easytier-core -c 该文件` 加载。

## 五、Server 2008 R2 特别注意事项

1. **在线下载需要 TLS 1.2**：.NET 3.5 默认不支持 TLS 1.2，本工具已尝试启用。若 2008 R2 下载失败，
   请安装补丁 **KB3140245** 并设置注册表（或直接换用“导入本地安装包”方式离线部署）。
2. **TUN 虚拟网卡驱动**：`纯转发服务器` 模式不创建虚拟网卡、**无需驱动**；若选择 DHCP/手动 IP 模式，
   官方包内的 `wintun.dll` 需与 easytier-core.exe 放在同一目录（官方 zip 默认就是同目录）。
3. **防火墙**：需放行监听端口（TCP/UDP 11010 等）；建议同时放行 RPC 管理端口 15888。
4. **杀毒软件**：个别杀软可能误报 easytier-core，请添加白名单。
5. **路径不要随意移动**：服务记录的是安装时的绝对路径，移动工具目录后需重新“卸载→安装”服务。

## 六、服务管理命令对照（工具按钮即执行以下命令）

| 操作 | 等价命令 |
| --- | --- |
| 安装服务 | `sc create EasyTierServer binPath= "\"路径\EasyTierServerTool.exe\" /service EasyTierServer" start= auto` |
| 崩溃重启策略 | `sc failure EasyTierServer reset= 86400 actions= restart/5000/restart/15000/restart/60000` |
| 启动 / 停止 / 删除 | `sc start EasyTierServer` / `sc stop EasyTierServer` / `sc delete EasyTierServer` |

## 七、无界面模式（高级/排障）

工具支持以下命令行参数（需管理员运行）：

```
EasyTierServerTool.exe /selftest      # 自检：打印当前配置与将生成的启动命令
EasyTierServerTool.exe /genconfig     # 按当前配置生成 easytier-server.toml
EasyTierServerTool.exe /service [名]  # 作为 Windows 服务运行（由“安装服务”内部使用）
```

## 八、从源码构建

环境：Windows + .NET Framework 3.5 或 4.x（本机已含 csc 编译器即可）。

```
双击 build.bat
```

产物：`EasyTierServerTool.exe`（.NET 3.5，兼容 Server 2008 R2，无需额外运行时）。

源码文件：`Program.cs`（入口/自检）、`MainForm.cs`（界面）、`Settings.cs`（配置存取）、
`CoreArgs.cs`（启动参数与 TOML 生成）、`ServiceManager.cs`（服务管理）、
`EasyTierService.cs`（服务宿主）、`Util.cs`（下载/解压/日志/端口探测）、`IconGen.cs`（图标生成）。

## 九、文件清单

```
EasyTierServerTool.exe    主程序（交付核心）
README.md                 本说明
build.bat                 一键构建脚本
app.manifest              管理员权限清单
icon.ico                  程序图标
*.cs                      源码
logs\                     运行后自动生成（easytier.log / service.log）
```

## 十、验证情况

- 已用官方 easytier-core v2.6.4 实测：生成的启动参数可正常启动服务器，TCP/UDP 11010 监听正常；
- 已实测导出 TOML 可被 `easytier-core -c` 正常加载；
- 界面三个标签页均渲染验证通过（含 125% 高分屏缩放验证）；
- 服务安装/启停的 `sc` 命令引号拼接已通过参数解析模拟验证；
- 服务安装的完整实机验证需在目标服务器（管理员权限）上进行。

## 十一、更新日志

### v1.2.2（配置加载修复 + 自动保存）

- **修复"关闭软件重开后配置丢失/不加载"的根本原因**：界面初始化阶段会把已加载的配置覆盖成默认值
  （`ApplySettingsToUi` 期间控件事件反向写回 `_s`），现已加防重入保护，配置在任何屏幕缩放、任何控件状态下都能正确回填；
- **配置变更即时自动保存**：任何修改（输入停顿 0.4 秒）即写入配置文件，即使程序被强杀、崩溃或异常退出，已改的配置也不会丢；
- 保存/加载失败不再静默：失败原因写入工具目录下 `settings-error.log`；
  若配置文件损坏，会自动把原文件备份为 `easytier-server-settings.xml.bad-时间戳` 后再用默认值启动，数据不丢失；
- 配置文件位置：exe 同目录下的 `easytier-server-settings.xml`（请从同一个解压目录运行，避免多份副本配置不互通）。

### v1.2.1（下载版本选择器）

- "下载最新版"弹窗升级为**版本下拉框**：内置 12 个常用稳定版本（v2.6.4 ~ v2.4.0）可直接选择；
- 同时**保留手动输入**任意版本号的方式（下拉框可直接键入）；
- 选择"最新稳定版（自动检测）"或留空时，自动查询 GitHub 获取最新发布版本，查询失败则回退到 v2.6.4。

### v1.2.0（镜像加速下拉选择）

- "下载镜像前缀"由纯输入框升级为**可下拉选择 + 可手动输入**的复合框，内置 9 个常用 GitHub 加速地址：
  `ghfast.top`、`gh-proxy.com`、`mirror.ghproxy.com`、`ghproxy.net`、`gh.llkk.cc`、`github.moeyy.xyz`、
  `ghps.cc`、`gh.ddlc.top`、`hub.gitmirror.com`；
- 也可直接在下拉框里输入任意自定义镜像前缀，留空则直连 GitHub；
- 说明：公共加速镜像可能随时失效，失效时换一个或直连即可。

### v1.1.1（再次修复说明文字遮挡）

- **"下载镜像前缀"标签**缩短为"下载镜像前缀(可选)"，完整示例放入鼠标悬停提示，不再压住输入框；
- **③ 监听行**整体左移重排，消除 125% 缩放下"WireGuard"与"WG端口"标签、WG 端口数字框的挤压重叠；
- **④ 地址行**压缩"本机IP 下拉框"宽度，"多线程运行"不再超出分组边界；"安全模式"与"固定私钥"、私钥输入框之间增加间距消除重叠；
- **tab2 服务说明**缩短并加宽，不再截断"安装需管理员权限"；
- **"导出TOML配置"按钮**加宽，文字不再被右侧裁切；
- **⑧ 客户端连接说明框**加高并禁止自动换行，五行说明完整显示、不再出现滚动条遮住末行；
- 所有固定宽度提示标签加高，消除高 DPI 下字形上下被裁。

### v1.1.0（界面显示与流畅度修复）

- **修复高分屏显示不全/重叠**：控件坐标与尺寸按系统 DPI 等比缩放（支持 100%/125%/150% 等缩放比例），
  "本机IP 下拉框""私有模式/安全模式""日志说明"等文字不再截断、重叠；
- **修复拖动窗口卡顿**：服务状态与端口探测改为后台线程异步执行（原先每 2-3 秒在主线程同步探测
  端口，最长可阻塞 1.5 秒）；日志框关闭自动换行、降低最大缓存，双缓冲重绘；
- 状态刷新频率从 3 秒优化为 2 秒。

> 本工具仅负责服务器（共享节点）的安装配置；组网协议为 EasyTier 官方实现，
> 项目主页：https://github.com/EasyTier/EasyTier （文档：https://easytier.cn）
