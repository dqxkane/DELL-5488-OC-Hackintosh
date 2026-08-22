# DELL-5488-OC-Hackintosh


# 电脑配置
| 规格     | 详细信息                                     |
| -------- | ---------------------------------------- |
| 电脑型号 | DELL-inspiron-5488             |
| 处理器   | 英特尔 酷睿 i5-8265u处理器             |
| 内存     | 玖合 32GBx2  DDR4 3200MHz（u板只支持到2400）                 |
| 硬盘     | 东芝 NVMe固态硬盘 XG6 256GB                  |
| 硬盘     | 东芝 SATA硬盘  500G                  |
| 集成显卡 | Intel GMA UHD 620                          |
| 独立显卡 | 无                       |
| 声卡     | Realtek ALC236（layout-id 68 + ComboJack 耳麦）                   |
| 无线网卡 | 更换为博通 BCM94360cs2 白苹果拆机免驱卡                   |
| 有线网卡 | Realtek PCIe Ethernet Controller Driver                  |
| 指纹    | Goodix Fingerprint Sensor Driver 无法驱动，通过定制usb屏蔽                        |
| 摄像头  | UVC Camera 功能正常                        |
| 读卡器  |Realtek Memory Card Reader Driver      通过usb定制功能正常                  |


# 声卡 / 二合一耳机孔（耳麦）
- 声卡为 Realtek **ALC236**，AppleALC 使用 **layout-id 68**（`ALC236 for Dell, use with ComboJack`）。
- `DeviceProperties -> PciRoot(0x0)/Pci(0x1f,0x3)` 下：
  - `layout-id` = 68
  - `alc-verbs` = `01000000`（启用 AppleALC 的 verb 接口，供 ComboJack 用，等价 boot-arg `alcverbs=1`）
- **二合一耳机孔（耳麦麦克风）**需要 ComboJack 用户态守护进程（插耳机时弹窗选择「耳机 / 耳麦」）。

## 安装 ComboJack
1. 终端执行（需 sudo）：
   ```sh
   sudo ./ComboJack/install.command
   ```
2. 重启。
3. 插入耳机后弹出选择框，选 **Headset（耳麦）** 后耳麦麦克风可用。

> 卸载：`sudo ./ComboJack/uninstall.command`

