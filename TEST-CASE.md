# TEST-CASE.md — App 01: 在线记事本（VLA-NTP v1）

## 1. 文档信息

| 项 | 值 |
|---|---|
| 项目 | VLA-NTP（App 01 在线记事本） |
| 版本 | v1.0 |
| 编写人 | Moss (QA) |
| 日期 | 2026-04-21 |
| 状态 | 待 PM 确认 |
| PRD 版本 | v1.0 (2026-04-13, Pichai) |
| 被测页面 | https://labradorsedsota.github.io/online-notepad-singer/ |
| 测试工具 | mano-cua（优先） |
| 执行规范 | mano-cua-execution-spec v1.6 |
| 设计规范 | test-case-design-spec v1.0 |

---

## 2. 测试范围

覆盖 PRD v1.0 验收汇总全部 21 个验收点 + 1 条 BVA 增补用例：

| 层级 | 数量 | 说明 |
|------|------|------|
| L1 — 核心功能 | 8 | 阻断发布 |
| L2 — 重要功能 | 8 | 影响体验 |
| L3 — 边缘功能 | 5 + 1(BVA) | 可降级 |
| **合计** | **22** | |

---

## 3. 测试环境

| 项 | 值 |
|---|---|
| 浏览器 | Google Chrome（最新稳定版） |
| 操作系统 | macOS (arm64) |
| 测试工具 | mano-cua |
| 被测 URL | https://labradorsedsota.github.io/online-notepad-singer/ |
| 网络 | 本地网络 → GitHub Pages |
| localStorage 前缀 | `vla-ntp-` |
| localStorage Keys | `vla-ntp-title`, `vla-ntp-content`, `vla-ntp-theme`, `vla-ntp-font` |

---

## 4. Fixture 清单

存放路径：`test/fixtures/online-notepad-singer/`

| 文件名 | 用途 | 内容摘要 |
|--------|------|---------|
| `title-normal.txt` | 通用标题 | `MOSS 测试笔记 2026`（10 字符） |
| `content-normal.txt` | 通用内容 | 3 段中文文本，含特殊字符（引号、&） |
| `title-boundary-100.txt` | BVA — 恰好 100 字符标题 | 100 个中文字符 |

### Fixture 内容

**title-normal.txt:**
```
MOSS 测试笔记 2026
```

**content-normal.txt:**
```
这是一条测试笔记的第一段内容。用于验证在线记事本的基本编辑功能。

系统演算显示，此段文本的存储概率为 99.97%。

第三段：包含特殊字符测试 — 引号"双引号"、'单引号'、&符号。
```

**title-boundary-100.txt:**
```
测试标题边界值一二三四五六七八九十一二三四五六七八九十一二三四五六七八九十一二三四五六七八九十一二三四五六七八九十一二三四五六七八九十一二三四五六七八九十一二三四五六七八九十一二三四五六七八九十测试完
```

---

## 5. 通用 Pre-flight 脚本

### 5a. 清除 localStorage + 打开页面 + 窗口最大化（EMPTY 状态）

```bash
# Step 1: 打开目标页面
open -a "Google Chrome" "https://labradorsedsota.github.io/online-notepad-singer/"
sleep 3

# Step 2: 窗口最大化
osascript -e '
tell application "Finder"
    set _b to bounds of window of desktop
    tell application "Google Chrome"
        set bounds of front window to {0, 0, item 3 of _b, item 4 of _b}
    end tell
end tell'

# Step 3: 清除 localStorage 并刷新
osascript -e '
tell application "Google Chrome"
    tell active tab of front window
        execute javascript "localStorage.clear(); location.reload();"
    end tell
end tell'
sleep 2
```

### 5b. 注入保存数据（CUSTOM 状态 — 含标题+内容已保存）

在 5a 基础上继续：
```bash
osascript -e '
tell application "Google Chrome"
    tell active tab of front window
        execute javascript "
            localStorage.setItem(\"vla-ntp-title\", \"MOSS 测试笔记 2026\");
            localStorage.setItem(\"vla-ntp-content\", \"这是一条测试笔记的第一段内容。用于验证在线记事本的基本编辑功能。\\n\\n系统演算显示，此段文本的存储概率为 99.97%。\\n\\n第三段：包含特殊字符测试。\");
            location.reload();
        "
    end tell
end tell'
sleep 2
```

### 5c. 注入暗色模式状态（CUSTOM 状态 — 暗色模式已保存）

```bash
osascript -e '
tell application "Google Chrome"
    tell active tab of front window
        execute javascript "
            localStorage.setItem(\"vla-ntp-theme\", \"dark\");
            location.reload();
        "
    end tell
end tell'
sleep 2
```

### 5d. 注入字体选择状态（CUSTOM 状态 — 等宽字体已保存）

