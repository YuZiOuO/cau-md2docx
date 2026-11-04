---
name: cau-md2docx
description: 把用户提供的报告初稿、数据与图片整理为论文格式 Markdown，并转换为 CAU 课程论文 Word 文档（表题/图题自动编号、Mermaid 转 PNG、封面与目录页码）。当用户要求把报告或初稿整理成论文格式 Word、提到“论文格式自动整理”网页、md2doc-cli 或 CAU 课程论文模板时使用。
---

# 报告材料 → CAU 课程论文 Word

将人工撰写的报告初稿和原始图片材料整理为 Word 文档。以原稿内容、数据和结论为准，不要擅自扩写实验结果或改变观点。

## 底线

- 内容、数据、结论以原稿为准：不扩写实验结果、不新增数据、不改动观点、不臆造章节。
- 图片缺失、引用对不上、数字或单位存疑时，明确列出并询问，不要自行补造。
- 交给转换工具的 Markdown 文件本身就是论文正文，不要在里面写解释性文字、前言或操作说明。

## Markdown 约定

1. 输出一个完整的 Markdown 文档，不要输出解释性文字。
2. 文档将与 images/ 目录一起打包为 zip；所有图片必须使用相对路径，例如：![流程图](images/flow.png)。
3. 一级 Markdown 标题 # 用作论文标题；##、###、#### 分别作为正文中的一级、二级、三级标题。
4. 不要手写标题编号，转换工具会自动编号。
5. 摘要可以写成“摘要：摘要正文”，也可以把“摘要”单独写一行后下一段写摘要正文；不要写成 Markdown 标题。
6. 关键词单独写一段，格式为“关键词：词一；词二；词三”。
7. 表格必须使用 GFM Markdown 表格语法。
8. 每个表格的表题必须单独放在表格正上方一行，格式为“表 1 表题文字”或“表1 表题文字”；表题行和表格之间不要插入其他内容。
9. 如果工具开启表题自动编号，表题行也可以写成“表 表题文字”，工具会自动整理为“表1-1”或“表1”。
10. 图片必须单独成段，不要和正文放在同一行。
11. 每张图片的图题必须单独放在图片正下方一行，格式为“图 图题文字”或“图1 图题文字”；图题行和图片之间不要插入其他内容。
12. 如果工具开启图题自动编号，图题会自动整理为“图1-1”或“图1”，不要在图题正文里重复编号。
13. 表格内容只写数据，不要在表格内部写表题；单元格内的竖线必须写作 `\|`，例如 `RDR1\|PING\|7\n`。
14. 正文段落保持自然中文论文表达，避免使用 HTML。
15. 普通代码使用 fenced code block；需要流程图、状态图、类图或时序图时，使用语言标记为 `mermaid` 的 fenced code block，并在其正下方写图题。不要把 Mermaid 预先截图为图片。
16. Mermaid 图表会按所选模板的图型开关转换为 PNG，并采用模板设置的论文黑白风格；未启用或渲染失败时会保留为代码块。

## 准备工具（首次需联网）

转换工具是单文件 CLI，与含目录页码的模板 JSON 一起，首次使用时下载到本 skill 所在目录的 `bin/` 下，之后复用。两者都是项目 CI 发布的固定地址，与网页版同源：

```bash
mkdir -p bin
curl -fsSL -o bin/md2doc-cli.mjs \
  https://huluntuntao.github.io/cau-md2docx/downloads/md2doc-cli.mjs
curl -fsSL -o bin/cau-course-paper-toc.json \
  https://huluntuntao.github.io/cau-md2docx/downloads/cau-course-paper-toc.json
```

- 已下载则跳过。用户要求更新版本时重新执行下载。
- CLI 已内嵌全部依赖，不需要 npm install。官方验证 Node v24.11.1 / Bun v1.3.12，Node v22 实测可用；只有 Bun 时把 `node` 换成 `bun`。
- 上述 `bin/` 相对路径相对本 SKILL.md 所在目录；在 Claude Code 中即 `${CLAUDE_SKILL_DIR}/bin`，在 Codex 中为该 skill 安装目录下的 `bin/`。

## 打包与转换

1. 将 Markdown 文档和 images/ 目录压缩为一个 zip 文件，zip 中只能有一个 .md 或 .markdown 文档。打包前确认：工作目录中只有一个 Markdown 文档；用 `grep -oE '!\[[^]]*\]\([^)]+\)'` 列出的图片引用逐一存在且都在 `images/` 下，缺失或不符的明确列出。在 Markdown 所在目录执行，保证文档位于 zip 根目录，排除草稿、隐藏文件和 `__MACOSX`：
   ```bash
   zip -r report.zip report.md images
   ```
2. 默认使用含目录页码模板并添加默认封面，优先使用 Node.js 执行；如果环境只有 Bun，也可以使用 Bun：
   ```bash
   node bin/md2doc-cli.mjs report.zip -o report.docx --template bin/cau-course-paper-toc.json --cover
   bun bin/md2doc-cli.mjs report.zip -o report.docx --template bin/cau-course-paper-toc.json --cover
   ```
   如果用户提供自定义封面，将 `--cover` 改为 `--cover-file cover.docx`。
3. 如果用户明确不需要封面，省略 `--cover`；如果用户明确不需要目录和页码，省略 `--template`，此时使用内置“CAU 课程论文”模板（不含目录和页码）。
4. 全部参数只有：`-o/--output`、`--template <file>`、`--cover`、`--cover-file <cover.docx>`（隐含 `--cover`）、`-v/--version`、`-h/--help`。其余行为（Mermaid 图型、黑白风格、标题编号、目录、页码）都是模板 JSON 字段，没有命令行开关。退出码为 0 且打印「已生成：…docx」「已读取文档：…；图片资源：N 个」才算成功，核对 N 与实际图片数是否一致；stderr 里的 `Warning: KaTeX doesn't work in quirks mode` 是无害噪音。

## 交付

- 完成后提供生成的 .docx 文件，并说明 Markdown 文件、图片目录、zip 文件和 Word 文件的位置。
- 说明 Mermaid 图表已转换为 PNG，或列出哪些因模板未启用/渲染失败而保留为代码块。
- 如果图片缺失或材料存在无法确认的内容，明确列出，不要自行补造。
- 使用含目录页码模板生成 Word 后，必须提醒用户用 Word/WPS 打开并更新目录/域，目录条目和页码才可靠；同时检查封面信息、图片大小与分页。
