# REA 需要下载的软件说明（中文）

> REA 本体只是调度层，自己不带分析引擎。工具能不能用，取决于下面这些软件装没装。
> 分三档：必需的只有两样，其余按用哪条线选装。标注「实测组合」的是 2026-11-21 在
> Linux x64 + JDK 25 + Ghidra 12.2 + jadx-headless-mcp 0.7.1 上跑通的组合；
> 其余按官方文档整理。凡走第三方加速下的大文件，下完必须对官方 SHA-256，对不上就删。

## 一、必需（不装 REA 起不来）

| 软件           | 作用                                          | 版本要求                      | 官方地址                                                     | 国内镜像                                                                                                       |
| -------------- | --------------------------------------------- | ----------------------------- | ------------------------------------------------------------ | -------------------------------------------------------------------------------------------------------------- |
| Node.js        | 跑 REA 本体（rea-agents 是 npm 包）           | 22.19+ 或 24+（实测 24.22.1） | nodejs.org                                                   | npmmirror `https://npmmirror.com/mirrors/node/`，或清华 `https://mirrors.tuna.tsinghua.edu.cn/nodejs-release/` |
| rea-agents     | REA 本体                                      | 实测 6.10.1                   | npm 包 `rea-agents`，只认作者 morluto 的仓库与官网 rea.tools | 先换源再装：`npm config set registry https://registry.npmmirror.com`，然后 `npx rea-agents setup`              |
| ps（仅 Linux） | Ghidra、JADX、固件、JS 线启动时做进程归属检查 | procps 兼容即可               | 系统自带或包管理器装 procps                                  | apt 源换清华后直接装                                                                                           |

## 二、原生二进制线（Ghidra 通道，实测组合）

| 软件           | 作用                                                                                        | 版本要求                                                                                        | 官方地址                                                                                                        | 国内镜像                                                                                     |
| -------------- | ------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------- |
| JDK（Temurin） | Ghidra 的运行环境；版本下限由 Ghidra 安装目录里 `application.java.min` 声明，REA 按声明校验 | Ghidra 12.2 声明 **JDK 25 或更新**、无上限；JDK 21 会被拒绝（实测）                             | adoptium.net                                                                                                    | 清华 TUNA Adoptium 镜像 `https://mirrors.tuna.tsinghua.edu.cn/Adoptium/`，选 25              |
| Ghidra         | 反汇编与反编译主力引擎                                                                      | REA 接受 12.x 且不低于 12.2（实测 12.2，包名 `ghidra_12.2_PUBLIC_20261016.zip`，解压约 879 MB） | GitHub `NationalSecurityAgency/ghidra` 的 Releases：`https://github.com/NationalSecurityAgency/ghidra/releases` | **无官方国内镜像**。官方直连或 GitHub 文件加速下载，下完对官方 release 页的 SHA-256 逐字核对 |

环境变量（装完要设，REA 靠这几个找引擎）：

```bash
export JAVA_HOME=<JDK 安装目录>
export GHIDRA_INSTALL_DIR=<Ghidra 解压目录>
export REA_JADX_MCP_JAR=<jadx-headless-mcp.jar 的绝对路径>
```

## 三、APK / Android 线

| 软件                  | 作用                         | 版本要求                                                                                        | 官方地址                                   | 国内镜像                                                                                                                                                                                          |
| --------------------- | ---------------------------- | ----------------------------------------------------------------------------------------------- | ------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| jadx-headless-mcp     | REA 调 JADX 的桥（单个 jar） | 实测 0.7.1，约 41 MB；`rea doctor` 的 JADX 检查会给出该版本的官方下载地址与 SHA-256，照它下即可 | 见 doctor 输出                             | 无官方镜像：加速下载 + 按 doctor 给的哈希验                                                                                                                                                       |
| apktool               | APK 资源与 smali 拆包        | 最新正式版即可                                                                                  | apktool.org（指向 GitHub Releases）        | 同上                                                                                                                                                                                              |
| platform-tools（adb） | 连真机与模拟器               | 最新版即可                                                                                      | developer.android.com 的 platform-tools 页 | 腾讯云 AndroidSDK 镜像 `https://mirrors.cloud.tencent.com/AndroidSDK/`，搜 `platform-tools_r` 开头的最新 zip 直下                                                                                 |
| frida-tools           | 动态插桩（frida 线工具）     | 与真机 frida-server 版本必须完全一致                                                            | pip 安装                                   | 先设清华 PyPI 源：`pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple`，再 `pip install frida-tools`。frida-server 本体去 frida 官网 GitHub Releases 下对应架构版，放进真机 |

## 四、抓包线

| 软件                   | 作用            | 官方地址      | 国内镜像                                               |
| ---------------------- | --------------- | ------------- | ------------------------------------------------------ |
| Wireshark（含 tshark） | pcap 分析线引擎 | wireshark.org | Windows/Mac 官方下；Linux 包管理器装，apt 源换清华即可 |

## 五、商业引擎（可选，REA 也能调，没装对应工具就报不可用）

| 软件    | 说明                                                          |
| ------- | ------------------------------------------------------------- |
| IDA Pro | 商业授权，REA 里是只读分析通道。官网 hex-rays.com，无国内镜像 |
| Hopper  | 商业授权，REA 的 setup 经你批准后可以代装。官网 hopperapp.com |
| Kuna    | 另一个原生分析提供方，按官方文档配置                          |
| JEB     | 商业 Android 反编译器，REA 有对应分析通道，按需再看           |

## 装完怎么验

跑 `rea doctor`（可加 `--provider ghidra`、`--provider jadx` 等单查一家）：它会逐个引擎报
健康或缺失，并给出修复指引。本机实测配齐的状态是 Ghidra ✓、JADX ✓，其余没装的会明确
报不可用，不会假装能用。缺哪个就按上面对应行补哪个，不用一次装全。
