# WMS Wave Assistant v0.6.1

Windows x64 桌面放单助手，中文 / Español。EXE 包含 Python 和 Chromium；桌面窗口需要 Microsoft Edge WebView2 Runtime。

## 使用

关闭旧版，下载并运行新 EXE，登录 WMS，并将 **WMS 网页切换为 English**。

选择首波、次波或尾波：
- **运行所选节点**：勾选这次要执行的阶段，按原时间线顺序运行；结束旧本机流程，备份台账并保留已有波次。
- **继续上次任务**：沿用旧轮次、策略和进度。
- **停止后续操作**：停止后续提交；已提交但结果不明的操作保留记录，避免重复提交。

已有未决提交会先检查结果；运行所选节点不会自动取消未选取消节点的历史波次。导出台账会在界面显示保存路径。

## 三波顺序

- 首波：暂存爆品 6–999 → 暂存剩余 Single → Multis → 取消 Single 暂存 → Single 1–3 → 取消爆品暂存 → 尽量接近且不主动超过 2000 包的 Top-selling。
- 次波：暂存爆品 → Single 1–5 → Multis，爆品留存。
- 尾波：暂存爆品 → Single 1–5 → Multis → 取消爆品暂存 → Top-selling 6–999。

正常释放分 Ground、Pallet L/H、Box L/H、Bin L/H；Multis 另外处理混区。全仓兜底保留分类及本轮数量范围，每类只执行一次。最终记录剩余订单，不循环追逐新订单。超过 999 的组单独记录。

## v0.6.1 更新

- 每个执行阶段显示分区、巷道与动作子节点，包括查询、提交、等待新波次、完成、留置和暂停；运行指示与计时持续更新。日志记录每次查询耗时。
- 整区不逐个点击巷道，只选择虚拟区。Box 部分范围仍从三个完整巷道开始，低于门槛逐个扩展；Pallet/Bin 直接整区 L/H。单区 Totally included，混区 Include。
- **所有释放批次，包括尾批与全仓兜底，Search 后至少 30 包才提交**，配置更高时采用更高值。不足则扩展 Box 或留置。用于隔离订单的暂存不受释放门槛限制。
- **查询门槛不等于实际波次大小保证**：WMS 动态分配或策略过滤后实际数量可能更少，程序记录真实数量，不自动取消已生成的释放波次。
- 取消暂存确认 Canceled 后继续，不再反复查询等待包裹数相等。
- 回执使用单独标签页，每轮基线仅首次读取，后续复用；不因数量差异阻断。默认关闭定期刷新，查询超时可清理 HTTP 缓存恢复，保留登录信息。

## 等待与记录

页面查询通常上限 30 秒；生成回执、取消状态上限各 120 秒。固定界面等待和 WMS 处理时间仍存在，未承诺生产环境耗时。每次最终 OK 只提交一次。程序匹配当前账号、仓库的新 wave number；同一账号请勿同时在另一个窗口创建或取消波次。

自动取消仅处理本机台账中、当前账号与仓库的暂存波次。失败或结果不明时保留 pending，不把未知结果视为成功。数据默认保存在 `~/.wms-wave-assistant`；分发包不包含登录凭据或台账。

## Español

Cierre la versión anterior, ejecute el EXE e inicie sesión. WMS debe estar en **English**; el asistente puede usar Español. Elija Primera, Segunda o Final y pulse **Iniciar desde el paso 0**, o **Continuar tarea anterior** para conservar el progreso.

Cada etapa muestra zona/pasillos, acción y tiempo de espera. Las zonas completas no seleccionan pasillos individualmente; solo Box amplía lotes. Se requieren al menos 30 paquetes en la consulta para liberar, incluso en la pasada final. Las reservas de aislamiento están exentas. WMS puede generar menos paquetes que los consultados; se registra el resultado real. Los pendientes se registran al terminar sin repetir indefinidamente. Tras confirmar una cancelación, el proceso continúa sin comparar el número de paquetes devueltos.

## 构建与验证

安装 `requirements.txt` 后运行 `python run.py`。Windows 构建运行 `python build.py`，包含单元测试、隔离浏览器测试、打包 Chromium 与桌面窗口测试。

本版本没有在生产 WMS 创建或取消测试波次；自动化回归测试不替代生产验收。定时配置仍为程序运行时提醒，不是无人值守启动三波。


## v0.6.1 节点组合执行

首波、次波、尾波均支持逐项勾选。未选节点明确显示“未选，跳过”，不会自动补回 Multis。快捷“仅 Single”选择当前场景的取消 Single 暂存（若存在）、Single 和剩余检查；“仅 Top-selling”选择取消爆品暂存、Top-selling 和剩余检查，次波不提供爆品释放。

“每个释放节点最多汇波批次”设为 3，表示每个选中的 Single / Multis / Top-selling 节点成功提交 3 批后转入下一个所选节点；0 不限。它不是 WMS 的系统容量配置，也不是包裹数或生成波次个数限制（一次提交可能生成多个波次）。暂存及取消不计入释放批次。

新任务保存所选节点和上限；续跑保留这些设置及已提交批次数。原来停在 Multis 时，可改选“仅 Single”后运行所选节点。已有未决提交仍先自动检查结果，不能跳过未知提交导致重复操作。未选取消节点时，对应暂存订单仍留在 WMS 中。

Español: seleccione las etapas o use Solo Single / Solo Top-selling. Ejecutar selección crea una ronda nueva sin repetir etapas no seleccionadas. El máximo se aplica a envíos exitosos por etapa de liberación; 0 significa sin límite. Continuar conserva selección y contador. Las reservas requieren su cancelación antes de liberarse.
