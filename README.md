# WMS Wave Assistant v0.4.3-test

Windows 桌面测试版 / Versión de prueba para Windows。仅通过 GitHub Actions Artifacts 提供测试包；用户确认发布后才创建 Release。

## 使用

退出旧 EXE（包括黑色窗口），解压测试包并运行 WMS-Wave-Assistant.exe。助手使用独立桌面窗口，不打开默认浏览器作为管理界面，也不显示控制台。中文 / Español 可在右上角切换并保存在本机。

点击“登录 / 打开 WMS”并完成登录。选择阶段后点击预览或正式汇波，程序自动进入 Outbound → Wave Consolidation；无需手动移步。未打开 WMS 时也会自动打开；若登录过期，完成登录后再次开始。

- 单独选区（包括 TTGROUND、货架某一类型 L/H）：Zone Info 与 Virtual zone 使用 Totally included。
- 第 8–13 阶段及配置的多区域交叉选择：使用 Include。
- 不再需要勾选“仓区匹配语义核验”。旧版设置自动按新规则解释，保留包裹门槛、巷道数量、定时提醒和阶段配置。
- 所有策略按 Search 后的 Total package 判断；不足门槛时追加完整巷道；范围末尾不足时默认暂留。勾选阶段收尾可放尾批。
- 正式汇波：picking strategy → Not Operated → 最终 OK。未确认成功时保留提交记录，禁止自动重复提交。
- 第 14、15、23 阶段仍是人工节点；09:00 / 12:00 仍是程序运行期间的提醒，并非无人值守定时放单。

助手窗口与受控 WMS 浏览器是两个窗口。WMS 登录和实际操作使用内置 Chromium。助手桌面窗口采用 pywebview / Windows WebView2；需要 Windows x64 和 Microsoft Edge WebView2 Runtime。若电脑缺少 Runtime，请使用 Microsoft 官方安装程序。关闭助手会停止后续任务并关闭受控浏览器；已发出的提交先等待回执或保留待核对记录。

## Español

Cierre la versión anterior y ejecute WMS-Wave-Assistant.exe. El asistente se abre como una aplicación de escritorio independiente. Seleccione Español en la esquina superior derecha.

Inicie sesión en la ventana WMS. Al iniciar una etapa, el programa abre Wave Consolidation automáticamente. Las zonas individuales usan Totally included; las selecciones entre zonas usan Include. No se requiere marcar una verificación manual. Se conservan los umbrales y horarios existentes.

La ventana del asistente y el navegador WMS son independientes. Se requiere Windows x64 con Microsoft Edge WebView2 Runtime. Los horarios son avisos mientras la aplicación está abierta. Las etapas 14, 15 y 23 siguen siendo manuales.

## 验证与构建

GitHub Actions 使用 Windows 构建 EXE，运行规则、模拟浏览器、语言和导航测试，随后验证打包 Chromium 与原生桌面窗口。模拟测试不等于真实 WMS 全流程已通过，实际测试由用户完成。测试包位于对应 Actions 运行的 Artifacts。

源码运行：安装 requirements.txt，执行 python run.py。--no-open 可用于本地开发服务。Windows 打包：python build.py。

本机数据默认位于用户目录 .wms-wave-assistant。源码包不包含登录状态、仓库订单或本机配置。

## v0.4.2-test

- 已在真实 WMS 验证到策略下拉框选择 Clean-up Multis、Not Operated 勾选、最终 OK 启用；取消弹窗，未提交真实波次。
- 修复前置确认与策略弹窗在退出动画中短暂同时存在导致的误报，等待过渡结束后再校验。
- 深链接被重定向到 Home 时，自动通过 Outbound → Wave → Wave Consolidation 进入。
- 新增双弹窗延迟退出及菜单导航回归测试；最终提交和成功提示仍须由用户测试。

## v0.4.1-test

修复 Select pick strategy 标题与下拉占位文字同名导致的严格定位冲突。按弹窗标题定位，并通过策略下拉框精确选项选择，核对选择后再选 Not Operated 单选项和最终 OK。仅供用户测试，未发布 Release。


## v0.4.3-test
- WMS 必须使用 English；助手保留中文/西语，首页明确提示。
- 顺序：暂存爆品（人工）→ Clean-up Single → Clean-up Multis → 核验剩余 → 取消暂存（人工）→ Top-selling。
- 默认货架按完整范围一次查询、一次汇波：Pallet 124–131，Box 130–179，Bin 178–191（含完整混合边界巷道）。Zone Info Include 与对应 Virtual zone Totally included 同时筛选取交集；L/H 分开。混区 Virtual zone 保持 Include。
- 旧本机阶段配置首次升级自动迁移为完整范围；阶段内部 ID 保留以兼容记录，显示步骤按新顺序排列。
- 点击正式汇波不再弹第二次确认。阶段成功结束后可直接点“下一个”执行下一阶段（不自动连跑）；人工节点显示操作提示，未知提交仍锁定。
- 未发布 Release，仅提供 Actions 测试包。
