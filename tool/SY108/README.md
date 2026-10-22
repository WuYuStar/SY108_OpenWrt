### 📂 前提条件

- **工作目录**：`C:\sy108_tool`
    
- **所需文件**：`spd_dump.exe`、`libusb-1.0.dll`、`custom_exec_no_verify_3f28.bin`、`fdl1`（无后缀）、`fdl2`（无后缀）
- **spd_dump.exe 的下载来源**：https://nightly.link/TomKing062/action_spd_dump_it/workflows/build/main
- **spd_dump版本说明**：spd_dump_it_oldpath_233_7a50382_x64_Release 旧版、spd_dump_it_main_249_2fe0b5b_x64_Release 新版
  

### 🔌 第一步：连接设备（进入 FDL2 模式）
注：在C盘新建目录sy108_tool，将所有下载的文件放入此目录，请勿嵌套目录，全部文件需要在同级，备份的文件都会存入这个目录。

1. 拔掉设备所有线，让它完全断电。
    
2. 按住通信壳的 RESET 键（或短接刷机触点），插上 USB 线连接电脑，然后松开按键。
    
3. 打开 `cmd` 命令行，进入文件夹：
    
```
    cmd
```
    
```
    cd C:\sy108_tool
```
    
4. **执行握手命令**（看到 `FDL2 >` 即代表成功）： 

```
    spd_dump.exe --wait 300 exec_addr 0x3f28 fdl ./fdl1 0x28007000 fdl ./fdl2 0x9efffe00 exec
```

### 💾 第二步：全量提取分区（备份命令）

**⚠️ 警告：** 进入 `FDL2 >` 后，一条一条依次输入下面的命令，**每敲完一条等它跑到 100% 并出现 `Read Part Done` 后**，再敲下一条。

**1. 核心引导（保命用）**

```
read_part splloader 0 256k splloader.bin
read_part sml 0 256k sml.bin
read_part uboot 0 1M uboot.bin
```

**2. 系统内核（救砖用）**

```
read_part boot 0 12M boot.bin
read_part recovery 0 16M recovery.bin
```

**3. 基带与射频参数（保信号用）**

```
read_part nr_fixnv1 0 8M nr_fixnv1.bin
read_part nr_fixnv2 0 8M nr_fixnv2.bin
read_part prodnv 0 4M prodnv.bin
read_part nr_runtimenv1 0 10M nr_runtimenv1.bin
read_part nr_runtimenv2 0 10M nr_runtimenv2.bin
read_part persist 0 2M persist.bin
```

**4. 语音与状态（补全用）**

```
read_part nr_agdsp 0 6M nr_agdsp.bin
```

---

### 🚑 第三步：如何用备份救砖？（写入命令）

如果未来某天刷机失败，或者想恢复原厂系统，重复第一步进入 `FDL2 >`，然后执行以下 `w`（write）命令（假设原厂 `system.bin` 在你当前的文件夹里）：

```
w splloader splloader.bin
w sml sml.bin
w uboot uboot.bin
w boot boot.bin
w recovery recovery.bin
w nr_fixnv1 nr_fixnv1.bin
w nr_fixnv2 nr_fixnv2.bin
w prodnv prodnv.bin
w system system.bin
reset
```
