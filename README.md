# WMS Wave Assistant v0.5.7-test

三波自动流程测试版。只提供 GitHub Actions 测试包，尚未发布正式 Release。

## 使用

关闭旧 EXE，下载并解压测试包，运行 WMS-Wave-Assistant.exe。点击“登录 WMS”完成登录，并把 **WMS 网页切换为 English**。助手可选中文 / Español。

选择 **首波、次波或尾波**，点击“开始执行”。无需逐个选择阶段。当前步骤、查询范围和包裹数会显示在时间线与日志里。执行时不要用同一账号从另一窗口创建、确认或取消波次。

- 首波：隔离爆品 6–999 → 暂存所有剩余 Single（One item only、不限制数量）→ 按区域/巷道释放 Multis → 取消 Single 暂存 → 释放 Single 1–3 → 取消爆品暂存 → 按区域/巷道释放尽量接近但不超过 2000 包的爆品。
- 次波：隔离爆品 → 全部普通 Single → 全部 Multis；爆品保持暂存留到尾波。
- 尾波：隔离爆品 → 全部普通 Single → 全部 Multis → 取消台账中爆品暂存 → 爆品 6–999 → 剩余检查。超过 999 的数量单独报告，留待人工。

普通 Single 释放范围为 1–5，避免在爆品隔离后把超过 999 的组误当普通 Single 放出；首波暂存 Single 仍不限制 Package quantity。

所有正常释放按 TTGROUND、Pallet L/H、Box L/H、Bin L/H 顺序。每批从 3 个完整巷道开始（可调）；头尾段一起选，Search 后 Total package 不足 30 则增加完整巷道，不跨当前区域和高低位。Zone Info 使用 Include，Virtual zone 单区使用 Totally included，同时取交集。Multis 额外运行已定义的混区阶段；Single 和 Top-selling 无混区阶段。全部 Multis、选定 Single 和尾波尾批即使不足门槛也处理。

首波爆品先从数量下限 100 开始只读搜索阈值，上限仍为 999；候选总量超过 2000 时也可提高下限。选定阈值后分区分巷道实际执行，每批重新核对剩余额度。数量变动时暂停或暂留超额批次，不使用任意行截断订单组，因此可能少于 2000。

## 自动核验与异常

提交前读取当前账号的 Wave Confirmation 基线。最终 OK 仅点一次，随后核对新增 wave number、Creator、Created at、来源、Created/Confirmed 状态，并记录新增波次实际包裹合计；与 Search 的数量差异只记日志。Initialized/Confirming 不视为完成，Failed/Canceled 等停止。无法排除同账号同时操作的情况，因此执行期间必须避免同账号并发放单。

暂存只保存在本机 round-ledger.json。自动取消仅操作该台账中当前账号、当前仓库的 held 波次；跨次波/尾波沿用仍有效的爆品暂存。Confirmed 波次不会自动取消。取消后等待 Canceled，再检查对应包裹数量回池，不靠固定延时直接继续。

主要等待上限：页面/查询 30 秒、弹窗过渡 12 秒、创建核验 120 秒、取消状态 120 秒、取消回池 120 秒。每个页面查询也有自己的超时。空查询只有在本次 Search 成功且稳定出现 “Total package: -” 与 “No data available” 时才按零包跳过。

任何写入前先落盘 pending 意图；异常后不自动重试。暂停后可“重新核验待处理结果”（只读）。暂存步骤核验成功后可“从下一步继续”；其他暂停位置若不支持续跑，“结束暂停轮次”只结束本机流程，不回滚已创建波次。若仍无法核验，保留锁定与台账并人工检查；不要删除台账绕过锁定。暂停不是从头自动重跑。

收尾会查各 Multis 页签、Single 1–5、Top 6–999 和 >999。每类完成分区后执行一次全仓兜底：清空实体/虚拟区域，保留类型和本轮数量范围，非零即处理。Multis 分三个明确页签；次波不释放爆品；首波爆品兜底保留已选门槛及累计 2000 上限，超额整批暂留。最后核验仍有应释放残留则暂停，不循环追赶新订单。

## 验证范围

真实 WMS 已检查：Creator 查询、完整 wave number 表格结构、状态枚举、Created 行的 Cancel 入口、巷道和虚拟区名称、空查询结果；此前已走到策略选择、Not Operated 与最终 OK 可用的位置，但未提交。

**没有在真实 WMS 创建或取消测试波次。** 当前可取消波次属于其他账号，未拿它测试。取消弹窗及完整三波写入路径由隔离 HTML / 假浏览器回归测试验证，仍需使用者在实际环境验收。

## Español

Cierre la versión anterior, extraiga el ZIP y ejecute el EXE. Inicie sesión en WMS y cambie WMS a English; el asistente permite Español. Elija Primera, Segunda o Final y pulse Ejecutar. Las etapas se ejecutan en orden; cualquier resultado incierto pausa el proceso sin repetir escrituras.

Primera: reservar top 6–999 y Single restantes, liberar Multis por zona/L/H/pasillo, cancelar reservas Single y liberar 1–3, cancelar reservas top y liberar hasta 2000 paquetes top en total. Segunda: reservar top, liberar Single normales y Multis; mantener top reservado. Final: reservar top, liberar Single normales y Multis, cancelar las reservas top registradas y liberar top 6–999. Los grupos superiores a 999 quedan para revisión manual.

