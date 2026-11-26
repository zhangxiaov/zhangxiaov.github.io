# AGENTS.md — 汉字词条多语言页面生成指令

当用户提供一个 `*-r.txt` 词条文件（路径或直接贴出内容）时，**直接按本文件执行，不要反复确认翻译或格式**，只报告最终结果。

## 1. 输入约定

源文件形如 `D:\code\pydemo\dict\content\r\yu-r.txt`：

```
鱼

r.png

“鱼”是一个古老的象形字……（正文段落）
“鱼”的古音今天已经失传……
例句：“我爱食鱼”，“我爱钓鱼”，“鱼缸里有5条金鱼”

“鱼”组合的字有100+，常用的比如“鲜穌魯爯”。

“鲜”=“鱼”+“羊”，……（衍生字解读）
例句：“此羊肉汤很鲜”。
“魯”=“鱼”+“口”，……
例句：“办事别太鲁”。

朝鲜是一个音译词……（末尾补充）
```

- 第 1 行是字头；第 3 行是原图文件名，多语言页**不用它**。
- `例句：“……”` 行：提取引号内句子，**引号和中文原句不翻译**。
- `“X”=“鱼”+“Y”` 行：转成衍生字区块，等号左边单独一行作为 `.combo`。
- 正文里所有 `“ ”` 内的字（如“冉”“鲜”“羊”“口”）一律保留原文。

## 2. 输出产物

- 输出目录：`C:\Users\servi\Documents\GitHub\zhangxiaov.github.io\r\<语言代码>\<源文件stem>.html`
  - 例：输入 `yu-r.txt` → 输出 `r/en/yu-r.html`、`r/cn/yu-r.html` …
  - 简体中文目录固定为 `cn`（`lang="zh-Hans"`），繁体为 `zh-hant`。
- 共享样式：`r/assets/style.css`，各页用 `../assets/style.css` 引用；**不要内联样式**。
- 37 个语言代码（顺序固定）：

```
en es de fr pt it ru ja ko vi th id ar hi tr ms fil bn fa ur nl pl uk sv da no fi cs hu el km lo my ta zh-hant mn cn
```

## 3. 字形演变图（必须替换手绘 SVG）

1. 数据库候选路径（按序尝试）：
   - `D:/code/ChineseCharacters/sw_single_table.db`
   - `D:/code/hz/sw_single_table.db`
   - `D:/code/imecode/sw_single_table.db`
2. 查询：`SELECT img FROM sw WHERE zi='<繁体>' OR zi2='<简体>' LIMIT 1`（如 `zi='魚' OR zi2='鱼'`）。
3. `img` 字段以 `;` 分隔，逐项检查 `ziimgs/` 下是否存在；只保留存在的文件。
4. 页面中生成图片网格，路径写为 `../../ziimgs/<子目录>/<文件>.png`；按数据库字段顺序排列。
5. 图区结构：

```html
<figure class="glyph">
  <p class="glyph-title">本地化的“字形演变”</p>
  <div class="glyph-grid">
    <img src="../../ziimgs/..." alt="字 — glyph form 1" loading="lazy" />
    ...
  </div>
  <figcaption>本地化的“从古代象形图画到后世书写风格”说明</figcaption>
</figure>
```

6. 各语言的标题与说明文案参考 `r/assets` 同级的生成规则；简体中文用：
   - 标题：`字形演变`
   - 说明：`“鱼”的历史字形 —— 从古代的象形图画到后世的书写风格。`（换字时替换字头）

## 4. 页面结构（每页一致）

```
<head>
  title / description / keywords（本地化）
  canonical: https://zhangxiaov.github.io/r/<code>/<stem>.html
  37 条 hreflang + x-default（指向 en）
  OG: type/site_name/title/description/url/locale
  JSON-LD DefinedTerm：name/description/termCode，subjectOf.inLanguage=<lang>
  <link rel="stylesheet" href="../assets/style.css">
</head>
<body id="top">
  header: brand + <details class="lang-picker"> 37 个语言链接
  article.card:
    eyebrow / h1 / subtitle
    figure.glyph（见上）
    section#origin：字源与异体
    section#pronunciation：古音与方言
    section#derived：衍生字（.derived + .combo）
    section#examples：5 个 .example（按源文件的例句数量）
    p.last-note：末尾补充
    aside.note：作者个人解读声明
    nav.related：相关字，链接统一 href="#"
  footer: © 2026 解字机 AK + 返回顶部
```

**例句三段式**（每个 `.example`）：

```html
<span class="label">中文</span>
<p class="ex-zh" lang="zh-Hans">“原句（引号保留）”</p>
<span class="label">Pinyin</span>
<p class="ex-roman">带声调拼音</p>
<span class="label">目标语言名</span>
<p class="ex-trans">目标语译文</p>
```

- 拼音用 `pypinyin` 的 `Style.TONE` 生成；简体中文页第三段标签用 `现代汉语`，繁体页用 `繁體中文`。

## 5. 关键坑（务必遵守）

- H1 里的强调标签：`h1 = escape(原文)` **之后**再 `.replace('鱼', '<span class="han">鱼</span>', 1)`；不要在插入标签后再 `escape`，否则页面会显示 `&lt;span`。
- `figure.glyph` 里不要再写 `<svg>`；图片数量必须与数据库 `img` 字段有效项一致。
- 所有字卡、相关字链接继续用 `href="#"`（详情页尚未建设）。
- 阿拉伯语 / 波斯语 / 乌尔都语：`dir="rtl"`；其余 `ltr`。
- 引号内容（包括正文里的字）不要翻译；只翻译叙述文字。
- 繁体页叙述用繁体，但引号内保留源文件的简体字。

## 6. 收尾校验（必须通过再汇报）

- 语言页面数量 = 37；每页 HTML 文件名与输入 stem 一致。
- 每页例句数量 = 源文件 `例句` 行数量；中文原句与引号完整保留。
- 每页 `<img src="../../ziimgs/...">` 数量 = 数据库有效图片数，且文件真实存在。
- 无残留 `<svg>`、无 `&lt;span`、无未替换模板变量。
- JSON-LD 可被 `json.loads` 解析；hreflang 数量 = 38（37 + x-default）。
- 完成后在 `.workbuddy/memory/当天日期.md` 追加一条简短记录。

## 7. 沟通原则

- 用户只给 txt 内容/路径时，直接执行上述流程，不追问格式、翻译或样式。
- 只有数据库记录和 `ziimgs/` 都找不到对应字，或源文件缺少字头时，才提问。
- 汇报仅需：生成页面数、文件路径、图片数、例句数、任何异常。
