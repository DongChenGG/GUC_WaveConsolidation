# WMS 放单助手 v0.1

这是可独立运行的 **Python + Playwright 只读试运行原型**，不需要 GPT 或 Codex 保持打开。需要联网访问 WMS。

## 此版本能做什么

- 预置用户定义的 30 个阶段，逐阶段询问开始或暂缓。
- 精确配置 Pallet 124–130、Box 131–178、Bin 179–191；混合边界巷道保留两半。
- 普通货架按三个完整巷道起步；在数量口径与仓区语义已核验的 Single 预览中，不足阈值则逐巷道扩展。尾批不足记录为留到后续。
- 混区阶段查询整个指定范围，L/H 分开。
- 设置 Zone / Virtual zone 匹配方式、最低数量、起始巷道数；阶段名称、虚拟区和策略可通过编辑按钮修改。
- 独立 Chromium 浏览器登录、填写筛选并查询；查询记录导出 JSON、本地日志。
- 默认纽约时区的 09:00 爆品提醒、12:00 截单收尾提醒。

## 明确不做什么

- **没有创建波次、Confirm Wave、取消波次、修改 WMS 策略的代码路径。** “预览”不是放单。
- 定时功能当前是提醒，不是自动放单。必须保持程序运行、电脑不休眠；错过时刻不补跑。
- Multis 的 All 查询包含 Single，无法据此取得真正可放 Multis 总量，所以不自动判断其扩展和放单。
- 12:00 是开始还是完成时点、同包跨 L/H 的归属、纯区/混区匹配语义尚未核验。当前版本不能承诺 12 点全部放完。
- 暂存爆品与取消暂存是人工检查点，不会自动创建或取消任何波次。
- 爆品保留 6–999 筛选范围；不会自动扩大上限。对 >999 的业务处理仍待明确。
- 生产系统真实登录后查询尚未在此 Python 原型中端到端验证。页面适配基于已观察到的 WMS 控件，界面变化或标签不一致会停止。

## 在其他电脑运行

这是跨平台源码分发包，不是已经分别编译好的 Windows EXE / macOS APP。

1. 解压整个文件夹，不要只复制启动文件。
2. 安装 Python 3.10 或更新版本（建议 3.11/3.12）。Windows 安装 Python 官方版本，确保 `py` 可用。
3. Windows 双击 `Start-Windows.bat`；macOS 双击 `Start-Mac.command`。
4. 首次启动会创建 `.venv`、安装依赖并下载 Playwright Chromium，需联网。macOS 如文件无执行权限，在终端运行 `sh Start-Mac.command`。
5. 程序打开本地管理页 `http://127.0.0.1:8765`。点击“打开 WMS 浏览器 / 登录”，在新的 Chromium 窗口自行登录。
6. 选择一个阶段，核对参数，点击“开始本阶段预览”。实际汇波仍需人工在 WMS 进行。
7. 停止程序：启动终端中按 Ctrl+C。不要同时打开两个本程序实例。

如通过终端安装：

```sh
python3 -m venv .venv
.venv/bin/python -m pip install -r requirements.txt
.venv/bin/python -m playwright install chromium --no-shell
.venv/bin/python run.py
```

Windows 对应使用 `.venv\Scripts\python.exe`。

需要公司 VPN / 内网时，运行电脑必须具备同样的访问条件。不要将现有浏览器配置目录复制到另一台电脑。

## 数据与配置

默认本机目录：`~/.wms-wave-assistant/`。

- `config.json`：数量门槛、匹配方式、时区和提醒时间。
- `phases.json`：30 个阶段的可编辑配置。
- `events.jsonl`：运行事件。
- `last-results.json`：预览结果，应用当前界面只显示当前进程结果。
- `browser-profile/`：独立浏览器登录状态；应按账号凭据保护，**不可随分发包共享**。

可用环境变量 `WMS_ASSISTANT_DATA` 改变数据目录。应用只监听 127.0.0.1，配置操作带本次启动的随机校验标记。

### 匹配方式

默认 Include 只是试查初始值，不意味着“纯区”。确认准确的 Zone Info / Virtual zone 逻辑后再勾选“已核验”。该标记也不会启用实际汇波。

第 12/13 步的 L/H 集合由程序生成，单独修改这两步的 `virtual_zones` 不生效。实体区类型范围通过 `zone_types` 控制。全仓阶段是明确的跨地堆/货架例外。

### 可移植打包

当前 ZIP 在 Windows/macOS 上使用各自的启动脚本。无需 GPT，首次需要 Python 与网络安装依赖。

Windows 单文件 EXE 的一键构建入口是 `Build-Windows-EXE.bat`。它内嵌 Python 和 Chromium，构建后在其他 Windows x64 电脑上无需安装依赖。当前 Mac 环境尚未生成 EXE，详细步骤见 [Windows-EXE说明](Windows-EXE说明.md)。附带可选 GitHub Actions 工作流，但未上传或触发远程构建。

## 开发与验证

```sh
python -m unittest discover -s tests -v
```

已验证：11 项规则/执行队列测试；Python/前端脚本语法；本地网页打开及阶段确认卡。真实 WMS 提交不在此版本中，未做生产放单测试。

适配器使用官方 Playwright Python：[文档](https://playwright.dev/python/docs/library)。锁定版本：[Playwright 1.58.0](https://pypi.org/project/playwright/1.58.0/)。
