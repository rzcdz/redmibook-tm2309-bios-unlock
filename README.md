# RedmiBook Pro 16 2024 BIOS 隐藏菜单解锁

> 通过 SREP 在内存中临时解除 BIOS 选项压制，解锁隐藏的 `Advanced` 菜单与 `电源菜单`，
> 用于修改 `State After G3`（通电自动开机）等厂商隐藏设置。
>
> Unlock hidden BIOS setup pages on Xiaomi RedmiBook laptops using SREP
> (SmokelessRuntimeEFIPatcher) in-memory patching. No flash write, no brick risk.

---

## ✅ 测试成功机型

| 项目 | 值 |
|---|---|
| **机型** | **Xiaomi RedmiBook Pro 16 2024** |
| **主板型号** | `TM2309`（SKU `TM2309-54230`）|
| **BIOS 版本** | **`RMAMT6B0P0B0B`** |
| **CPU** | Intel Core Ultra 5 125H |
| **实测结果** | ✅ **成功** —— `State After G3` 改为 `S0 State` 后，插上电源自动开机 |

> ⚠️ **本配置仅适用于上述机型与 BIOS 版本。**
> 换到其他型号或 BIOS 版本：轻则无效（找不到匹配字节，安全），重则匹配到错误位置导致界面错乱或卡死。
> 若需用于其他机器，必须重新分析该机器的固件，**不能直接套用本配置**。

---

## 一、这个工具能做什么

通过 SREP（SmokelessRuntimeEFIPatcher）在内存中解除 BIOS 界面的选项压制，
让你看到并修改厂商隐藏的设置项。

**已实测可解锁：**

- **电源菜单（Power）** —— 原本完全隐藏，含 11 项唤醒设置
- **Advanced 菜单** —— 原本无入口，含 `PCH-IO Configuration` 等
- `PCH-IO Configuration` → **`State After G3`** —— 通电自动开机的开关

<img width="1609" height="1279" alt="mmexport1795186258091" src="https://github.com/user-attachments/assets/ef27e446-2e43-4d91-a856-c4d341919cab" />



**典型用途：**

把 `State After G3` 从 `S5 State` 改为 `S0 State`，实现「插上电源自动开机」，
配合智能插座即可实现远程开机。

**重要特性：**

- 补丁**只作用于内存**，不写入 SPI Flash，**重启后自动消失**
- 因此**不会变砖**，最坏情况是卡住，强制重启即可恢复
- 但你在 BIOS 里修改并保存的设置会**永久生效**

---

## 二、准备工作

需要的东西：

1. 一个 U 盘（8 GB 足够）
2. 本仓库的文件
3. 能进 BIOS 的电脑

开始前请确认：

- [ ] 笔记本电量在 30% 以上（避免操作中没电）
- [ ] 关闭所有正在运行的程序，保存工作
- [ ] 记住你的 Windows 登录密码（万一需要重启验证）

---

## 三、第一步：关闭安全启动（Secure Boot）

### 为什么要关

UEFI Shell（`BOOTX64.EFI`）不是微软签名的程序。安全启动开启时，
固件会拒绝运行未签名程序，U 盘根本引导不起来。

### 关闭方法

1. **关机**（开始菜单 → 电源 → 关机）

2. 开机，在出现小米 Logo 前**连续按 `F2`**，进入 BIOS 设置

3. 用方向键移动到左侧的**【安全菜单】**（盾牌图标）

4. 找到 **Secure Boot / 安全启动** 选项

   - 如果它已经显示【关闭】/【Disabled】，直接跳到第 7 步
   - 如果它是**灰色不可选**，继续第 5 步

5. 在同一页面找到**【设置管理员密码】/ Set Supervisor Password**
   设置一个你能记住的密码（可以很简单，比如 `12345678`）

   设置密码后，原本灰色不可选的项目会解锁。

6. 回到 Secure Boot，改为**【关闭】/【Disabled】**

7. 按 **`F10`** 保存并退出，机器自动重启

### 风险提示

- **BIOS 密码一定要记住。** 忘记了只能拆机清 CMOS，很麻烦。
- 如果 Windows 开启了 **BitLocker 加密**，关闭安全启动可能触发恢复密钥要求。
  操作前请先确认 BitLocker 状态（`Get-BitLockerVolume`），
  或在「设置 → 隐私和安全性 → 设备加密」中确认是否开启。
- 如果以后想恢复安全启动，用同样的方法改回【开启】即可。
  设了密码的话，可以再进安全菜单把密码清空（新密码留空）。

---

## 四、第二步：制作 U 盘

### 方法一：全新 U 盘

1. 把 U 盘插入电脑