```bash
osascript -e '
tell application "Google Chrome"
    tell active tab of front window
        execute javascript "
            localStorage.setItem(\"vla-ntp-font\", \"mono\");
            location.reload();
        "
    end tell
end tell'
sleep 2
```

### 5e. 关闭测试标签页（Post-flight）

```bash
osascript -e '
tell application "Google Chrome"
    set matchPath to "labradorsedsota.github.io/online-notepad-singer"
    set closedCount to 0
    repeat with w in windows
        set tabList to tabs of w
        repeat with i from (count of tabList) to 1 by -1
            if URL of item i of tabList contains matchPath then
                delete item i of tabList
                set closedCount to closedCount + 1
            end if
        end repeat
    end repeat
    return closedCount
end tell'
```

---

## 6. 测试用例

---

### L1.1 标题输入正常显示和输入

| 项 | 值 |
|---|---|
| mosstid | `vla-ntp-v1-L1.1-01` |
| 关联 AC | AC-01-1, AC-01-2 |
| 设计技术 | PRD 追溯 |
| 数据依赖 | EMPTY |
| 冲突标记 | 无 |
| 前置策略 | 策略一 |
| fixture | `title-normal.txt` |

**Pre-flight：** 执行脚本 5a（清除 localStorage + 打开页面 + 窗口最大化）

**任务描述：**
```
在当前页面上：
1. 观察页面顶部标题输入框，确认输入框已显示且 placeholder 为"输入笔记标题…"
2. 点击标题输入框，输入文字"MOSS 测试笔记 2026"
3. 确认输入的文字实时显示在标题输入框中
仅在当前页面操作，不要导航到其他网址。
```

**Expected Results（逐条）：**
1. 标题输入框显示在页面顶部，placeholder 为"输入笔记标题…"（AC-01-1）
2. 在标题框中输入文字后，标题文字"MOSS 测试笔记 2026"实时显示在输入框中（AC-01-2）

---

### L1.2 内容编辑区正常显示和输入

| 项 | 值 |
|---|---|
| mosstid | `vla-ntp-v1-L1.2-01` |
| 关联 AC | AC-02-1, AC-02-2 |
| 设计技术 | PRD 追溯 |
| 数据依赖 | EMPTY |
| 冲突标记 | 无 |
| 前置策略 | 策略一 |
| fixture | `content-normal.txt` |

**Pre-flight：** 执行脚本 5a

**任务描述：**
```
在当前页面上：
1. 观察卡片包裹的文本编辑区域，确认编辑区已显示且 placeholder 为"开始记录…"
2. 点击编辑区域，输入以下文字：
   "这是一条测试笔记的第一段内容。用于验证在线记事本的基本编辑功能。"
3. 按回车两次换行，继续输入："系统演算显示，此段文本的存储概率为 99.97%。"
4. 观察编辑区高度是否随内容自动扩展
仅在当前页面操作，不要导航到其他网址。
```

**Expected Results（逐条）：**
1. 页面显示卡片包裹的文本编辑区域，placeholder 为"开始记录…"（AC-02-1）
2. 输入的文字实时显示在编辑区中（AC-02-2）
3. 编辑区自动扩展高度以容纳内容，不出现内部滚动条，最小高度 300px（AC-02-2）

---

### L1.3 保存到 localStorage 成功

| 项 | 值 |
|---|---|
| mosstid | `vla-ntp-v1-L1.3-01` |
| 关联 AC | AC-03-1 |
| 设计技术 | PRD 追溯 |
| 数据依赖 | EMPTY |
| 冲突标记 | 无 |
| 前置策略 | 策略一 |
| fixture | `title-normal.txt`, `content-normal.txt` |

**Pre-flight：** 执行脚本 5a

**任务描述：**
```
在当前页面上：
1. 点击标题输入框，输入"MOSS 测试笔记 2026"
2. 点击编辑区域，输入"这是一条测试笔记的第一段内容。用于验证在线记事本的基本编辑功能。"
3. 点击"💾 保存"按钮
4. 观察是否出现成功提示消息"保存成功"
仅在当前页面操作，不要导航到其他网址。
```

**Expected Results（逐条）：**
1. 点击"保存"按钮后，数据保存到 localStorage（AC-03-1）
2. 页面顶部弹出成功提示消息"保存成功"（AC-03-1）

**Post-flight 验证（MOSS 执行）：**
```bash
osascript -e '
tell application "Google Chrome"
    tell active tab of front window
        execute javascript "
            JSON.stringify({
                title: localStorage.getItem(\"vla-ntp-title\"),
                content: localStorage.getItem(\"vla-ntp-content\")
            })
        "
    end tell
end tell'
```
验证返回值中 title 和 content 不为 null。

---

### L1.4 刷新页面恢复标题和内容

