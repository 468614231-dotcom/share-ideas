# 交互说明（interactions.md）

## 目录

1. 交互清单
2. 主题切换与定制
3. 常见定制（速度 / 开关效果 / 新增页面类型）
4. 演讲稿备注用法
5. 常见问题

---

## 1. 交互清单

模板 `<script>` 为 IIFE，一次加载、零依赖、离线可用。

| 交互 | 触发方式 | 实现位置 | 说明 |
|---|---|---|---|
| 上一页/下一页 | `← ↑` / `→ ↓` / 空格 / PageUp / PageDown | keydown | 空格 preventDefault，避免误滚 |
| 跳首页/末页 | Home / End | keydown | — |
| 数字跳页 | 1–9 | keydown | 跳到第 N 页（1 = 封面） |
| **切换主题** | T | keydown → `setTheme()` | tech → consult → finance 循环，右上角显示主题名 |
| 滚轮翻页 | 任意方向滚轮 | wheel | 900ms 防抖、|deltaY|<12 忽略 |
| 触屏滑动 | 横向滑动 > 60px | touchstart/touchend | 移动端可用 |
| 点击翻页 | 左 22% 区回退、右 78% 区前进 | click | 自动避开 a/button/圆点 |
| 全屏 | F | keydown + Fullscreen API | 再次按 F 退出 |
| 演讲备注 | N | keydown | 显示当前页 `data-note`，Esc 关闭 |
| 圆点导航 | 点击右侧圆点 | 动态生成 | 悬停显示页名 |
| 底部翻页按钮 | ◀ ▶ | click | 鼠标用户主入口 |
| **数字滚动** | 进入含 `.num[data-count]` 的页 | sync → countUp() | 900ms easeOut |
| **面积图描线** | 进入含 `.chart-area` 的页 | CSS `.active` | 描线生长 + 渐变淡入 |
| **环形进度** | 进入含 `.ring` 的页 | CSS `.active` | 圆环生长（`--pct`） |
| **卡片倾斜跟随** | 鼠标悬停 card/panel | mousemove | 仅鼠标设备（hover:hover），触屏自动关闭 |

## 2. 皮肤切换与定制

**选定固定皮肤**：改 `<body class="theme-xxx">` 为 tech / consult / finance 之一。保留 T 键则演示时仍可切换，不想要可删 keydown 中 `case 't'` 分支。

**改皮肤 = 改 token 块**（`:root` 为 tech 默认，`body.theme-consult/finance` 为覆盖）：
- 强调色：只改 `--primary / --accent / --highlight` 三个变量；不新增第 4 个强调色变量
- **浅色底**：`--bg/--bg-2` 用白/浅灰，`--text/--dim` 用深色系；禁止黑色底
- 皮肤语言：`--radius-*` 圆角、`--shadow-*` 阴影、`--font-display` 标题字体（可切衬线）、`--card-glass/--card-blur` 毛玻璃、`--bg-pat` 底纹
- `--danger` 只用于风险/负值语义；`--line/--line-dim` 随主色同步调整

**新增皮肤**：复制一个 `body.theme-xxx{}` 块，命名如 `body.theme-brand`，再往 JS `themes` 数组与 `themeNames` 映射中加一项即可（T 键自动纳入循环）。

## 3. 常见定制

**改翻页速度 / 防抖：**
```js
// wheel 防抖毫秒数（默认 900）
if (now - lastWheel < 900 || Math.abs(e.deltaY) < 12) return;
// 进场时长与错峰在 CSS：
.slide .reveal{transition:opacity .55s ease, transform .55s ...}
.slide.active .reveal:nth-child(n){transition-delay:.04s + n*0.07s}
```

**关闭某效果：**
- 打字机：删元素上的 `.tw` 类
- 进场错峰：删 `.reveal` 类（`prefers-reduced-motion` 下已自动关闭）
- 主题切换：删 keydown 的 `case 't'` 分支
- 卡片倾斜：删 JS `matchMedia('(hover:hover)')` 块；触屏设备本就自动关闭

**数字滚动参数：**
```html
<span class="num" data-count="26.8" data-prefix="¥" data-suffix="" data-dec="1">0</span>
<!-- data-count 目标值 · data-prefix 前缀 · data-suffix 后缀 · data-dec 小数位（缺省按目标值自动判断） -->
```

**封面版式三选一：**
```html
<section class="slide cover" data-year="2026">…居中大标题…</section>
<section class="slide cover left" data-year="2026">…左对齐（回归旧版）…</section>
<section class="slide cover split" data-year="2026">…左标题 + 右结论卡（.cover-note）…</section>
```

**新增页面类型（团队页、预算表等）：**
1. 复制任一 `.slide` 结构，改 `data-title` / `data-note` 与内容
2. 用现有组件拼装（panel/card/stat/list…），新样式写在模板 `<style>` 内并复用主题变量
3. 直接子元素加 `.reveal` 获得错峰动画
4. 页码/进度条/圆点自动适配，无需改 JS

## 4. 演讲稿备注用法

每页 `<section>` 上写：
```html
<section class="slide" data-title="风险与应对" data-note="先讲结论：三项风险均已收敛，重点看合规项。">
```
- 演示时按 `N` 呼出/隐藏备注面板（米白/琥珀高亮边，观众不可见），`Esc` 关闭
- 无备注页显示"（本页无备注）"

## 5. 常见问题

- **页面超长内容被截断**：`.slide` 有 `overflow-y:auto`，但汇报页不应滚动——删条目/缩文案，别依赖滚动
- **点击翻页与按钮冲突**：click 已排除 `a/button/.dot`；新加可点元素记得纳入排除
- **全屏后 Esc 行为**：浏览器先退出全屏，再按一次处理备注
- **改字色后看不清**：回 design.md 令牌表核对对比度，正文只用 `--text/--dim`
- **复制模板后 JS 失效**：确认 `<section class="slide">` 未删、`id`（deck/progress/counter/theme-tag 等）未改名
- **数字键与输入框冲突**：keydown 已跳过 input/textarea
- **多个面积图渐变重名**：模板只有一个面积图（`#gArea`）；复制第二个时改 id（如 `gArea2`）与 `fill="url(#gArea2)"`
- **毛玻璃无效果**：浅色底需配合 `body::before` 背景斑块才可感知；若删掉了斑块，玻璃看起来像纯色卡片，属正常
- **衬线皮肤标题发虚**：consult 的 `--font-display` 是衬线栈，Windows 下 SimSun 加粗偏糊属系统字体限制，可换 `"Noto Serif SC"` 或回退无衬线