2. 打开「此电脑」，右键 U 盘 → **格式化**
   - 文件系统选 **FAT32**
   - 分配单元大小用默认
   - 勾选「快速格式化」

   > **exFAT 和 NTFS 都不行**，UEFI 固件不认这两个格式。
   > 如果 U 盘大于 32 GB，Windows 自带工具可能没有 FAT32 选项，
   > 这时用 [Rufus](https://rufus.ie/) 格式化。

3. 把本仓库所有文件解压/复制到 **U 盘根目录**

   正确结构：

   ```
   U盘根目录\
     ├─ EFI\
     │    └─ BOOT\
     │         └─ BOOTX64.EFI
     ├─ SREP.efi
     ├─ SREP_Config.cfg
     └─ startup.nsh
   ```

   > **常见错误**：把文件放进了一个子文件夹，比如 `U盘\BIOS解锁\EFI\...`
   > 这样固件找不到引导文件，U 盘无法启动。

### 方法二：用现成的 U 盘

直接按上面的结构复制那 4 个文件（`EFI` 目录 + 3 个文件）即可，
不需要格式化，但 U 盘必须是 FAT32 格式。

---

## 五、第三步：从 U 盘启动

1. **关机**（注意是关机，不是重启）

2. 插上做好的 U 盘

3. 开机，在小米 Logo 出现前**连续按 `F12`**

4. 出现启动菜单后，选择你的 U 盘
   通常显示为 `UEFI: 你的U盘品牌名` 这类字样

5. 如果一切正常，屏幕会进入黑底白字的命令行界面（UEFI Shell），
   并**自动开始运行补丁程序**。

6. 等待它运行，你会看到类似输出：

   ```
   Patched 1427 statements
   Success! Main Setup DXE found and patched!
   PackageList ... Patched 16 statements
   ...
   ```

   > 中间可能出现**俄文**（`Выполняется аргумент Exec` 等），
   > 这是 SREP 的界面语言，不影响使用。

7. 补丁完成后，屏幕会切换到 **BIOS 设置界面**

---

## 六、第四步：找到并修改设置

进入 BIOS 后，看左侧菜单。

**正常情况** —— 左侧有 5 项：

```
主菜单 / 安全菜单 / 电源菜单 / 启动菜单 / 退出菜单
```

**解锁后** —— 左侧仍是 5 项，但「启动菜单」的位置变成了 **Advanced**：

```
主菜单 / 安全菜单 / 电源菜单 / Advanced / 退出菜单
                                ↑ 就是这里
```

### 操作路径

```
Advanced
  └─ PCH-IO Configuration        ← 进入 Advanced 后第一项
       └─ 向下翻找 State After G3
            ├─ S0 State           ← 选这个（通电即开机）
            └─ S5 State           ← 默认是这个（通电不开机）
```

### 具体步骤

1. 选中左侧 **【Advanced】**，按回车进入
2. 选中 **【PCH-IO Configuration】**，按回车进入
3. 用方向键向下翻，找到 **【State After G3】**
   说明文字：*Specify what state to go to when power is re-applied after a power failure (G3 state)*
4. 按回车或用 `F5`/`F6` 把它改成 **【S0 State】**
5. 按 **`F10`** 保存并退出
6. 机器重启

---

## 七、第五步：验证是否成功

### 实际测试（推荐）

1. 完全关机
2. 拔掉充电器
3. 等 10 秒
4. **插上充电器**
5. 观察：**如果电脑自动开机，说明成功了**

### 查看变量值（技术验证）

需要管理员权限，在 PowerShell 中读取 `PchSetup` 变量：

- 变量名 `PchSetup`
- GUID `4570B7F1-ADE8-4943-8DC3-406472842384`
- 偏移 **`0x21`**
- **`0x00` = S0 State（通电开机）** ✅
- `0x01` = S5 State（通电不开机）

---

## 八、第六步：恢复安全启动（建议）

操作完成后，建议把安全启动重新打开，保持系统安全性。

1. 关机，开机按 `F2` 进 BIOS
2. 进【安全菜单】
3. 把 Secure Boot 改回【开启】/【Enabled】
4. `F10` 保存退出

如果之前为了解锁选项设置了 BIOS 密码，可以顺手清掉：
进【安全菜单】→ 设置管理员密码 → 新密码留空 → 确认

---

## 九、注意事项（重点）

### 安全方面

1. **这个 U 盘插上开机就会自动运行补丁并进入解锁的 BIOS 界面。**
   平时不用时，把 `startup.nsh` 改名为 `startup.nsh.off`，
   这样插上只会进普通 Shell，不会自动跑补丁。需要用时再改回来。

2. **不要在解锁后的 BIOS 里乱改不认识的选项。**
   特别是以下这些，改了可能开不了机：
   - 电压相关（Voltage）
   - 内存时序（tCL / tRCD / tRP 等）
   - 频率设置（Frequency）
   - 启动相关（Boot / CSM）
   - 安全相关（TPM / PTT / SGX）

   **不确定的选项一律不动。** 只看你需要的那个。

3. 补丁不会写 flash，所以**不会变砖**。但 BIOS 里保存的设置会永久生效，
   改错了需要按 `F9` 载入默认值恢复。

### 已知现象

4. 「AC 插拔唤醒」（电源菜单最后一项）与「State After G3」是**两个独立设置**，
   建议同时开启：
   - `State After G3 = S0` → 关机状态下插上电源自动开机
   - `AC 插拔唤醒 = Enable` → 睡眠状态下 AC 插入/拔出触发唤醒

5. 本机的「AC 插拔唤醒」（电源菜单最后一项）**只对睡眠状态有效**，
   不是「通电开机」。真正控制通电开机的是 `State After G3`。

6. 本机无线网卡（Intel Wi-Fi 6E AX211）的驱动**不支持系统级唤醒**，
   在 Windows 里执行 `powercfg /deviceenablewake` 会提示无权限。
   这是驱动限制，不是设置问题。

7. 解锁后「启动菜单」（Boot）被 Advanced 顶替了。
   这不影响使用 —— 开机按 `F12` 照样能选择启动设备。

### 操作习惯

8. 每次进 U 盘修改完，记得**拔出 U 盘再正常开机**，
   否则又会自动进入解锁界面。

9. BIOS 里 `F9` 是「载入默认设置」，`F10` 是「保存并退出」，
   `ESC` 是「不保存退出」。操作前想清楚按哪个。

10. **如果开机卡住不动：**
    - 长按电源键 **15 秒**强制关机
    - 拔掉 U 盘
    - 重新开机（此时补丁已消失，恢复正常）

---

## 十、故障处理

| 问题 | 原因 | 解决 |
|---|---|---|
| 按 F12 后看不到 U 盘 | 不是 FAT32，或安全启动没关 | 重新格式化为 FAT32，确认 Secure Boot 已关闭 |
| 选 U 盘后黑屏或跳回启动菜单 | `EFI\BOOT\BOOTX64.EFI` 位置不对 | 确认路径是 `U盘根目录\EFI\BOOT\BOOTX64.EFI` |
| 进入 Shell 但没有自动运行 | `startup.nsh` 缺失或文件名不对 | 在 Shell 下手动输入 `fs0:` 然后 `SREP.efi`（若 fs0 不对，先 `map -r` 查看） |
| 报 `The Form Set couldn't be found` | BIOS 版本不匹配 | 确认 BIOS 版本是 `RMAMT6B0P0B0B` |
| 界面出来了但没有 Advanced | 补丁未生效 | 确认看到 `Success! Main Setup DXE found and patched!` |
| 改完设置后电脑开不了机 | 改错了选项 | 开机按 `F2` → `F9` 载入默认 → `F10` 保存 |

---

## 十一、文件清单

| 文件名 | 大小(字节) | SHA256(前16位) | 说明 |
|---|---:|---|---|
| `EFI\BOOT\BOOTX64.EFI` | 1,163,264 | `4922D9885AC92D8A` | EDK2 UEFI Shell 2.2 |
| `SREP.efi` | 657,920 | `540ACBBEDA0AA7B4` | SmokelessRuntimeEFIPatcher 0.2.5 |
| `SREP_Config.cfg` | 1,339 | `9E5236FAFFC52137` | 本机型补丁配置 |
| `startup.nsh` | 22 | — | 开机自动执行脚本 |

---

## 十二、技术细节

### 为什么普通方法改不了

本机固件对 Setup 类变量的写入做了严格限制：

| 尝试方式 | 结果 |
|---|---|
| Windows `SetFirmwareEnvironmentVariableEx` | ❌ 错误 19 `WRITE_PROTECT` |
| UEFI Shell `dmpstore -l` | ❌ `Write Protected` |
| H2OUVE（Windows 版） | ❌ `IHIS Table Not Found`（无 IHISI 接口） |
| RU.EFI | ❌ 只枚举到 1 个变量 |

**但 BIOS 界面自身的保存路径是通的** —— 这是关键转折点。
所以思路改为「先让隐藏选项显示出来，再用 BIOS 界面的正常流程保存」。

### 补丁原理

`SREP_Config.cfg` 做两件事：

1. **`Op SuppressIfPatcher`**
   扫描 HII 数据库，把所有 `EFI_IFR_SUPPRESS_IF` 的条件改为恒假，
   让被压制的选项全部显示。这一步解锁了「电源菜单」。

2. **`Op Patch`**（替换菜单 GUID 表）
   `SetupUtilityApp` 中存在两张硬编码的菜单 GUID 表：

   ```
   数组 A @ 0x25B0 : Main, Security, Power, Boot, Exit          ← 实际显示的 5 项
   数组 B @ 0x2610 : Main, Advanced, Security, Power, Boot, Exit
   ```

   数组 A 与界面菜单完全一致。补丁把数组 A 中 `Boot` 的 GUID
   替换为 `Advanced` 的 GUID，从而让 Advanced 出现在菜单里。

   - `Boot` → `2D068309-12AC-45AB-9600-9187513CCDD8`
   - `Advanced` → `C6D4769E-7F48-4D2A-98E9-87ADCCF35CCC`

### 关键变量

| 变量名 | GUID | 大小 | 偏移 | 说明 |
|---|---|---:|---|---|
| `Setup` | `A04A27F4-DF00-4D42-B552-39511302113D` | 1200 | `0xF2` | AC 插拔唤醒（仅睡眠） |
| `PchSetup` | `4570B7F1-ADE8-4943-8DC3-406472842384` | 6082 | **`0x21`** | **State After G3** |

> **注意**：IFR 里把 `A04A27F4` 这个 VarStore 叫作 `SystemConfig`，
> 但实际的 UEFI 变量名是 **`Setup`**，且同 GUID 下还有一个 1 字节的
> `NetworkSetup`，操作时注意区分。

### 电源菜单选项对照（SystemConfig `0xF2` 附近）

| 偏移 | 名称 | 默认 |
|---|---|---|
| `0xE9` | 内置键盘唤醒 | 1 (开) |
| `0xEA` | 外置键盘鼠标唤醒 | 0 (关) |
| `0xEB` | 触摸屏唤醒 | 1 (开) |
| `0xEC` | 触摸板唤醒 | 1 (开) |
| `0xED` | 开盖唤醒 | 1 (开) |
| `0xEE` | 有线网络唤醒 | 0 (关) |
| `0xEF` | 无线局域网唤醒 | 0 (关) |
| `0xF0` | 无线广域网唤醒 | 0 (关) |
| `0xF1` | 语音唤醒 (WoV) | 1 (开) |
| `0xF2` | AC 插拔唤醒 | 0 (关) |
| `0xF3` | USB 充电 | 0 (关) |
| `0xF4` | USB 充电阈值 | `0x1E` (30%) |

---

## 十三、恢复默认

- BIOS 里按 **`F9`** 载入默认值，再按 **`F10`** 保存，即可恢复出厂设置
- 或者重新进 `Advanced` → `PCH-IO Configuration`，
  把 `State After G3` 改回 `S5 State`

---

## 免责声明

**本仓库内容仅供学习与技术研究使用。修改 BIOS 设置存在风险，请自行承担后果。**

- 作者不对因使用本工具导致的任何设备损坏、数据丢失或保修失效负责
- 补丁仅作用于内存，不修改 Flash，但仍可能因误改 BIOS 选项导致无法开机
- **请务必确认机型与 BIOS 版本匹配后再使用**
- 操作前建议记录原始设置值，以便恢复

---

## 致谢与许可

本仓库的配置文件与文档为原创，采用 [MIT License](LICENSE)。

其中包含的第三方二进制文件遵循各自许可：

| 组件 | 来源 | 许可 |
|---|---|---|
| `EFI/BOOT/BOOTX64.EFI` | [EDK2 UEFI Shell](https://github.com/tianocore/edk2) | BSD-2-Clause-Patent |
| `SREP.efi` | [SmokelessRuntimeEFIPatcher](https://github.com/Maxinator500/SmokelessRuntimeEFIPatcher-RUS)（原作者 SmokelessCPUv2） | 上游未声明明确许可证 |

> SREP 原始仓库已被删除，本仓库使用的是社区维护的备份分支。
> 若原作者要求，将移除相关二进制文件。

感谢：

- [SmokelessCPUv2](https://github.com/SmokelessCPUv2) 开发 SREP
- [Maxinator500](https://github.com/Maxinator500) 维护备份分支并提供 `SuppressIfPatcher`
- [TianoCore / EDK2](https://github.com/tianocore/edk2) 提供 UEFI Shell

---

## 相关项目

- [zsanya322-maker/redmibook-tm2309-linux-modes](https://github.com/zsanya322-maker/redmibook-tm2309-linux-modes)
  —— 同机型（TM2309 / RMAMT6B0P0B0B）的性能模式控制，本文档的部分机型信息参考自此