| 项 | 值 |
|---|---|
| mosstid | `vla-ntp-v1-L1.4-01` |
| 关联 AC | AC-01-3, AC-02-3, AC-03-3 |
| 设计技术 | PRD 追溯 |
| 数据依赖 | CUSTOM — 需要 localStorage 中已有保存的标题和内容 |
| 冲突标记 | 无 |
| 前置策略 | 策略二 |
| fixture | `title-normal.txt`, `content-normal.txt` |

**Pre-flight：** 执行脚本 5a → 5b（清除后注入保存数据，页面刷新后加载已保存数据）

**任务描述：**
```
在当前页面上（页面已加载完毕，不要刷新）：
1. 观察标题输入框，确认其中显示"MOSS 测试笔记 2026"
2. 观察编辑区，确认其中显示已保存的笔记内容（以"这是一条测试笔记"开头）
仅在当前页面操作，不要导航到其他网址。
```

**Expected Results（逐条）：**
1. 标题框恢复上次保存的标题内容"MOSS 测试笔记 2026"（AC-01-3）
2. 编辑区恢复上次保存的内容（AC-02-3）
3. 关闭浏览器后重新打开页面，标题和内容恢复（AC-03-3，由 Pre-flight 注入 localStorage 模拟）

---

### L1.5 导出 TXT 文件成功

| 项 | 值 |
|---|---|
| mosstid | `vla-ntp-v1-L1.5-01` |
| 关联 AC | AC-04-1, AC-04-2, AC-04-3 |
| 设计技术 | PRD 追溯 |
| 数据依赖 | CUSTOM — 需要编辑器中有标题和内容 |
| 冲突标记 | 无 |
| 前置策略 | 策略二（注入内容）+ 策略一（导出操作是被测功能） |
| fixture | `title-normal.txt`, `content-normal.txt` |

**Pre-flight：** 执行脚本 5a，然后注入内容到编辑器（不通过 localStorage 保存，直接写入 DOM）：
```bash
osascript -e '
tell application "Google Chrome"
    tell active tab of front window
        execute javascript "
            document.getElementById(\"titleInput\").value = \"MOSS 测试笔记 2026\";
            document.getElementById(\"contentArea\").value = \"这是一条测试笔记的第一段内容。用于验证在线记事本的基本编辑功能。\\n\\n系统演算显示，此段文本的存储概率为 99.97%。\";
            document.getElementById(\"contentArea\").dispatchEvent(new Event(\"input\"));
        "
    end tell
end tell'
sleep 1
```

**任务描述：**
```
在当前页面上（标题和内容已填写）：
1. 点击"📤 导出"按钮
2. 观察是否弹出导出确认对话框
3. 在对话框中，观察格式选择（Radio），确认有 TXT / Markdown / HTML 三个选项，且默认选中 TXT
4. 保持 TXT 格式选中，点击"确认导出"按钮
5. 观察是否触发文件下载
仅在当前页面操作，不要导航到其他网址。
```

**Expected Results（逐条）：**
1. 点击"导出"按钮后，弹出导出确认对话框（AC-04-1）
2. 对话框显示格式选择 Radio：TXT / Markdown / HTML，默认选中 TXT（AC-04-2）
3. 选择 TXT 格式并点击"确认导出"后，触发 `.txt` 文件下载（AC-04-3）
4. 下载的文件名为标题内容（即"MOSS 测试笔记 2026.txt"），文件内容为纯文本（AC-04-3）

---

### L1.6 导出 Markdown 文件成功

| 项 | 值 |
|---|---|
| mosstid | `vla-ntp-v1-L1.6-01` |
| 关联 AC | AC-04-4 |
| 设计技术 | PRD 追溯 |
| 数据依赖 | CUSTOM — 编辑器中需有标题和内容 |
| 冲突标记 | 无 |
| 前置策略 | 策略二（注入内容）+ 策略一（导出是被测功能） |
| fixture | `title-normal.txt`, `content-normal.txt` |

**Pre-flight：** 同 L1.5 Pre-flight（脚本 5a + DOM 注入）

**任务描述：**
```
在当前页面上（标题和内容已填写）：
1. 点击"📤 导出"按钮
2. 在弹出的导出确认对话框中，选择"Markdown"格式（点击 Markdown Radio 按钮）
3. 点击"确认导出"按钮
4. 观察是否触发文件下载
仅在当前页面操作，不要导航到其他网址。
```

**Expected Results（逐条）：**
1. 选择 Markdown 格式并确认后，触发 `.md` 文件下载（AC-04-4）
2. 下载的文件中，标题作为一级标题（`# 标题`），正文在标题下方（AC-04-4）

---

### L1.7 导出 HTML 文件成功

