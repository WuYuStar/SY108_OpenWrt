# GitHub Actions 云端编译流水线

## 说明
本目录存放一键云端编译脚本 `build.yml`。

## 工作流逻辑
1. 释放磁盘空间并安装底层依赖（QEMU、编译工具链等）。
2. **动态拉取**另外两个代码仓库（`linux-udx710` 内核源码 和 `sy108-uboot-spl` 引导源码）。
3. 自动打补丁绕过原脚本的死板哈希校验。
4. 依次编译主系统 (`firmware.py main`) 和恢复系统 (`firmware.py recovery`)。
5. 调用 U-Boot 封装脚本，读取 `privates` 目录下的原厂签名，生成最终带不死机制的 `uboot.bin`。
6. 打包所有生成的固件（`.bin`）供下载。
