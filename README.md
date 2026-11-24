# WMS Wave Assistant v0.7.0-test

**测试版，不是正式发布。** Windows x64，中文 / Español。EXE 包含 Python 和 Chromium；桌面窗口需要 Edge WebView2 Runtime。EXE 尚未代码签名，Smart App Control 可能拦截；本次功能更新不解决签名问题。

## 开始

关闭旧版，运行测试版并登录 WMS。**WMS 网页必须切换为 English**，助手本身可用中文或西语。运行中不要使用同一账号在其他窗口创建或取消波次。

首波和尾波现在是固定流程。点击“开始运行”新建一轮并保留已有波次与台账；点击“继续上次任务”继续原版本的计划和进度（包括旧次波）。新任务不再提供次波。未决提交先核对结果，避免重复提交。

## 首波

1. 按区域、L/H 释放 Top-selling，查询目标尽量接近且不超过 1500 包（6–999 范围内，从 100 起调整下限）。
2. 锁定剩余爆品（全仓 One item only，6–999）。
3. 锁定 Single（全仓 One item only，不限制 Package quantity）。
4. 分区、L/H 释放 Multis，使用 All + Clean-up Multis 合并三种类型。
5. 解除本程序记录的 Single 锁定。
6. 分区、L/H 释放 Single，Package quantity **1–4**。
7. 解除本程序记录的爆品锁定，结束。**不查询剩余包裹。**

1500 是查询预算，不是 WMS 实际生成数量的精确承诺；动态订单和策略过滤会导致差异。低于释放门槛或超过剩余预算的批次留置，不为了凑数循环追单。

## 尾波

解除本程序记录的已有爆品锁定 → Top-selling 6–999 → 解除已有 Single 锁定 → Single（One item only，**Package quantity 留空**）→ Multis → 查询并记录剩余包裹后结束。

尾波的 Single 不按件数排除爆品；按照用户指定的先 Top-selling 后 Single 顺序运行。新到订单可能在后续节点被选中。

## Flex 自选放单

固定首波/尾波的自选节点功能移到 Flex：
- 锁定 / 解除锁定：全仓锁爆品、锁 Single、解除已记录的爆品或 Single 锁定，按界面显示顺序执行。
- 释放区域：Top-selling / Single / Multis 各自可选 Ground、Pallet L/H、Box L/H、Bin L/H，共 21 个独立节点。
- Top-selling 使用 6–999；Single 使用 1–5；Multis 使用 All + Clean-up Multis。
- **只处理勾选区域，不自动进行全仓兜底或剩余核验。** 无选项时不能启动。
- 每个释放区域可设置提交批次上限，0 不限；一次提交可能生成多个波次。达到后进入下个所选区域。
- 快捷按钮只选择释放区域，不自动附加锁单或解锁。订单仍处于锁定状态时，需要明确选择对应解锁动作。
- 锁 Single 没有数量范围，可能包含未隔离的爆品；锁定顺序为先爆品后 Single。锁定和解除同时勾选会按顺序都执行。

## Box 先大后小查询

每个 Box L/H、类型先查询整区（Zone Info 留空，只选虚拟区）。总量少于 30 时留置；稀疏且达到门槛时整区释放。总量大时按总量估算分组，密集区保持至少配置的完整巷道数（默认 3）。局部不足 30 时成倍扩大范围，不再一次增加一个巷道。每个范围的实际 Search 数量才用于提交判断；估算不作为包裹分配保证。

Pallet / Bin 直接整区 L/H。Box 边界使用 130–131 和 178–179 巷道，通过虚拟区取交集。单区域 Totally included，混区 Include。释放至少 30 包（配置更高时遵守更高值），暂存隔离不受该门槛限制。

首波/尾波每类分区释放后保留分类和该阶段数量范围做一次全仓兜底；Flex 不进行范围外兜底。实际 WMS 波次可能小于查询数，记录差异但不据此重复提交。

## 等待、恢复和记录

实时阶段、区域/巷道、查询、提交、回执等待和用时持续显示。回执使用独立标签页，默认不定期刷新。查询超时有界恢复；已提交但结果未知保留 pending。取消只操作本机台账中、当前账号与仓库的暂存波次。数据默认位于 `~/.wms-wave-assistant`，更新不删除台账。导出台账按钮显示实际保存路径。

## Español

Versión de prueba. WMS debe estar en **English**. Primera: top hasta 1500 → reservar top → reservar Single → Multis → cancelar reserva Single → Single 1–4 → cancelar reserva top; sin auditoría final. Final: cancelar reservas top → top → cancelar reservas Single → Single sin filtro de cantidad → Multis → pendientes.

Flex permite seleccionar reservas y zonas Ground / Pallet / Box / Bin, L/H, para Top-selling (6–999), Single (1–5) y Multis. Solo ejecuta las zonas marcadas, sin pasada global. Box consulta primero toda la zona, luego divide por volumen y amplía por duplicación cuando faltan paquetes. Mínimo de consulta: 30. Continuar conserva el plan anterior; iniciar usa el nuevo plan.

## 构建

安装 requirements.txt 后用 `python run.py` 启动源码桌面版。Windows x64 运行 `python build.py`：回归测试 → 打包 EXE → Chromium / 桌面自检。生产 WMS 创建/取消测试未执行。程序内时间配置仍为提醒，不是无人值守定时启动。