| 项 | 值 |
|---|---|
| mosstid | `vla-ntp-v1-L1.7-01` |
| 关联 AC | AC-04-5 |
| 设计技术 | PRD 追溯 |
| 数据依赖 | CUSTOM — 编辑器中需有标题和内容 |
| 冲突标记 | 无 |
| 前置策略 | 策略二（注入内容）+ 策略一（导出是被测功能） |
| fixture | `title-normal.txt`, `content-normal.txt` |

**Pre-flight：** 同 L1.5 Pre-flight（脚本 5a + DOM 注入）

**任务描述：**
```
在当前页面上（标题和内容已填写）：
1. 点击"📤 导出"按钮
2. 在弹出的导出确认对话框中，选择"HTML"格式（点击 HTML Radio 按钮）
3. 点击"确认导出"按钮
4. 观察是否触发文件下载
仅在当前页面操作，不要导航到其他网址。
```

**Expected Results（逐条）：**
1. 选择 HTML 格式并确认后，触发 `.html` 文件下载（AC-04-5）
2. 下载的文件包含完整 HTML 结构：`<html><head><title>标题</title></head><body>...`（AC-04-5）
3. 文件中正文段落用 `<p>` 包裹，按换行分段（AC-04-5）

---

### L1.8 清空功能正常

| 项 | 值 |
|---|---|
| mosstid | `vla-ntp-v1-L1.8-01` |
| 关联 AC | AC-05-1, AC-05-2 |
| 设计技术 | PRD 追溯 |
| 数据依赖 | CUSTOM — 编辑器中需有标题和内容，且已保存到 localStorage |
| 冲突标记 | ⚠ 写操作 — 清除 localStorage 数据 |
| 前置策略 | 策略二（注入已保存内容）+ 策略一（清空操作是被测功能） |
| fixture | `title-normal.txt`, `content-normal.txt` |

**Pre-flight：** 执行脚本 5a → 5b（注入已保存数据，刷新后页面显示内容）

**任务描述：**
```
在当前页面上（标题和内容已填写）：
1. 确认标题输入框和编辑区中有内容
2. 点击"🗑 清空"按钮
3. 观察是否出现确认提示"确定要清空所有内容吗？此操作不可撤销"
4. 点击"确定"按钮确认清空
5. 观察标题输入框和编辑区是否已清空
仅在当前页面操作，不要导航到其他网址。
```

**Expected Results（逐条）：**
1. 点击"清空"按钮后，显示确认提示"确定要清空所有内容吗？此操作不可撤销"（AC-05-1）
2. 用户确认后，标题和内容清空（AC-05-2）
3. localStorage 中的保存数据也清除（AC-05-2，Post-flight 验证）

**Post-flight 验证（MOSS 执行）：**
```bash
osascript -e '
tell application "Google Chrome"
    tell active tab of front window
        execute javascript "
            JSON.stringify({
                title: localStorage.getItem(\"vla-ntp-title\"),
                content: localStorage.getItem(\"vla-ntp-content\")
            })
        "
    end tell
end tell'
```
验证 title 和 content 均为 null。

**注意：** F-05 清空使用浏览器原生 `confirm()` 对话框，mano-cua 需能处理原生对话框交互。

---

### L2.1 暗色模式切换正常

| 项 | 值 |
|---|---|
| mosstid | `vla-ntp-v1-L2.1-01` |
| 关联 AC | AC-06-1, AC-06-2 |
| 设计技术 | PRD 追溯 |
| 数据依赖 | EMPTY |
| 冲突标记 | 无 |
| 前置策略 | 策略一 |
| fixture | 无 |

**Pre-flight：** 执行脚本 5a（确保亮色模式初始状态）

**任务描述：**
```
在当前页面上（当前为亮色模式，背景色为浅色）：
1. 找到右上角暗色模式 Switch 开关
2. 点击 Switch 开启暗色模式
3. 观察页面是否切换为暗色模式（背景变为深色 #1C1C1E，文字变为浅色）
4. 再次点击 Switch 关闭暗色模式
5. 观察页面是否切换回亮色模式（背景恢复为浅色 #FAFAFA）
仅在当前页面操作，不要导航到其他网址。
```

**Expected Results（逐条）：**
1. 切换 Switch 为开启状态后，页面切换为暗色模式：背景深色、文字浅色（AC-06-1）
2. 切换 Switch 为关闭状态后，页面切换回亮色模式（AC-06-2）

---

### L2.2 暗色模式状态持久化

| 项 | 值 |
|---|---|
| mosstid | `vla-ntp-v1-L2.2-01` |
| 关联 AC | AC-06-3 |
| 设计技术 | PRD 追溯 |
| 数据依赖 | CUSTOM — localStorage 中需有 `vla-ntp-theme: dark` |
| 冲突标记 | 无 |
| 前置策略 | 策略二 |
| fixture | 无 |

