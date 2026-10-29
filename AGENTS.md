# AGENTS.md — AI 协作约定

本文件是 AI 修改本项目时的唯一权威约定。人类可读的使用说明在 README.md。

> 🔴 需求、架构、内容或流程发生任何变更后，必须同时更新 README.md（人看）
> 和本文件 AGENTS.md（AI 看）。只改一个视为未完成。

## 项目性质

大学英语课小组作业，课堂展示用。三件套，全部单文件、零依赖、零构建、离线可用：

- `presentation.html` — 仿 PPT 展示页（15 页，投屏用）
- `interaction.html` — 互动页（学生扫码，全交互合并于此）
- `speech.txt` — 演讲稿（纯文本，禁止 Markdown 格式）

## 硬性约束（违反即错误）

1. **单文件**：每个 HTML 自包含全部 CSS/JS，不得引入外部 JS/CSS/字体/CDN/图片。
2. **无后端**：不允许任何服务器逻辑、存储、第三方 API、网络上传。数据只存浏览器
   localStorage（键前缀 `wxf.`，仅展示页使用）或内存。
3. **互动路由**：互动页仅通过 `?activity=` 区分活动，已注册：
   - `survey` — 12 题自查问卷，题目写死在前端
   - `race` — 抢答，参数 `q`（题号 1–3）、`start`（开始时间，毫秒时间戳或 HH:MM:SS）、
     `preview`（=1 立即解锁，测试用）、`answer`（已选答案字母 A–D，作答后写入用于防刷新重答）、
     `t`（作答实际用时，毫秒整数，与 `answer` 同时写入，刷新时原样显示）
   抢答每题只允许作答一次：点选后将答案字母和用时用 `history.replaceState` 写进 `answer` / `t`
   参数（不新增历史记录），刷新时若 `answer` 合法则直接渲染结果页并沿用 `t`（缺失或非法时
   才按 `now()-start` 回算），不得再次出现选项；
   抢答只能从二维码进入：无参数落地页不得提供抢答入口（只保留自查问卷与扫码提示），
   `race` 缺少合法 `start` 且无 `preview=1` 时须渲染扫码提示页，不得出题；
   新互动必须合并进 interaction.html，不新建文件。
4. **抢答开始时间**由展示页第 13 页抢答二维码下方的控件生成进二维码 URI
   （`start` 毫秒时间戳），互动页解析后倒计时解锁；时钟偏差用 HTTP `Date`
   头校准，失败回退设备时钟。控件为 `rqAhead`（秒数）+ `rqApply`
   （"Apply · 应用"）：点击即把"当前时间 + N 秒"格式化成 HH:MM:SS 写入
   `cfg.start` 并立即保存、刷新二维码，无需再去设置面板。设置面板（S）
   只保留 `inpBase`（互动页网址）与 `inpPreview`（预览模式）；`rqQ` 题号
   下拉框同样位于第 13 页二维码下方，变更即时保存并刷新二维码。
5. **二维码**：内置自写编码器（字节模式、EC 级 M、版本 1–10、掩码自动选优），
   禁止改用外部生成服务。改动编码器必须重新验证（见"验证"）。
6. **无图片资源**：视觉全部用 CSS/SVG，不使用 `<img>` 或外部图片 URL。
7. **兼容性基线（按页面分级，不要再一刀切）**：
   - `interaction.html` — 兼容要求最高。学生用自己的手机扫码，设备与浏览器不可控：
     不用 `color-mix()`；`dvh` 必须在其前一行写 `vh` 回退（双写）；
     保证 2021 年后的手机浏览器可用；支持 `prefers-reduced-motion`。
   - `presentation.html` — 投屏设备可控，可用全部 Baseline 特性
     （`color-mix()`、`dvh`、`svh/lvh` 等均可直接用，无需再写回退）；
     仍须支持 `prefers-reduced-motion`。
   - 现状：两页均未使用 `color-mix()`；展示页仅用 `vh`，互动页已做 `vh`+`dvh` 双写。
     放宽只是解除限制，不要求主动改写现有样式。
8. **讲稿是纯文本**：speech.txt 不得出现 Markdown 语法（#、**、表格等）。
9. **离线优先**：展示页在 `file://` 下也必须能完整翻页；二维码在托管后才可用。

