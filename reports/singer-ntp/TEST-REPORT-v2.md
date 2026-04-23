# Singer 在线记事本 — TEST-REPORT v2

## 1. 文档信息

| 项 | 值 |
|---|---|
| 应用 | Singer 在线记事本 |
| Repo | labradorsedsota/online-notepad-singer |
| 部署 URL | https://labradorsedsota.github.io/online-notepad-singer/ |
| 测试轮次 | v2（应用更新后重测） |
| 测试日期 | 2026-04-23 |
| 测试工具 | mano-cua (VLA) |
| 测试执行 | MOSS |
| 监督 | Pichai |
| TEST-CASE 版本 | singer-ntp-v1.0 |

## 2. 执行摘要

| 指标 | 值 |
|---|---|
| 总用例数 | 25 |
| PASS | 14 |
| FAIL | 10 |
| BLOCKED | 1 |
| 通过率 | 56% |
| 新发现 Bug | 3 个 |
| 旧 Bug 未修复 | 3 个 |

**与上轮对比：** 上轮（v1）通过率约 60%，本轮 56%。旧 Bug 3 个（BUG-001、BUG-002、BUG-005 暗色模式）均未修复，新增 3 个 Bug（导出崩溃、清空确定不生效、清空取消异常），整体质量下降。

## 3. 全量结果

### L1 核心功能 (9 条: 2 PASS / 7 FAIL)

| ID | 用例名 | 结果 | 关联 AC | 备注 |
|---|---|---|---|---|
| L1.1 | 标题输入正常显示和输入 | **FAIL** | AC-01-1, AC-01-2, AC-01-3 | BUG-002: 标题输入框高度为 0，文字不可见。VLA 观察到标题区域极窄（thin area），输入后文字不显示 |
| L1.2 | 内容编辑区正常显示和输入 | **PASS** | AC-02-1, AC-02-2, AC-02-3 | 编辑区正常显示、可输入、支持多行 |
| L1.3 | 保存到 localStorage 成功 | **PASS** | AC-03-1 | 点击保存后出现绿色 "保存成功" toast |
| L1.4 | 刷新页面恢复标题和内容 | **FAIL** | AC-03-3, AC-03-4 | BUG-001: localStorage key 互换。title 值存入 vla-ntp-content，content 值存入 vla-ntp-title，reload 后数据交叉显示 |
| L1.4b | 端到端保存-恢复 | **FAIL** | AC-03-3, AC-03-4 | 同 BUG-001。Console 验证：vla-ntp-title 存储内容文本，vla-ntp-content 存储标题文本 |
| L1.5 | 导出 TXT 文件成功 | **FAIL** | AC-04-1, AC-04-2, AC-04-3 | **新 Bug (BUG-003):** 对话框正常弹出、格式选项正确，但点击确认导出后红色 "导出失败，请重试" 错误。上轮此用例 PASS |
| L1.6 | 导出 Markdown 文件成功 | **FAIL** | AC-04-4 | BUG-003: 同 L1.5，Markdown 导出同样失败 |
| L1.7 | 导出 HTML 文件成功 | **FAIL** | AC-04-5 | BUG-003: 同 L1.5，HTML 导出同样失败 |
| L1.8 | 清空功能正常 | **FAIL** | AC-05-1, AC-05-2 | **新 Bug (BUG-004):** 确认对话框正常弹出（文案正确），点击确定后内容不清除，仍显示原文。多次独立测试均复现 |

### L2 重要功能 (9 条: 7 PASS / 2 FAIL)

| ID | 用例名 | 结果 | 关联 AC | 备注 |
|---|---|---|---|---|
| L2.1 | 暗色模式切换正常 | **FAIL** | AC-06-1, AC-06-2 | **旧 Bug (BUG-005，同上轮 BUG-003):** toggle 开关点击后变蓝（视觉状态变化），但页面背景不变暗，CSS 未切换。注意 L2.2 持久化测试 PASS——通过 localStorage 注入 dark 后 reload 可以正确加载暗色模式，说明 CSS 存在但 toggle 点击事件未正确触发样式切换。与上轮 BUG-003（setAttribute 被注释）为同一问题 |
| L2.2 | 暗色模式状态持久化 | **PASS** | AC-06-3 | localStorage 注入 vla-ntp-theme=dark 后 reload，页面正确加载暗色模式（深色背景、浅色文字），toggle 显示开启 |
| L2.3 | 字体选择切换正常 | **PASS** | AC-07-1, AC-07-2 | 下拉菜单包含默认/等宽/衬线/无衬线四个选项，切换后编辑区文字立即变化 |
| L2.3b | 等宽字体特征验证 | **PASS** | AC-07-2 强化 | 4 行测试文本（ABCDEFGHIJ/1234567890/WWWWWWWWWW/iiiiiiiiii）在等宽模式下右边缘对齐，W 和 i 等宽 |
| L2.4 | 字体选择状态持久化 | **PASS** | AC-07-3 | localStorage 注入 vla-ntp-font=mono 后 reload，下拉菜单显示"等宽"，文字以等宽字体渲染 |
| L2.5 | 导出对话框取消关闭 | **PASS** | AC-04-6 | 点击"取消"关闭对话框 ✓；点击遮罩外部关闭对话框 ✓；均不触发导出 |
| L2.6 | 空内容阻止保存和导出 | **PASS** | AC-03-2, AC-04-7 | 空内容点保存：橙色 toast "内容为空，无法保存" ✓；空内容点导出：橙色 toast "内容为空，无法导出"，不弹对话框 ✓ |
| L2.7 | 清空确认和取消 | **FAIL** | AC-05-1, AC-05-3 | **新 Bug (BUG-006):** AC-05-1 PASS（确认对话框正常弹出）。AC-05-3 FAIL——点击"取消"按钮后内容仍被清空，显示 "已清空" toast。两次独立 VLA 测试均复现此行为。取消按钮与确定按钮效果相同 |
| L2.8 | 未保存提醒显示和消失 | **PASS** | AC-10-1, AC-10-2, AC-10-3 | 未修改时无 alert ✓；修改后黄色 "您有未保存的更改" alert 出现 ✓；保存后 alert 消失 + 绿色 "保存成功" toast ✓ |

