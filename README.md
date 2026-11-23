# WMS 放单助手 v0.3.1

此版本支持真实 Wave Consolidation，选择 picking strategy、Not Operated，然后点击最终 OK。需要 Windows x64、网络和 WMS 账号，不需要 Python 或 GPT。

## 中文 / Español

同一个 EXE 支持中文和西班牙语。右上角选择语言，浏览器记住选择。界面、默认阶段、确认提示和程序消息按所选语言显示；WMS 的策略、区域及匹配选项名称、编辑器中的 JSON 和导出原始记录保持不变，避免改变执行配置。用户自定义名称和外部系统返回的原始错误可能保留原文。

Un solo EXE admite chino y español. Seleccione Español arriba a la derecha; el navegador recuerda su elección. Los nombres de WMS y los datos JSON se conservan sin traducir. Cierre la versión anterior antes de ejecutar esta actualización.

## 使用

下载 Release 中的 EXE，双击运行。在本地管理页打开专用 WMS 浏览器并登录。先用预览核对筛选条件；核验 Zone Info / Virtual zone 匹配语义后勾选对应配置并保存。选择阶段，点击“正式汇波”，确认本阶段执行。查询预览按钮仍可使用。

每次提交前写入本地提交记录，点击最终 OK 后等待新出现的成功提示及策略弹窗关闭，再进入下一批。提交失败、超时或程序中断会保留锁定记录。需在 WMS 核对波次号和状态后，在管理页记录核对结果并解除锁定；不会自动重试。停止按钮只停止后续操作，不能撤销已提交波次。

## 范围与限制

- 30 个阶段，每阶段单独开始；普通货架每组三个完整巷道，可配置。
- Multis / Single / Top-selling 统一按每次 Search 后页面 Total package 判断，不足门槛时逐巷道扩展，尾批默认留到后续。阶段确认卡可勾选收尾，允许低于门槛提交。
- 数量门槛默认 30，可配置。每次追加一个完整巷道后重新 Search，读取新的 Total package；达到门槛才提交。不会扩出当前阶段配置的仓区范围。混区阶段仍按整个范围查询。
- L/H 分开；混区使用整个配置范围。Bin 包含 191，Top-selling 数量范围保持 6–999。
- 爆品暂存、第 14 步剩余核验、取消暂存仍需人工操作。
- 定时仍是 09:00 / 12:00 提醒，不是无人值守放单。电脑必须保持运行；不能承诺 12 点全部完成。
- 正式流程使用精确策略名称和 Not Operated 单选项，页面不匹配会停止，不回退到默认策略。若成功提示不符合识别规则，也会锁定要求人工核对。
- 生产 WMS 端到端尚未验证；由使用者进行实际环境测试。自动测试使用本地模拟页面，不操作生产订单。

## 数据

本机配置、日志、登录状态和待核对记录保存在 `~/.wms-wave-assistant/`。不要共享 browser-profile。新版本复用本机配置。一次只运行一个实例，升级前退出旧程序。

## 构建

源码在 `WMS-Wave-Assistant-source.zip`。解压后 Windows 运行 `Build-Windows-EXE.bat`。GitHub Actions 在 Windows 构建，运行规则测试、本地模拟弹窗测试及内嵌 Chromium 启动测试。Release EXE 内置 Python 和 Chromium。

### v0.3.1 修复 / Corrección

修复 Include 文字覆盖内部输入框导致的点击超时；点击 Auxo 可见选择区域，并将选项限定到对应控件的下拉列表。新增复现覆盖层、多选及精确选项测试。

Se corrige el tiempo de espera al pulsar el campo cubierto por Include. Se pulsa la superficie visible del selector y se buscan opciones solo en su lista correspondiente.
