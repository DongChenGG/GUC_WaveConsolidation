# WMS Wave Assistant v0.5.2-test

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

提交前读取当前账号的 Wave Confirmation 基线。最终 OK 仅点一次，随后核对新增 wave number、Creator、Created at、来源、Created/Confirmed 状态，以及新增波次包裹合计与本次 Search 数量一致。Initialized/Confirming 不视为完成，Failed/Canceled 等停止。无法排除同账号同时操作的情况，因此执行期间必须避免同账号并发放单。

暂存只保存在本机 round-ledger.json。自动取消仅操作该台账中当前账号、当前仓库的 held 波次；跨次波/尾波沿用仍有效的爆品暂存。Confirmed 波次不会自动取消。取消后等待 Canceled，再检查对应包裹数量回池，不靠固定延时直接继续。

主要等待上限：页面/查询 30 秒、弹窗过渡 12 秒、创建核验 120 秒、取消状态 120 秒、取消回池 120 秒。每个页面查询也有自己的超时。空查询只有在本次 Search 成功且稳定出现 “Total package: -” 与 “No data available” 时才按零包跳过。

任何写入前先落盘 pending 意图；异常后不自动重试。暂停后可“重新核验待处理结果”（只读）。核对完成后，“结束暂停轮次”只结束本机流程，不回滚已创建波次。若仍无法核验，保留锁定与台账并人工检查；不要删除台账绕过锁定。暂停不是从头自动重跑。

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

## 查询恢复（v0.5.2-test）
默认每 40 次成功查询清理 HTTP 缓存并刷新，可设为 0 关闭或调整到 1–1000。查询超时/结果不稳定最多重试两次：先刷新，再重新打开页面；恢复后核对账号仓库并重设全部筛选。保留 Cookie、登录存储和台账。登录失效、定位歧义和业务异常直接暂停。恢复本身失败也暂停。汇波和取消写入不重试，未核验意图禁止自动恢复。

## v0.5.2-test 查询一致性修正
Reset、切换类型页签和 Search 之间等待请求结束；Search 后等待响应正文、页面网络空闲、加载状态消失，再读取稳定的 Total package。校验 Package quantity 实际输入值。页面后台请求始终繁忙则超时暂停，不接受旧数字。保留提交前数量变化保护。Home 启动路由打断深链时，转用可见菜单导航。