**Pre-flight：** 执行脚本 5a → 5c（清除后注入暗色模式状态，刷新页面）

**任务描述：**
```
在当前页面上（页面已加载完毕，不要刷新）：
1. 观察页面整体配色：背景是否为深色（#1C1C1E）、文字是否为浅色
2. 观察右上角暗色模式 Switch 是否处于开启状态
仅在当前页面操作，不要导航到其他网址。
```

**Expected Results（逐条）：**
1. 刷新页面后，保持暗色模式（背景深色、文字浅色）（AC-06-3）
2. Switch 开关处于开启状态（AC-06-3）

---

### L2.3 字体选择切换正常

| 项 | 值 |
|---|---|
| mosstid | `vla-ntp-v1-L2.3-01` |
| 关联 AC | AC-07-1, AC-07-2 |
| 设计技术 | PRD 追溯 |
| 数据依赖 | CUSTOM — 编辑器中需有内容以观察字体变化 |
| 冲突标记 | 无 |
| 前置策略 | 策略二（注入内容）+ 策略一（字体切换是被测功能） |
| fixture | `content-normal.txt` |

**Pre-flight：** 执行脚本 5a，然后注入内容到编辑器：
```bash
osascript -e '
tell application "Google Chrome"
    tell active tab of front window
        execute javascript "
            document.getElementById(\"contentArea\").value = \"这是一条测试笔记的第一段内容。用于验证字体切换效果。ABCDEFG 1234567\";
            document.getElementById(\"contentArea\").dispatchEvent(new Event(\"input\"));
        "
    end tell
end tell'
```

**任务描述：**
```
在当前页面上（编辑区已有文字内容）：
1. 找到右上角字体下拉菜单（Select），确认包含字体选项
2. 选择"等宽"字体选项
3. 观察编辑区文字是否切换为等宽字体（字母和数字等宽对齐）
4. 再选择"衬线"字体选项
5. 观察编辑区文字是否切换为衬线字体
仅在当前页面操作，不要导航到其他网址。
```

**Expected Results（逐条）：**
1. 字体下拉菜单显示，至少包含：默认、等宽（Monospace）、衬线（Serif）、无衬线（Sans-serif）（AC-07-1）
2. 选择等宽字体后，编辑区文字立即切换为等宽字体（AC-07-2）
3. 选择衬线字体后，编辑区文字立即切换为衬线字体（AC-07-2）

---

### L2.4 字体选择状态持久化

| 项 | 值 |
|---|---|
| mosstid | `vla-ntp-v1-L2.4-01` |
| 关联 AC | AC-07-3 |
| 设计技术 | PRD 追溯 |
| 数据依赖 | CUSTOM — localStorage 中需有 `vla-ntp-font: mono` |
| 冲突标记 | 无 |
| 前置策略 | 策略二 |
| fixture | `content-normal.txt` |

**Pre-flight：** 执行脚本 5a → 5d（清除后注入等宽字体状态），然后注入编辑器内容以便观察字体效果：
```bash
osascript -e '
tell application "Google Chrome"
    tell active tab of front window
        execute javascript "
            document.getElementById(\"contentArea\").value = \"验证字体持久化 ABCDEFG 1234567\";
            document.getElementById(\"contentArea\").dispatchEvent(new Event(\"input\"));
        "
    end tell
end tell'
```

**任务描述：**
```
在当前页面上（页面已加载完毕，不要刷新）：
1. 观察右上角字体下拉菜单的当前选中值，是否为"等宽"
2. 观察编辑区文字是否以等宽字体显示
仅在当前页面操作，不要导航到其他网址。
```

**Expected Results（逐条）：**
1. 刷新页面后，字体下拉菜单选中值为"等宽"（AC-07-3）
2. 编辑区文字以等宽字体显示（AC-07-3）

---

### L2.5 导出对话框取消关闭

| 项 | 值 |
|---|---|
| mosstid | `vla-ntp-v1-L2.5-01` |
| 关联 AC | AC-04-6 |
| 设计技术 | PRD 追溯 |
| 数据依赖 | CUSTOM — 编辑器中需有内容以触发导出对话框 |
| 冲突标记 | 无 |
| 前置策略 | 策略二（注入内容） |
| fixture | `title-normal.txt`, `content-normal.txt` |

**Pre-flight：** 同 L1.5 Pre-flight（脚本 5a + DOM 注入标题和内容）

**任务描述：**
```
在当前页面上（标题和内容已填写）：
1. 点击"📤 导出"按钮，弹出导出确认对话框
2. 点击对话框中的"取消"按钮
3. 观察对话框是否关闭，且没有执行导出操作
4. 再次点击"📤 导出"按钮，弹出导出确认对话框
5. 点击对话框外部区域（遮罩层）
6. 观察对话框是否关闭
仅在当前页面操作，不要导航到其他网址。
```

