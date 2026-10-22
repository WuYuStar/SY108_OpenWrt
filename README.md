# SY108 (UDX710) 5G通信壳 OpenWrt 自定义移植与云端构建

本项目基于 [jason-liu-4980/sy108-mainline](https://github.com/jason-liu-4980/sy108-mainline) 以及相关内核、U-Boot 仓库构建。利用 GitHub Actions 自动化流程，无需本地极其高昂的算力，即可在云端一键编译出包含“不死固件”回退机制的 OpenWrt 固件。

## 🚨 核心准备工作：提取原厂数据 (必做)

刷机前，**必须**从你的物理设备中提取原厂分区，尤其是用于底层引导封装的 `splloader.bin` 和 `sml.bin`，它们是云端封装 `uboot.bin` 的“钥匙”。

### 1. 寻找底层刷机工具
你需要以下几个工具和文件（请自行搜索下载并放在同一文件夹内）：
- **`spd_dump`**：展锐底层刷机工具（推荐从 [TomKing062 的 GitHub](https://github.com/TomKing062/spreadtrum_flash) 下载）。
- **`custom_exec_no_verify_3f28.bin`**：绕过 FDL 签名校验的补丁（推荐从 [dann2333/udx710-mifi-info](https://github.com/dann2333/udx710-mifi-info/tree/main/r106) 的 `r106` 目录下载）。
- **`fdl1`** 和 **`fdl2`**：第一阶段和第二阶段引导文件（同上，在 `r106` 目录下载，**不要加后缀**）。

### 2. 进入 FDL 刷机模式
1. 设备完全断电，按住设备的 RESET 键（或短接刷机触点）。
2. 将 USB 线插入电脑，等待几秒后松开 RESET。设备管理器应识别到 `SPRD U2S Diag (COMx)`。
3. 如果使用 `spd_dump` 无法通信，请使用 **Zadig** 工具将驱动替换为 `libusb-win32`。

### 3. 提取/备份原厂文件 (使用 `spd_dump`)
打开命令行终端（确保与下载的工具在同一目录），执行以下命令进入底层 `FDL2>` 模式：
```cmd
spd_dump.exe --wait 300 exec_addr 0x3f28 fdl ./fdl1 0x28007000 fdl ./fdl2 0x9efffe00 exec
