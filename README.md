# numpad12 —— 12 键数字小键盘（BLE + USB 双模）

基于 ZMK，板子：nologo ProMicro NRF52840（兼容 Nice!Nano）

## 这个仓库是什么

一个 **ZMK 用户配置仓库**。ZMK 固件不在本地编译，而是：
本地只有配置（几十 KB） → 推到 GitHub → GitHub Actions 云端编译 → 下载 .uf2 → 拖进键盘

这是 ZMK 官方推荐、也是所有开源键盘项目的标准做法。
（本地编译需要 Zephyr + ZMK + ARM 工具链，5~8GB）

## 接线（12 个按键，直连式，不用二极管）

每个按键：**一端接下面的引脚，另一端统一接 GND**。

| 引脚 | 键位 |
|---|---|
| D0  | 7 |
| D1  | 8 |
| D2  | 9 |
| D3  | 4 |
| D4  | 5 |
| D5  | 6 |
| D6  | 1 |
| D7  | 2 |
| D8  | 3 |
| D9  | 0 |
| D10 | . |
| D14 | Enter |

没接按键时，用镊子/杜邦线把某个引脚碰一下 GND，就相当于按下那个键，
可以直接在电脑上打字验证。

## 目录

```
build.yaml                                   构建矩阵（编译哪几个固件）
.github/workflows/build.yml                  GitHub Actions 流水线
config/west.yml                              ZMK 版本（跟随 main）
config/boards/shields/numpad12/numpad12.overlay   引脚定义
config/boards/shields/numpad12/numpad12.keymap    键位表
config/boards/shields/numpad12/Kconfig.*          键盘名字等
```

## 使用步骤

1. 在 GitHub 新建一个仓库（名字随意，例如 `zmk-config`），**不要**勾选 README
2. 把本目录所有文件推上去：
   ```
   git remote add origin https://github.com/<你的用户名>/zmk-config.git
   git branch -M main
   git push -u origin main
   ```
3. 打开仓库的 **Actions** 页，等 5~10 分钟，出现绿色对勾
4. 点进那次运行，页面底部 **Artifacts** 下载 `numpad12`（是个 zip）
5. 解压得到 `numpad12-nice_nano.uf2`
6. 板子：0.5 秒内短接 RST-GND 两次 → 电脑出现 `NICENANO` 盘 → 把 uf2 拖进去
7. 磁盘自动弹出，键盘启动

## 配对与切换（双模）

- **蓝牙**：键盘开机后自动广播，电脑/手机蓝牙里添加设备，看到 `NumPad12`
- **USB**：插上数据线，ZMK 默认**优先走 USB**（同时也在充电）
- 最多可配对 **5 台**蓝牙设备，默认 keymap 里没放切换键。
  以后加三个键：`&bt BT_SEL 0` / `&bt BT_SEL 1` / `&bt BT_CLR` 就能切换和清空
- 键盘连不上：先刷 `settings_reset` 固件（build.yaml 里已经配好），再重新配对

## 改键位

改 `numpad12.keymap` 里的 `bindings`，顺序和 overlay 里的引脚一一对应。
改完 push，Actions 重新编译。

可视化改键工具（推荐）：https://nickcoutsos.github.io/keymap-editor/
授权 GitHub 后可以直接在浏览器里拖格子，它会自动提交改动。

## 常见坑

- `build.yaml` 里 board 必须写 `nice_nano//zmk`。网上教程的 `nice_nano_v2` 在
  2025-12 之后的 ZMK 上已经不能用了（board 改名 + revision 机制）。
- overlay 和 keymap 的**顺序必须一致**，否则按 A 出 B。
- 直连式最多用 12 个引脚；想做到 17 键标准数字键盘，就要改成行列矩阵（要二极管）。
- 烧完不亮/无反应：重新拖一次 uf2；还不行检查是否误刷了 `settings_reset`。