**Expected Results（逐条）：**
1. 点击"取消"按钮后，对话框关闭，不执行导出（AC-04-6）
2. 点击对话框外部区域后，对话框关闭，不执行导出（AC-04-6）

---

### L2.6 空内容阻止保存和导出

| 项 | 值 |
|---|---|
| mosstid | `vla-ntp-v1-L2.6-01` |
| 关联 AC | AC-03-2, AC-04-7 |
| 设计技术 | PRD 追溯 |
| 数据依赖 | EMPTY |
| 冲突标记 | 无 |
| 前置策略 | 策略一 |
| fixture | 无 |

**Pre-flight：** 执行脚本 5a（确保空白状态）

**任务描述：**
```
在当前页面上（标题和内容均为空）：
1. 直接点击"💾 保存"按钮（不输入任何内容）
2. 观察是否出现警告提示"内容为空，无法保存"
3. 等待提示消失后，直接点击"📤 导出"按钮
4. 观察是否出现警告提示"内容为空，无法导出"，且不弹出导出对话框
仅在当前页面操作，不要导航到其他网址。
```

**Expected Results（逐条）：**
1. 标题和内容均为空时点击"保存"，显示警告提示"内容为空，无法保存"（AC-03-2）
2. 标题和内容均为空时点击"导出"，显示警告提示"内容为空，无法导出"，不弹出对话框（AC-04-7）

---

### L2.7 清空确认和取消

| 项 | 值 |
|---|---|
| mosstid | `vla-ntp-v1-L2.7-01` |
| 关联 AC | AC-05-1, AC-05-3 |
| 设计技术 | PRD 追溯 |
| 数据依赖 | CUSTOM — 编辑器中需有内容 |
| 冲突标记 | 无 |
| 前置策略 | 策略二（注入内容）+ 策略一（清空交互是被测功能） |
| fixture | `title-normal.txt`, `content-normal.txt` |

**Pre-flight：** 执行脚本 5a → 5b（注入已保存数据）

**任务描述：**
```
在当前页面上（标题和内容已填写）：
1. 点击"🗑 清空"按钮
2. 观察是否出现确认提示"确定要清空所有内容吗？此操作不可撤销"
3. 点击"取消"按钮
4. 观察标题和内容是否保持不变（未被清空）
仅在当前页面操作，不要导航到其他网址。
```

**Expected Results（逐条）：**
1. 点击"清空"按钮后，显示确认提示"确定要清空所有内容吗？此操作不可撤销"（AC-05-1）
2. 用户取消后，标题和内容保持不变，不做任何操作（AC-05-3）

**注意：** F-05 清空使用浏览器原生 `confirm()` 对话框。

---

### L2.8 未保存提醒显示和消失

| 项 | 值 |
|---|---|
| mosstid | `vla-ntp-v1-L2.8-01` |
| 关联 AC | AC-10-1, AC-10-2, AC-10-3 |
| 设计技术 | PRD 追溯 |
| 数据依赖 | CUSTOM — localStorage 中需有已保存数据，页面加载后显示保存内容 |
| 冲突标记 | 无 |
| 前置策略 | 策略二（注入保存数据）+ 策略一（修改和保存操作是被测功能） |
| fixture | `title-normal.txt`, `content-normal.txt` |

**Pre-flight：** 执行脚本 5a → 5b（注入已保存数据，页面显示保存的标题和内容，此时无未保存变更）

**任务描述：**
```
在当前页面上（页面已加载保存的笔记内容）：
1. 首先观察页面顶部，确认没有显示"您有未保存的更改"的提醒条（Alert）
2. 点击编辑区，在已有内容末尾输入"新增一行内容"
3. 观察页面顶部是否出现 Alert 提醒"您有未保存的更改"
4. 点击"💾 保存"按钮
5. 观察 Alert 提醒是否消失
仅在当前页面操作，不要导航到其他网址。
```

**Expected Results（逐条）：**
1. 内容未修改时（与上次保存一致），页面顶部不显示 Alert（AC-10-3）
2. 修改内容后（相对于上次保存），页面顶部显示 Alert 提醒"您有未保存的更改"（AC-10-1）
3. 点击保存后，Alert 消失（AC-10-2）

---

### L3.1 按钮 Tooltip 显示和消失

| 项 | 值 |
|---|---|
| mosstid | `vla-ntp-v1-L3.1-01` |
| 关联 AC | AC-08-1, AC-08-2, AC-08-3, AC-08-4 |
| 设计技术 | PRD 追溯 |
| 数据依赖 | ANY |
| 冲突标记 | 无 |
| 前置策略 | 策略一 |
| fixture | 无 |