### L3 边缘功能 (7 条: 5 PASS / 1 PASS / 1 BLOCKED)

| ID | 用例名 | 结果 | 关联 AC | 备注 |
|---|---|---|---|---|
| L3.1 | 按钮 Tooltip 显示和消失 | **PASS** | AC-08-1~4 | 保存→"保存笔记到浏览器" ✓；导出→"导出为文件" ✓；清空→"清空所有内容" ✓；移出后消失 ✓ |
| L3.2 | 底部关于链接可点击 | **PASS** | AC-09-1, AC-09-2 | 关于弹窗显示"在线记事本 v1.0"、应用描述、GitHub 链接 |
| L3.3 | 底部帮助链接可点击 | **PASS** | AC-09-3 | 帮助弹窗显示 6 项功能说明（记录/保存/导出/清空/暗色模式/字体） |
| L3.4 | 空白状态点清空提示 | **PASS** | AC-05-4 | 空白状态点清空，蓝色 toast "已经是空白状态"，无确认对话框 |
| L3.5 | 编辑区 placeholder 显示 | **PASS** | AC-02-4 | 空白状态下灰色 "开始记录..." placeholder 正确显示 |
| L3.6 | BVA 标题最大长度边界 | **PASS** | BR-01 | 注入 100 字符 A 后尝试追加 XXXXX，console 确认 title.value.length=100，maxlength 生效 |
| L3.7 | BVA 导出文件名特殊字符 | **BLOCKED** | BR-04 | 导出功能全线崩溃（BUG-003），无法触发文件下载，无法验证文件名处理 |

## 4. Bug 清单

### 旧 Bug（未修复，3 个）

| Bug ID | 标题 | 严重度 | 影响用例 | 上轮状态 | 本轮状态 |
|---|---|---|---|---|---|
| BUG-001 | localStorage save/load key 互换 | Critical | L1.4, L1.4b | FAIL | **仍 FAIL** |
| BUG-002 | 标题输入框高度为 0，文字不可见 | Critical | L1.1 | FAIL | **仍 FAIL** |
| BUG-005 | 暗色模式 toggle 点击无效（同上轮 BUG-003） | Major | L2.1 | FAIL | **仍 FAIL**。toggle 视觉状态正确变化（变蓝），但页面 CSS 不切换。localStorage 注入 dark + reload 可正确加载暗色模式（L2.2 PASS），说明 toggle 事件绑定有问题，非 CSS 缺失。与上轮 BUG-003（setAttribute 被注释）为同一问题 |

### 新 Bug（3 个）

| Bug ID | 标题 | 严重度 | 影响用例 | 描述 |
|---|---|---|---|---|
| BUG-003 | 导出功能全线崩溃 | Critical | L1.5, L1.6, L1.7, L3.7 | TXT/Markdown/HTML 三种格式导出均报 "导出失败，请重试"。上轮导出全 PASS。本次更新引入的回归 Bug |
| BUG-004 | 清空确定后内容不清除 | Major | L1.8 | 点击清空→确定后，内容区不清空，文字仍在。多次复现。注意：L2.7 测试中曾观察到一次清空成功（green "已清空" toast），行为不一致，可能为间歇性 Bug |
| BUG-006 | 清空取消按钮行为异常 | Major | L2.7 | 确认对话框中的"取消"按钮点击后仍执行清空操作，行为与"确定"按钮相同。两次独立 VLA 测试均复现 |

## 5. 测试环境说明

| 项 | 值 |
|---|---|
| 浏览器 | Google Chrome (macOS) |
| OS | macOS (Darwin 24.2.0 arm64) |
| 测试工具 | mano-cua VLA (1280x720) |
| 后端稳定性 | 间歇性不稳定——session 创建约 50% 概率挂起需重试，部分测试遭遇 502/404 错误 |

## 6. 结论

本轮更新未修复任何旧 Bug（3 个），反而引入 3 个新 Bug，导致整体质量下降。导出功能全线崩溃（BUG-003）是最严重的回归问题。清空功能同时存在两个 Bug（确定不清空 BUG-004 + 取消也清空 BUG-006），行为不一致。暗色模式 toggle 事件绑定失效（BUG-005）为上轮遗留问题，仍未修复。

**建议：** 优先修复 BUG-001 (key 互换) 和 BUG-003 (导出崩溃)，这两个 Critical Bug 影响核心功能。

---

_报告生成时间: 2026-04-23 16:48 GMT+8_
_执行方: MOSS | 监督: Pichai_
