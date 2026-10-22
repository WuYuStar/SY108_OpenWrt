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
看到 FDL2 > 后，依次输入以下命令，将最关键的底层文件提取出来：

text
# 提取底层引导和签名数据（云端编译封装 U-Boot 必需）
read_part splloader 0 256k splloader.bin
read_part sml 0 256k sml.bin

# 提取原厂 U-Boot 和内核（用于变砖后的救砖）
read_part uboot 0 1M uboot.bin
read_part boot 0 12M boot.bin

# 提取基带射频校准参数（极其重要，丢失会导致无信号）
read_part nr_fixnv1 0 8M nr_fixnv1.bin
read_part nr_fixnv2 0 8M nr_fixnv2.bin
read_part prodnv 0 4M prodnv.bin
4. 将文件放入仓库
将刚刚提取出来的 splloader.bin 和 sml.bin 上传到本仓库的 privates/ 文件夹中。(请妥善在本地保存所有提取出来的文件，以防万一)。

☁️ 云端一键编译固件
本仓库集成了 GitHub Actions，你不需要在本地安装复杂的 OpenWrt 编译环境。

在本仓库页面，点击顶部的 Actions 标签。

在左侧列表点击 Build SY108 OpenWrt Firmware。

右侧点击 Run workflow。

Use workflow from 保持默认（main）。

是否自动创建 GitHub Release 发布固件？：如果你勾选，编译成功后会自动发布到 Releases 页面供长期下载；如果不勾选，则只作为临时构建产物供下载。

点击绿色的 Run workflow 按钮，云端将自动执行拉取源码、编译内核、编译主系统、编译 Recovery、封装 U-Boot 等全套流程（约需 1~2 小时）。

编译完成后，在 Actions 运行详情页底部下载 SY108-OpenWrt-Firmware.zip（或在 Releases 页面下载）。

💾 刷入新固件 (FDL 刷机)
云端编译并解压后，你将获得 4 个文件：

uboot.bin：带看门狗回退机制的自定义 U-Boot。

sysupgrade.bin：OpenWrt 主系统。

boot.bin (main)：主系统的启动 Capsule。

boot.bin (recovery)：恢复系统 Capsule。

再次让设备进入 FDL2> 模式。

按照以下顺序输入刷写命令（根据你的实际文件名调整）：

text
w uboot uboot.bin
w system sysupgrade.bin
w recovery recovery_boot.bin
reset
(注：具体写入的 system 和 recovery 分区名称，请根据分区表确认)。

⚠️ 免责声明：刷机有风险，本项目仅供技术交流与折腾，因操作不当导致的硬件损坏、数据丢失、设备变砖等后果需自行承担。