**Pre-flight：** 执行脚本 5a

**任务描述：**
```
在当前页面上：
1. 将鼠标悬停在"💾 保存"按钮上，等待约 0.5 秒，观察是否出现 Tooltip 提示"保存笔记到浏览器"
2. 鼠标移出"保存"按钮区域，观察 Tooltip 是否消失
3. 将鼠标悬停在"📤 导出"按钮上，等待约 0.5 秒，观察是否出现 Tooltip 提示"导出为文件"
4. 鼠标移出
5. 将鼠标悬停在"🗑 清空"按钮上，等待约 0.5 秒，观察是否出现 Tooltip 提示"清空所有内容"
6. 鼠标移出，观察 Tooltip 是否消失
仅在当前页面操作，不要导航到其他网址。
```

**Expected Results（逐条）：**
1. 悬停在"保存"按钮上超过 300ms 后，显示 Tooltip "保存笔记到浏览器"（AC-08-1）
2. 悬停在"导出"按钮上超过 300ms 后，显示 Tooltip "导出为文件"（AC-08-2）
3. 悬停在"清空"按钮上超过 300ms 后，显示 Tooltip "清空所有内容"（AC-08-3）
4. 鼠标移出按钮区域后，Tooltip 消失（AC-08-4）

**注意：** mano-cua 需支持 hover 操作。若 mano-cua 无法精确执行 hover + 等待，标记 BLOCKED 并附诊断。

---

### L3.2 底部关于链接可点击

| 项 | 值 |
|---|---|
| mosstid | `vla-ntp-v1-L3.2-01` |
| 关联 AC | AC-09-1, AC-09-2 |
| 设计技术 | PRD 追溯 |
| 数据依赖 | ANY |
| 冲突标记 | 无 |
| 前置策略 | 策略一 |
| fixture | 无 |

**Pre-flight：** 执行脚本 5a

**任务描述：**
```
在当前页面上：
1. 滚动到页面底部，观察是否显示"关于"和"帮助"两个链接
2. 点击"关于"链接
3. 观察是否弹出对话框（Dialog）或跳转，显示关于信息
仅在当前页面操作，不要导航到其他网址。
```

**Expected Results（逐条）：**
1. 页面底部显示"关于"和"帮助"两个链接（AC-09-1）
2. 点击"关于"链接后，弹出关于对话框，显示关于信息（应用名称、版本、描述）（AC-09-2）

---

### L3.3 底部帮助链接可点击

| 项 | 值 |
|---|---|
| mosstid | `vla-ntp-v1-L3.3-01` |
| 关联 AC | AC-09-3 |
| 设计技术 | PRD 追溯 |
| 数据依赖 | ANY |
| 冲突标记 | 无 |
| 前置策略 | 策略一 |
| fixture | 无 |

**Pre-flight：** 执行脚本 5a

**任务描述：**
```
在当前页面上：
1. 滚动到页面底部，点击"帮助"链接
2. 观察是否弹出对话框，显示使用说明信息（包括记录、保存、导出、清空、暗色模式、字体相关说明）
仅在当前页面操作，不要导航到其他网址。
```

**Expected Results（逐条）：**
1. 点击"帮助"链接后，弹出帮助对话框，显示使用说明信息（AC-09-3）

---

### L3.4 空白状态点清空提示

| 项 | 值 |
|---|---|
| mosstid | `vla-ntp-v1-L3.4-01` |
| 关联 AC | AC-05-4 |
| 设计技术 | PRD 追溯 |
| 数据依赖 | EMPTY |
| 冲突标记 | 无 |
| 前置策略 | 策略一 |
| fixture | 无 |

**Pre-flight：** 执行脚本 5a（确保空白状态）

**任务描述：**
```
在当前页面上（标题和内容均为空）：
1. 直接点击"🗑 清空"按钮
2. 观察是否出现信息提示"已经是空白状态"（不是确认对话框）
仅在当前页面操作，不要导航到其他网址。
```

**Expected Results（逐条）：**
1. 标题和内容均为空时点击"清空"按钮，显示信息提示"已经是空白状态"（AC-05-4）

---

### L3.5 编辑区 placeholder 显示

| 项 | 值 |
|---|---|
| mosstid | `vla-ntp-v1-L3.5-01` |
| 关联 AC | AC-02-4 |
| 设计技术 | PRD 追溯 |
| 数据依赖 | EMPTY |
| 冲突标记 | 无 |
| 前置策略 | 策略一 |
| fixture | 无 |

**Pre-flight：** 执行脚本 5a（确保空白状态）

**任务描述：**
```
在当前页面上（标题和内容均为空）：
1. 观察标题输入框，确认显示 placeholder "输入笔记标题…"
2. 观察编辑区，确认显示 placeholder "开始记录…"
仅在当前页面操作，不要导航到其他网址。
```

