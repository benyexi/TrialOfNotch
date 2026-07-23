# 拳皇97 网页版 (KOF '97 Web Player)

一个纯前端、可在浏览器中直接游玩《拳皇97》的网页。基于开源模拟器
[EmulatorJS](https://emulatorjs.org/)（Neo·Geo / arcade 核心），无需安装任何软件。

## 使用方法

1. 用浏览器打开 `index.html`（建议通过本地 HTTP 服务器，详见下方）。
2. 点击「选择 ROM 文件并开始」，选中你**自己合法拥有**的 `kof97.zip`。
   - Neo·Geo 游戏通常还需要 `neogeo.zip` BIOS，可与游戏 ROM 一起打包成一个 zip 选择。
3. 按 **Shift** 投币，按 **Enter** 开始游戏。

### 默认按键（玩家 1）

| 功能 | 键盘 |
| --- | --- |
| 方向 | 方向键 |
| 轻拳 / 轻脚 / 重拳 / 重脚 | Z / X / A / S |
| 投币 Coin | Shift |
| 开始 Start | Enter |

按键可在游戏内设置菜单中重新映射，并支持手柄；移动端自动显示虚拟按键。

## 本地运行

部分浏览器对 `file://` 下的 WebAssembly 有限制，建议用本地服务器打开：

```bash
cd kof97
python3 -m http.server 8080
# 然后浏览器访问 http://localhost:8080
```

## 版权说明

《拳皇97 / The King of Fighters '97》版权归 **SNK** 所有。本项目**不包含、不分发任何
游戏 ROM**，仅提供开源模拟器前端。请仅使用你依法拥有的游戏副本。
