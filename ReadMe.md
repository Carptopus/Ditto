# Ditto 大历史记录优化版（本 Fork）

> 本段内容仅说明 `Carptopus/Ditto` Fork 的大历史记录优化，与下方上游项目原始 README 分开维护。

本分支针对十万级剪贴板历史下 Quick Paste 首次打开、首屏加载和连续滚动迟缓的问题进行了优化，同时保留 Ditto 原有的单 SQLite 数据库和数据格式。

[下载 `large-history-v1` x64 增量更新包](https://github.com/Carptopus/Ditto/releases/tag/large-history-v1)

这是 `Carptopus/Ditto` Fork 的测试构建，不是上游官方安装包。更新包包含 `Ditto.exe`、`ICU_Loader.dll` 和 `Addins/DittoUtil.dll`，需要在 Ditto 正常退出后覆盖现有文件；不会包含或覆盖 `Ditto.Settings`、语言文件、主题及剪贴板数据库。

## 优化内容

- 首屏记录加载完成后立即显示，不再等待完整 `COUNT(*)`；总数查询使用独立的只读数据库连接，避免占用列表加载所用的主连接。
- 连续向后浏览使用与现有排序一致的游标分页，并将预取范围扩大到三屏；虚拟列表的随机深处跳转继续保留 `OFFSET` 回退，原有交互不变。
- Ditto 启动后预创建隐藏的 Quick Paste 窗口，提前完成窗口和控件初始化。
- 为默认主列表新增匹配筛选及排序的 `Main_NoGroup2` 复合索引，避免 SQLite 选择错误索引后建立临时排序树。
- 修复固定盘被误判为网络共享的问题，例如位于 `E:\` 的数据库现在可以正常复用进程内列表缓存；UNC 路径和映射网络盘仍按网络共享处理。
- 移除每分钟随机读取数据库 1 KiB 的无效缓存预热逻辑。

## 实测效果

在一份包含 118,498 条记录、约 212 MiB 的实际数据库副本上，旧首屏查询计划的冷查询可达到约 2 秒；新增索引后，首屏 40 条记录的 SQL 查询约为 0.115 至 0.222 毫秒，结果 ID 完全一致。用户在本机实际运行后确认：进程启动和搜索显示明显加快，重启后的首次 Quick Paste 首屏也能非常快地显示。

上述毫秒数据只代表 SQL 路径，不等同于完整窗口打开耗时；不同机器、配置和数据库内容会影响最终体验。

## 兼容性与边界

- 不拆分数据库，不迁移剪贴板数据，继续使用 Ditto 原有数据库文件。
- 第一次连接数据库时会执行 `CREATE INDEX IF NOT EXISTS Main_NoGroup2 ...`，新增索引约占 2.6 MiB。
- 列表缓存保存在进程内，退出后会清空；复合索引保存在数据库中，重启后仍然有效。
- 该 Release 是 GitHub Actions 生成的 x64 开发构建，未使用上游正式发布的代码签名。

---

## 上游项目原始 README

# [Ditto - Clipboard Manager](https://github.com/sabrogden/Ditto/releases/download/3.25.113.0/DittoSetup_3_25_113_0.exe)

![GitHub Downloads (all assets, latest release)](https://img.shields.io/github/downloads/sabrogden/Ditto/latest/total) ![GitHub commits since latest release](https://img.shields.io/github/commits-since/sabrogden/Ditto/latest) ![GitHub contributors](https://img.shields.io/github/contributors/sabrogden/Ditto)



[Help/Wiki](https://github.com/sabrogden/Ditto/wiki)&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;[Forums](https://github.com/sabrogden/Ditto/issues)&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;[Donate](https://www.paypal.com/donate/?item_name=Donation+to+Ditto&cmd=_donations&business=sabrogden%40gmail.com&Z3JncnB0=)&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;[Beta](https://ditto-cp.sourceforge.io/beta/)&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;[Source](https://github.com/sabrogden/Ditto)&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;[History](https://github.com/sabrogden/Ditto/releases)

Ditto is an extension to the standard windows clipboard. It saves each item placed on the clipboard allowing you access to any of those items at a later time. Ditto allows you to save any type of information that can be put on the clipboard, text, images, html, custom formats.


1. [Installer](https://github.com/sabrogden/Ditto/releases/download/3.25.113.0/DittoSetup_3_25_113_0.exe)
2. [Portable](https://github.com/sabrogden/Ditto/releases/download/3.25.113.0/DittoPortable_3_25_113_0.zip)
3. [Chocolatey](https://chocolatey.org/packages/ditto/3.23.124.0) choco install ditto
4. [Chocolatey Portable](https://chocolatey.org/packages/ditto.portable/3.23.124.0) choco install ditto.portable
5. [Winget](https://winget.run/pkg/Ditto/Ditto) winget install -e --id Ditto.Ditto
6. [Windows Store App](https://www.microsoft.com/en-us/store/p/ditto-cp/9nblggh3zbjq)  


## Basic Usage

1. Run Ditto
2. Copy things to the clipboard, e.g. using Ctrl-C with text selected in a text editor.
3. Open Ditto by clicking its icon in the system tray or by pressing its Hot Key which defaults to Ctrl + ` – i.e. hold down Ctrl and press the back-quote (tilde ~) key.
4. Double click or press enter on the item to paste it to the previous window.

## Local First
- No login
- No cloud
- No telemetry

## Windows Code-Signing Policy
Free code signing on Windows binaries provided by SignPath.io, certificate by SignPath Foundation.
<br>
<br>

<img src="ditto.gif">