## 结构与一致性

- 幻灯片是 `presentation.html` 内的 `<section class="slide" data-title="...">`，
  共 15 页；每页含一个隐藏 `.note`（演讲者备注，格式固定：
  `【第N页｜主讲 X｜约NN秒】`，页内用 `\n` 换行）。
- `speech.txt` 按"第 N 页 · 主讲 X"逐页对应幻灯片，页码与主讲人必须与
  `.note` 一致。改任一侧必须同步另一侧（15 页 / 15 段，含 A–F 六角色）。
- 双语要求：互动页所有正文（题干、选项、说明、按钮、提示、页脚）必须中英双语，
  英文在前、中文在后（中文用 `.zh` 小号 muted 样式）；展示页第 9、10、13 页
  的互动内容同样适用。新增互动文案时不得只写一种语言。
  **例外**：抢答（race）作答页的题干与四个选项只显示英文（`question.en` / `question.options`），
  不加 `.zh`；其余（计时提示、结果页、解释、按钮）仍须中英双语。
- 幻灯片页序与讲稿页序、互动页题号三者联动，改题数或插页需三处核对。

## 关键数据位置（改内容先找这里）

| 内容 | 位置 |
|---|---|
| 问卷 12 题（英/中/维度） | interaction.html `SURVEY` 数组 |
| 三维定义与建议 | interaction.html `DIMS` |
| 分数分档（≤30 / ≤42 / ≥43） | interaction.html `BANDS` |
| 抢答 3 题与答案解释 | interaction.html `RACE` |
| 幻灯片正文 | presentation.html 各 `<section>` |
| 演讲者备注 | 各 `<div class="note">` |
| 二维码编码器 | presentation.html 顶部 `QRCodeGen` IIFE |
| 设置持久化 | localStorage `wxf.base / wxf.start / wxf.q / wxf.preview / wxf.notes / wxf.timer*` |
| 讲稿正文 / Q&A / 应急预案 | speech.txt 对应章节 |

## 验证

环境：Node ≥ 18；jsdom 已作为 devDependency 装在仓库（`pnpm install` 即可），
`package.json` 为 `"type": "module"`，测试脚本用 `.mjs`。测试脚本本身仍为临时文件，不入库。
Python venv（qrcode、opencv-python-headless）可用时做二维码验证。

1. 语法：提取两 HTML 的 `<script>` 用 `new Function()` 过一遍。
2. 行为：jsdom 加载页面（`runScripts: "dangerously"`），
   - 交互页：问卷 12 题出分、抢答 未来 start 倒计时→解锁→作答→结果、
     过去/preview 立即出题、未知 activity 提示、菜单链接带正确参数；
     无参数落地页无抢答入口且提示扫码、`race` 缺 start 且无 preview 时出扫码提示页；
     答题后 `answer` / `t` 参数写入且 `history.length` 不增长、带 `answer`+`t` 重载直接出结果页
     且用时与 `t` 一致、`answer` / `t` 非法时忽略；
   - 展示页：翻页（键盘/点击/边缘回退）、备注换行、计时器开关、
     设置保存后二维码 SVG 生成且 URL 文案正确（q/start/preview 参数）、localStorage 落盘。
   - 断言范围限定 `#app` 或 `.slide` 内容——`body.textContent` 会包含内嵌脚本源码导致误判。
3. 二维码（改编码器后必做）：生成矩阵与 Python `qrcode` 库（EC-M、同掩码）
   逐行比对完全一致，并用 OpenCV `QRCodeDetector` 解码回原文。
   已知坑：版本 9 分组表数据码字为 [36,37]；对齐图形必须跳过与定位符
   重叠的中心（`(r<=8&&c<=8) || (r<=8&&c>=size-9) || (r>=size-9&&c<=8)`）。

## 变更流程

1. 先读本文件与目标文件相应章节。
2. 改代码/内容，跑上述验证。
3. 若变更涉及需求、架构、流程、页数、角色分工、互动规则或使用方式：
   **同步更新 README.md 和 AGENTS.md**（本节即该规则的出处）。
4. 纯排版/错别字修正不必动这两个文件，但页数、题数、快捷键等可数事实变了必须动。