**Expected Results（逐条）：**
1. 编辑区为空且用户未输入任何内容时，显示 placeholder 文字"开始记录…"（AC-02-4）

---

### L3.6 标题长度边界（100 字符）[BVA]

| 项 | 值 |
|---|---|
| mosstid | `vla-ntp-v1-L3.6-01` |
| 关联 AC | BR-01（标题长度限制 100 字符） |
| 设计技术 | BVA — 标题字段上界（D1） |
| 目标边界 | maxlength = 100（HTML 属性强制截断） |
| 数据依赖 | EMPTY |
| 冲突标记 | 无 |
| 前置策略 | 策略二（注入 100 字符标题）+ 策略一（观察截断行为） |
| fixture | `title-boundary-100.txt` |

**Pre-flight：** 执行脚本 5a，然后注入 100 字符标题：
```bash
osascript -e '
tell application "Google Chrome"
    tell active tab of front window
        execute javascript "
            document.getElementById(\"titleInput\").value = \"测试标题边界值一二三四五六七八九十一二三四五六七八九十一二三四五六七八九十一二三四五六七八九十一二三四五六七八九十一二三四五六七八九十一二三四五六七八九十一二三四五六七八九十一二三四五六七八九十测试完\";
            document.getElementById(\"titleInput\").dispatchEvent(new Event(\"input\"));
        "
    end tell
end tell'
```

**任务描述：**
```
在当前页面上（标题输入框中已有一段很长的标题文字）：
1. 观察标题输入框中的文字长度
2. 尝试在标题末尾继续输入"溢出"两个字
3. 观察是否能继续输入（标题应被限制在 100 字符以内，输入框应阻止超过 100 字符的输入）
仅在当前页面操作，不要导航到其他网址。
```

**Expected Results（逐条）：**
1. 标题输入框已显示 100 字符的标题文字（BR-01）
2. 尝试继续输入时，输入被截断/阻止，标题长度不超过 100 个字符（BR-01，HTML maxlength="100" 属性生效）

---

## 7. 冲突扫描结果

### 写操作用例清单

| 用例 | 写操作 | 影响范围 |
|------|--------|---------|
| L1.3 | 保存标题+内容到 localStorage | `vla-ntp-title`, `vla-ntp-content` |
| L1.8 | 清空标题+内容 + 删除 localStorage | `vla-ntp-title`, `vla-ntp-content` |
| L2.1 | 切换暗色模式 | `vla-ntp-theme` |
| L2.3 | 切换字体 | `vla-ntp-font` |

### 冲突分析

由于每条用例执行前均执行 Pre-flight 脚本 5a（清除 localStorage + 刷新），数据状态在每条用例启动时被重置为 EMPTY。所有 CUSTOM 依赖通过注入脚本（5b/5c/5d 或 DOM 注入）独立构造。

**结论：** 按 L1 → L2 → L3 顺序执行无数据冲突。条款 1c 的清理由 5a 统一保障。

---

## 8. 执行注意事项

1. **浏览器原生 `confirm()` 对话框：** L1.8 和 L2.7 涉及 `confirm()` 弹窗，mano-cua 需能处理原生对话框。若无法处理，标记 BLOCKED 并按条款 6a/7a 附诊断。

2. **文件下载验证：** L1.5/L1.6/L1.7 的导出文件由浏览器下载，mano-cua 可观测到下载触发（如下载条出现）。文件内容的详细验证由 MOSS 在 Post-flight 阶段通过检查下载文件夹完成。

3. **Tooltip hover：** L3.1 需要 mano-cua 执行 hover 操作。mano-cua 的 VLA 模型应支持鼠标移动+悬停。

4. **GitHub Pages 网络依赖：** 被测页面部署在 GitHub Pages，测试前确认页面可正常访问。

5. **mano-cua 日志路径规范：**
   ```
   ~/.openclaw/workspace/reports/mano-cua/logs/2026-04-21/vla-ntp-v1-<测试点ID>.log
   ```

---

## 9. 文档合规 Checklist

- [x] 文档信息（项目、版本、编写人、日期、状态）
- [x] 测试范围（L1/L2/L3 完整覆盖）
- [x] 测试用例（22 条，每条含 mosstid / AC / 策略 / Pre-flight / 任务描述 / Expected Results）
- [x] Fixture 清单（4 个文件，内容详列）
- [x] 测试环境（浏览器、OS、工具、URL）
- [x] BVA 增补（条款 D1 — 标题 100 字符边界）
- [x] 数据隔离声明（条款 D2 — 每条用例标注数据依赖）
- [x] 冲突扫描（条款 D2.4）
- [x] Expected Results 逐条列出（条款 9）
