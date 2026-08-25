下面是针对仓库 fangxu6/Podcast-Subtitle (https://github.com/fangxu6/Podcast-Subtitle/tree/main)，重点生成 docs\episodes 的教程，同时说明历史上的 `pie-markdown`、`pie-html` 和当前静态站点的生成方式。

## 1. 生成关系

pie-podcast-nav-v2.md
        +
pie-srt\v2\*.srt
        |
        v
scripts\build_vitepress_site.py
        |
        +-- docs\episodes\*.md
        +-- docs\public\pie-srt\v2\*.srt
        +-- docs\public\wordclouds\*.svg
        +-- docs\index.md

docs\episodes 中的 Markdown 页面不是手写的，而是根据导航和 SRT 自动生成。

历史归档格式则是另一条旧流程：

```text
pie-srt\*.srt
        |
        +-- 人工/一次性整理成段落
        |       +-- pie-markdown\*.md
        |       +-- pie-html\*.html
        |
        +-- 当前可复现的静态站点脚本
                +-- transcript-site\
```

## 2. 环境准备

在仓库根目录执行：

cd D:\Workspace\obsidian\03-Resources\Podcast-Subtitle

python --version
node --version
npm --version

Python 建议 3.10 以上。生成 docs\episodes 不需要额外 Python 依赖。

如果要本地预览或构建 VitePress：

npm ci

## 3. 准备新一期内容

例如新增第 228 期：

### 放入字幕文件

pie-srt\v2\pie-ep228.mp3.srt

### 更新导航文件

在 pie-podcast-nav-v2.md 顶部加入：

[第228期 节目标题](https://cdn2.wavpub.com/path/pie-ep228.mp3)
[字幕](./pie-srt/v2/pie-ep228.mp3.srt)

音频链接和字幕链接需要成对出现，中间不要插入其他内容。

## 4. 生成 docs\episodes

### 只生成指定一期

推荐新增单集时使用：

python scripts\build_vitepress_site.py --episode ep-228

生成结果：

docs\episodes\ep-228.md
docs\public\pie-srt\v2\pie-ep228.mp3.srt
docs\public\wordclouds\ep-228.svg

首页 docs\index.md 也会自动增加该期的目录卡片。

也可以按 SRT 文件生成：

python scripts\build_vitepress_site.py --srt pie-ep228.mp3.srt

### 增量补齐缺失内容

python scripts\build_vitepress_site.py --append-new

该模式会扫描导航文件，处理缺少页面、SRT、词云或首页卡片的条目。它的实际实现会扫描整个导航，不仅仅是最新一期。

### 补齐全部缺失资源

python scripts\build_vitepress_site.py --fill-missing

它只补缺失文件，不会重建已经存在的单集页面。

### 全量重建

python scripts\build_vitepress_site.py

注意：全量模式会删除并重建：

docs\episodes
docs\public\pie-srt\v2
docs\public\wordclouds

同时覆盖：

docs\index.md

因此不要直接修改这些自动生成目录中的文件。

## 5. 验证生成结果

以第 228 期为例：

Test-Path docs\episodes\ep-228.md
Test-Path docs\public\pie-srt\v2\pie-ep228.mp3.srt
Test-Path docs\public\wordclouds\ep-228.svg

Select-String docs\index.md -Pattern '/episodes/ep-228'

全部存在并且首页能搜索到 ep-228，说明生成完成。

## 6. 本地预览

生成后启动 VitePress：

npm run docs:dev

访问：

http://localhost:5173/

正式构建：

npm run docs:build

最终静态文件位于：

docs\.vitepress\dist

GitHub Actions 会在推送到 main 后自动执行同样的构建流程。

## 7. 当前仓库需要注意的状态

当前 pie-podcast-nav-v2.md 中存在：

- 多个历史标题换行问题；
- 3 个导航引用但实际不存在的字幕文件：pie-ep156.mp3.srt、pie-ep155.mp3.srt、pie-ep121.mp3.srt；
- docs\episodes 目前只生成了一部分历史页面。

可以先检查：

python scripts\update_nav_v2.py --check

修复标题换行：

python scripts\update_nav_v2.py --fix-wrap
python scripts\update_nav_v2.py --check

缺失的 SRT 需要补回文件，或者修正导航中的字幕链接。

## 8. `pie-markdown` 和 `pie-html` 如何生成

### 8.1 它们是历史归档，不是当前生成流程

`pie-markdown` 和 `pie-html` 中的文件与旧版 `pie-srt` 一一对应。例如：

```text
pie-srt\ep62.mp3.srt
pie-markdown\ep62.mp3.md
pie-html\ep62.mp3.html
```

文件名只改变扩展名，特殊文件也保持原来的 stem，例如：

```text
pie-srt\epx01.mp3.srt       -> pie-markdown\epx01.mp3.md
                              -> pie-html\epx01.mp3.html
pie-srt\books-22-05-1.mp3.srt -> pie-markdown\books-22-05-1.mp3.md
                                  -> pie-html\books-22-05-1.mp3.html
```

仓库当前没有 `SRT -> pie-markdown` 或 `SRT -> pie-html` 的转换脚本，也没有对应的 npm 命令。这些文件是在 2023 年的历史提交中直接加入的归档结果，不能从当前仓库找到一个可重复执行的生成命令。

### 8.2 文件格式

Markdown 文件类似：

```markdown
# ep62.mp3

- 第一段整理后的字幕文字……
- 第二段整理后的字幕文字……
```

HTML 文件保存的是同一批段落，只是包在 HTML 标签中：

```html
<h1>ep62.mp3</h1>

<ul>
<li>第一段整理后的字幕文字……</li>
<li>第二段整理后的字幕文字……</li>
</ul>
```

它们不是把每条 SRT cue 原样复制成一行，而是先把多条字幕合并成较长的阅读段落。因此，如果要继续维护这种旧格式，通常流程是：

1. 以 `pie-srt\*.srt` 为原始来源；
2. 合并字幕 cue，人工修正断句和段落；
3. 保存为 `pie-markdown\同名.mp3.md`；
4. 把每个 Markdown 段落放入 `<li>`，保存为 `pie-html\同名.mp3.html`。

当前仓库没有自动完成第 2 步的旧版工具，所以不能只运行一条现有命令生成与历史文件完全一致的结果。

### 8.3 当前可复现的 HTML 生成方式

如果目标只是得到可访问、可搜索、带音频时间轴的 HTML 站点，使用当前脚本：

```powershell
python scripts\build_transcript_site.py
```

默认输入和输出是：

```text
输入：pie-podcast-nav-v2.md
输入：pie-srt\v2\*.srt
输出：transcript-site\index.html
输出：transcript-site\episodes\*.html
输出：transcript-site\pie-srt\v2\*.srt
```

指定输出目录：

```powershell
python scripts\build_transcript_site.py --output .\transcript-site
```

只生成导航文件前面的若干集：

```powershell
python scripts\build_transcript_site.py --limit 8
```

这个脚本会自动解析 SRT、合并字幕段落、生成单集 HTML、保留原始音频链接，并生成时间导航。它不会写入 `pie-html` 或 `pie-markdown`，也不会写入 `docs\episodes`。

### 8.4 三种输出应该如何选择

```text
需要 VitePress 文档页       -> python scripts\build_vitepress_site.py
需要独立 HTML 阅读站        -> python scripts\build_transcript_site.py
需要历史 pie-markdown 格式  -> 当前没有自动生成命令，需人工整理
需要历史 pie-html 格式      -> 当前没有自动生成命令，需人工整理
```

因此，新增节目时推荐优先维护 `pie-podcast-nav-v2.md` 和 `pie-srt\v2`，然后生成 `docs\episodes` 或 `transcript-site`。只有在必须兼容旧归档格式时，才继续维护 `pie-markdown` 和 `pie-html`。

主要实现见：

- scripts/build_vitepress_site.py:21
- scripts/build_transcript_site.py:490
- package.json:1

当前内置浏览器不可用；我已用本地仓库和 origin/main 的同一提交核对了上述流程。
