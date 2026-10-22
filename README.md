# SY108 (UDX710) 5G通信壳 OpenWrt 自定义移植与云端构建

本项目基于开源社区的 UDX710 主线适配方案，利用 GitHub Actions 自动化流水线，无需在本地配置复杂的 OpenWrt 编译环境，即可在云端一键编译出包含“不死固件”看门狗回退机制的 OpenWrt 固件与自定义 U-Boot。

## 核心准备工作：提取原厂数据 (必做)

在云端编译前，必须从你的物理设备中提取原厂分区数据。其中最关键的是 **splloader.bin** 和 **sml.bin**，它们是云端封装 U-Boot 引导程序的必需材料。此外，备份完整的原厂分区可以在设备变砖时救命。

### 1. 寻找底层刷机工具

你需要以下几个工具和文件，请自行搜索下载并放在 Windows 的同一个文件夹内：

- **spd_dump.exe** 及配套的 **libusb-1.0.dll**：展锐底层刷机工具。推荐从 TomKing062 的 GitHub 仓库（spreadtrum_flash）下载 Release 版本。
- **custom_exec_no_verify_3f28.bin**：绕过 FDL 签名校验的漏洞利用文件。推荐从 dann2333 的 GitHub 仓库（udx710-mifi-info）的 r106 目录中下载。
- **fdl1** 和 **fdl2**：第一阶段和第二阶段底层引导文件。同样在 r106 目录中下载。注意：下载后不要加后缀，直接保持无后缀状态。
- **Zadig 工具**：如果在 Win11 下 spd_dump 报错，需使用 Zadig 将设备的驱动程序替换为 libusb-win32。

### 2. 进入 FDL 刷机模式

- 设备完全断电，按住设备上的 RESET 键，或者使用镊子短接主板上的刷机触点。
- 将 USB 线插入电脑，等待几秒后松开 RESET 键。
- 如果在设备管理器中看到 SPRD U2S Diag (COMx) 端口，说明驱动正常，设备已进入刷机模式。

### 3. 提取与备份原厂文件

打开 Windows 命令行（CMD），确保当前路径与你存放工具的文件夹一致。执行以下命令进入底层 FDL2 交互模式：

```cmd
spd_dump.exe --wait 300 exec_addr 0x3f28 fdl ./fdl1 0x28007000 fdl ./fdl2 0x9efffe00 exec
看到 FDL2 > 提示符后，说明已经成功接管设备。请依次逐行复制以下命令并回车执行，等待前一条读取完成后，再执行下一条。

提取云端编译所需的底层引导与签名数据：

text
read_part splloader 0 256k splloader.bin
text
read_part sml 0 256k sml.bin
提取原厂 U-Boot 和内核，用于日后变砖时救砖：

text
read_part uboot 0 1M uboot.bin
text
read_part boot 0 12M boot.bin
提取基带射频校准参数，丢失这些数据会导致设备无信号：

text
read_part nr_fixnv1 0 8M nr_fixnv1.bin
text
read_part nr_fixnv2 0 8M nr_fixnv2.bin
text
read_part prodnv 0 4M prodnv.bin
提取完成后，输入 reset 回车或直接拔线退出设备。

4. 将文件放入仓库
将刚刚提取出来的 splloader.bin 和 sml.bin 上传到本 GitHub 仓库的 privates/ 文件夹中。请妥善在本地保存所有提取出来的文件，以防万一。

云端一键编译固件
本仓库集成了 GitHub Actions，你不需要在本地安装任何 OpenWrt 编译环境。

在本仓库页面，点击顶部的 Actions 标签。

在左侧列表点击 Build SY108 OpenWrt Firmware。

右侧点击 Run workflow。

你可以选择是否自动创建 GitHub Release 发布固件。如果勾选，编译成功后会自动发布到 Releases 页面供长期下载；如果不勾选，则只作为临时构建产物在 Actions 页面供下载。

点击绿色的 Run workflow 按钮。云端将自动执行拉取源码、编译内核、编译主系统、编译 Recovery、封装 U-Boot 等全套流程，约需 1 至 2 小时。

编译完成后，在 Actions 运行详情页底部下载 SY108-OpenWrt-Firmware 压缩包，或者在 Releases 页面下载。

刷入新固件
云端编译并解压后，你将获得 4 个文件：uboot.bin、sysupgrade.bin、boot.bin、recovery_boot.bin。

再次让设备按照前述步骤进入 FDL2 模式。

依次逐行输入以下刷写命令，将固件写入设备。

写入自定义 U-Boot：

text
w uboot uboot.bin
写入 OpenWrt 主系统：

text
w system sysupgrade.bin
写入恢复系统：

text
w recovery recovery_boot.bin
重启设备：

text
reset
注意：具体写入的分区名称，请根据原厂分区表确认。

免责声明
刷机有风险，本项目仅供技术交流与折腾，因操作不当导致的硬件损坏、数据丢失、设备变砖等后果需自行承担。
