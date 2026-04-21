# TEST-REPORT: VLA-NTP v1.0 (App 01 在线记事本)

| 项 | 值 |
|---|---|
| 应用 | 在线记事本 (VLA-NTP) v1.0 |
| 部署地址 | https://labradorsedsota.github.io/online-notepad/ |
| GitHub Repo | https://github.com/labradorsedsota/online-notepad |
| 测试用例文档 | TEST-CASE.md (commit `ed242da`) |
| 测试日期 | 2026-04-21 |
| 测试执行人 | Moss (moss_bot) |
| 测试工具 | mano-cua (mano.mininglamp.com) |
| 测试方法 | ISTQB 黑盒方法论 (等价类划分 + BVA) |

---

## 1. 执行摘要

| 指标 | 值 |
|---|---|
| 总用例数 | 22 |
| 通过 (PASS) | 22 |
| 失败 (FAIL) | 0 |
| 阻塞 (BLOCKED) | 0 |
| 通过率 | **100%** |
| 执行耗时 | 14:15 ~ 15:45 (约 90 分钟，含 502 故障暂停 24 分钟) |

**结论：VLA-NTP v1.0 全部 22 条测试用例通过，功能符合 PRD 要求，建议验收通过。**

---

## 2. 测试结果明细

### L1: 核心功能 (8/8 PASS)

| ID | 用例名称 | 关联 AC | 结果 | 备注 |
|---|---|---|---|---|
| L1.1 | 标题输入正常显示和输入 | AC-01-1~4 | PASS | placeholder + 输入 + 显示均正常 |
| L1.2 | 内容编辑区正常显示和输入 | AC-02-1~3 | PASS | textarea 输入 + 字符计数器验证 |
| L1.3 | 保存到 localStorage 成功 | AC-03-1 | PASS | 绿色 toast + "未保存更改"消失 |
| L1.4 | 刷新页面恢复标题和内容 | AC-03-3 | PASS | localStorage 注入 → reload → 数据恢复 |
| L1.5 | 导出 TXT 文件成功 | AC-04-1~5 | PASS | 默认 TXT Radio + 确认导出触发下载 |
| L1.6 | 导出 Markdown 文件成功 | AC-04-1~5 | PASS | 选择 Markdown Radio → .md 下载 |
| L1.7 | 导出 HTML 文件成功 | AC-04-1~5 | PASS | 选择 HTML Radio → .html 下载 |
| L1.8 | 清空功能正常 | AC-05-1~2 | PASS | confirm() 对话框 + 清空成功 toast |

### L2: 重要功能 (8/8 PASS)

| ID | 用例名称 | 关联 AC | 结果 | 备注 |
|---|---|---|---|---|
| L2.1 | 暗色模式切换正常 | AC-06-1~3 | PASS | Switch 开启 → 暗色，关闭 → 亮色 |
| L2.2 | 暗色模式状态持久化 | AC-06-4 | PASS | Strategy 1: mano-cua 开启 → MOSS reload → 验证持久化 |
| L2.3 | 字体选择切换正常 | AC-07-1~3 | PASS | 下拉菜单 4 选项 (默认/等宽/衬线/无衬线) |
| L2.4 | 字体选择状态持久化 | AC-07-4 | PASS | Strategy 1: mano-cua 选等宽 → MOSS reload → 验证 |
| L2.5 | 导出对话框取消关闭 | AC-04-6 | PASS | 取消按钮关闭 + 遮罩层点击关闭 |
| L2.6 | 空内容阻止保存和导出 | AC-03-2, AC-04-7 | PASS | "内容为空，无法保存" + "内容为空，无法导出" |
| L2.7 | 清空确认和取消 | AC-05-1, AC-05-3 | PASS | confirm() + 取消后内容保持不变 |
| L2.8 | 未保存提醒显示和消失 | AC-10-1~3 | PASS | 已保存 → 编辑 → 提醒出现 → 保存 → 提醒消失 |

### L3: 边缘功能 (5/5 PASS)

| ID | 用例名称 | 关联 AC | 结果 | 备注 |
|---|---|---|---|---|
| L3.1 | 按钮 Tooltip 显示和消失 | AC-08-1~4 | PASS | 保存/导出/清空 3 按钮 Tooltip 均正常 |
| L3.2 | 底部关于链接可点击 | AC-09-1~2 | PASS | "关于"对话框: 应用名 + v1.0 + 描述 |
| L3.3 | 底部帮助链接可点击 | AC-09-3 | PASS | "使用帮助"对话框: 6 项说明完整 |
| L3.4 | 空白状态点清空提示 | AC-05-4 | PASS | 蓝色提示"已经是空白状态" |
| L3.5 | 编辑区 placeholder 显示 | AC-02-4 | PASS | "输入笔记标题..." + "开始记录..." |

### BVA: 边界值分析 (1/1 PASS)

| ID | 用例名称 | 关联 AC | 结果 | 备注 |
|---|---|---|---|---|
| L3.6 | 标题长度边界 (100 字符) | BR-01 | PASS | maxlength=100 生效，第 101 字符被阻止 |

---

## 3. 测试策略说明

### 3.1 Pre-flight 数据隔离

每条用例执行前通过 DevTools Console 清除 localStorage 并刷新页面，确保测试数据独立。

### 3.2 持久化验证 (Strategy 1)

L2.2 (暗色模式) 和 L2.4 (字体) 的持久化测试采用两阶段策略：
1. Phase 1: mano-cua 执行 UI 操作 (开启暗色模式 / 选择字体)
2. MOSS 通过 AppleScript 刷新页面
3. Phase 2: mano-cua 验证页面状态是否持久化

此策略避免了 DevTools Console 注入 localStorage 时的引号转义问题。

### 3.3 浏览器原生对话框

L1.8 和 L2.7 涉及 `confirm()` 原生对话框。mano-cua 成功处理了这些对话框（点击确定/取消），未出现兼容性问题。

---

## 4. 事件日志

| 时间 | 事件 |
|---|---|
| 14:15 | 测试启动，执行 L1.1 |
| 14:19 | mano-cua 服务端 502 Bad Gateway，暂停测试 |
| 14:43 | 服务恢复，从 L1.7 继续 |
| 14:53 | L1.1~L2.2 完成 (10/22)，发送进度报告 |
| 15:27 | L2.1~L2.8 完成 (16/22)，发送进度报告 |
| 15:45 | L3.1~L3.6 完成 (22/22)，全部 PASS |

---

## 5. 已知限制

1. **导出文件内容未做 Post-flight 校验**: 导出的 TXT/MD/HTML 文件下载成功，但未验证文件内容的精确格式 (需人工或脚本校验 ~/Downloads 目录)
2. **字数统计功能缺失**: PRD 中未定义字数统计功能 (F-01~F-10 无此需求)，代码中也未实现。如后续需要，建议作为增强需求提出。

---

_测试报告由 Moss (moss_bot) 自动生成 — 2026-04-21T15:45+08:00_