Cada creación se verifica en Wave Confirmation. Solo se cancelan reservas registradas para la misma cuenta y almacén, y se espera su estado Canceled y la devolución de paquetes. No use la misma cuenta simultáneamente en otra ventana. Las pruebas reales de creación y cancelación aún requieren validación del usuario; no se han ejecutado en producción durante el desarrollo.

## 运行与构建

Windows x64，需 Microsoft Edge WebView2 Runtime。EXE 自带 Python 和 Chromium，助手独立窗口与 WMS 浏览器是两个窗口。无需另装 Python。旧的定时配置仍只是程序运行时提醒，不是无人值守启动三波。

源码：安装 requirements.txt，python run.py。开发服务：python run.py --no-open。Windows 构建：python build.py；构建运行单元/浏览器模拟测试，验证打包 Chromium 和原生桌面窗口。

本机数据默认 ~/.wms-wave-assistant；分发包不含登录凭据、波次台账或仓库数据。

## 查询恢复（v0.5.7-test）
默认每 40 次成功查询清理 HTTP 缓存并刷新，可设为 0 关闭或调整到 1–1000。查询超时/结果不稳定最多重试两次：先刷新，再重新打开页面；恢复后核对账号仓库并重设全部筛选。保留 Cookie、登录存储和台账。登录失效、定位歧义和业务异常直接暂停。恢复本身失败也暂停。汇波和取消写入不重试，未核验意图禁止自动恢复。

## v0.5.7-test 查询一致性修正
Reset、切换类型页签和 Search 之间等待请求结束；Search 后等待响应正文、页面网络空闲、加载状态消失，再读取稳定的 Total package。校验 Package quantity 实际输入值。页面后台请求始终繁忙则超时暂停，不接受旧数字。保留提交前数量变化保护。Home 启动路由打断深链时，转用可见菜单导航。

## v0.5.7-test：数量差异与续跑
创建回执核对账号、新波次编号、创建时间、来源和有效状态；实际包裹数为正即可登记。数量差异记录在 quantity_differences，不再作为提交后阻断条件。提交前数量变化保护及首波爆品预算保护保留；预算按实际已释放数量累计。
已有“暂存爆品/暂存 Single”待核验记录：登录相同账号仓库，点击“核验已创建波次”，成功后点击“从下一步继续”。保留同一轮 ID，跳过已核验的整个暂存步骤。其他中途失败的多批次步骤不会自动跳过。
导出台账改为后端保存到数据目录 exports 子目录，并在界面显示可复制的完整路径，不依赖 WebView 下载。

## v0.5.7-test 自动恢复入口

登录后选择原任务对应的首波/次波/尾波，点击开始执行。程序在后台检查待处理结果，有效暂存继续下一步；全部明确为 Canceled 的暂存记录归档为已取消，再执行该暂存步骤。Failed、未完成取消、状态不明或不支持续跑的中途释放仍暂停，不把未知结果视为成功。不需要反复点核验按钮。此节替代上文旧版的手动核验入口说明。

Inicio automático: seleccione el modo de la tarea pendiente y pulse Ejecutar. La aplicación comprueba el resultado anterior y continúa; una reserva completamente cancelada vuelve al paso de reserva. Los resultados inciertos siguen en pausa.

## v0.5.7-test：直接使用最新查询提交

移除提交前两次 Total package 相等的要求。最新 Search 非零就按该查询 Consolidate，零则跳过；仍遵守首波爆品累计 2000 上限。旧版因数量变化而暂停、尚未提交的暂存步骤，点击开始即可恢复。新波次回执仍用于记录编号和后续取消自己的暂存波次。

## v0.5.7-test 当前规则（替代上文旧版分批说明）

Multis 用 All + Clean-up Multis，一次包含三类，不再逐页签拆单。Ground 一批；Pallet、Bin 每个 L/H 各取完整区域，非零即放；仅 Box 从三个完整巷道起步，不足 30 增加一个，尾批非零即放。这一分区规则适用于 Multis/Single/Top。Pallet 实体巷道含 130–131，Box 含 130–131 至 178–179，Bin 从 178–179 至 190–191；单区用 Totally included 虚拟区取交集，混区用 Include。

回执单独标签页；每轮仅首次取完整基线，后续复用并追加已创建编号；不再为取基线跳离筛选或重复 Search。默认关闭每 40 次定期刷新，仍在查询超时后恢复；自定义刷新周期可保留。缩短固定等待，仍等待 Search 请求和加载结束。末尾残留审计仍分三类读取，不创建碎波。

无 pending 且明确在最终 OK 之前停止的任务，开始后关闭未提交的策略弹窗，从当前阶段剩余订单重查；保留已创建台账，跳过已完成的暂存阶段。未确认的提交仍先核对。

Español: Multis usa All + Clean-up Multis. Solo Box amplía desde tres pasillos si hay menos de 30 paquetes; Pallet/Bin usan toda la zona L/H. Zonas puras: Totally included; mixtas: Include. Los recibos usan una pestaña separada, sin repetir Search. Se desactiva la actualización periódica por defecto.

## v0.5.7-test：明确区分重新开始和续跑

“从第 0 步开始”按所选首波/次波/尾波新建一轮，结束旧本机流程、自动备份台账并保留所有波次；不会取消 WMS 中已有波次。“继续上次任务”沿用原策略、轮次和进度。若有未核对提交，先后台核对；核对失败则不覆盖旧任务。

“Iniciar desde el paso 0” crea una ronda nueva, guarda una copia del registro y conserva las oleadas existentes. “Continuar tarea anterior” mantiene la ronda y el progreso anterior.
